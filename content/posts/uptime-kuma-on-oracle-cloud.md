+++
date = '2025-02-23T15:45:46+01:00'
draft = true
title = 'Uptime Kuma on Oracle Cloud'
+++
[Uptime Kuma](https://uptime.kuma.pet/) is a self-hosted monitoring tool.
Using the
[Oracle Cloud Always Free resources](https://docs.oracle.com/en-us/iaas/Content/FreeTier/freetier_topic-Always_Free_Resources.htm),
I set up a monitoring instance.

## Oracle Cloud setup

At the time of writing, one can have two instances of
[`VM.Standard.E2.1.Micro`](https://docs.oracle.com/en-us/iaas/Content/Compute/References/computeshapes.htm)
(basically 1 OPCU of AMD EPYC 7xxx and 1 GB memory)
with a total of 200 GB block volume,
which is more than enough to run this.
I created one such instance together with 50 GB of block storage, using the
default options.
Make sure to add your SSH key(s) so that you can log on to the machine once
it has been provisioned.

You should then be able to `ssh` to the machine using the IP provided on the
[Instances Dashboard](https://cloud.oracle.com/compute/instances):

```shell
ssh opc@x.x.x.x
```

Before moving on, let us add some more storage to the instance.
First, enable the Block Volume Management Cloud Agent:

- Open the navigation menu and click Compute
- Under Compute, click Instances
- Click your compute instance name to view its details
- Navigate to the “Oracle Cloud Agent” tab page and enable the “Block Volume Management” plugin

Now we create a block volume:

- Navigate to [Block Storage -> Block Volumes](https://cloud.oracle.com/block-storage/volumes)
- Click "Create Block Volume"
- In the form that pops up, enter a name for the volume
- Click "Create Block Volume" at the bottom of the page

Next, we attach the volume to the instance:

- On Block Volume Details page where you should be after creating that volume, scroll down
- From the left-hand side menu, select "Attached Instances"
- Click "Attach to Instance"
- Select the VM created above
- Tick the box "Use Oracle Cloud Agent to automatically connect to iSCSI-attached volumes"
- Select a path, e.g. `/dev/oracleoci/oraclevdb`

Now `ssh` to the instance (see above) to format and mount the Block Volume.

```shell
# Become root
sudo su -
# Check available block devices:
lsblk
```

With the volume path selected above, the `lsblk` command should now yield an `sdb` block device:

```shell
NAME               MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                  8:0    0 46.6G  0 disk
├─sda1               8:1    0  100M  0 part /boot/efi
├─sda2               8:2    0    1G  0 part /boot
└─sda3               8:3    0 45.5G  0 part
  ├─ocivolume-root 252:0    0 35.5G  0 lvm  /
  └─ocivolume-oled 252:1    0   10G  0 lvm  /var/oled
sdb                  8:16   0   50G  0 disk
```

This is the block device we just attached, which can also be checked using
`ls -l /dev/oracleoci/oraclevdb` (it is a symbolic link to `/dev/sdb`).

Create a new partition:

```shell
# Create file system
mkfs -t ext4 /dev/oracleoci/oraclevdb
# Create directory to mount device
mkdir /data
# Mount the device
mount -t auto /dev/oracleoci/oraclevdb /data
# Check that it worked
df -h /data
# Give everyone write permissions
chmod 777 /data
```

To make this mount persist between reboots, we need to add to the `/etc/fstab` file.
First, we need to find out the block ID:

```shell
blkid | grep sdb
```

which should yield something like:

```shell
/dev/sdb: UUID="a45752a3-c920-4529-8512-cdf38a8ea7b9" BLOCK_SIZE="4096" TYPE="ext4"
```

Using this UUID, add to `/etc/fstab`:

```text
UUID=a45752a3-c920-4529-8512-cdf38a8ea7b9 /data	ext4 defaults,noatime,_netdev 0 2
```

One can verify that the entries in the file are correct by running:

```shell
findmnt --verify --verbose
```

### Creating swap memory

When I first tried to `dnf install` something, the console would freeze
and then after some minutes or so I would get a `Killed` statement.
It turns out that the instances runs out of memory too quickly even though
there is already 2 GB of swap memory available.

I therefore created an additional swap file of 4 GB size:

```shell
sudo su -
# Create a 4 GB swap file (1024 * 4096 MB)
dd if=/dev/zero of=/swapfile1 bs=1024 count=4194304
```

Secure and enable swap file:

```shell
chmod 0600 /swapfile1
mkswap /swapfile1
swapon /swapfile1
```

Then add the following line to `/etc/fstab`:

```text
/swapfile1 none swap sw 0 0
```

Check available memory:

```shell
free -m -h
grep Swap /proc/meminfo
swapon -s
```

### Installing docker

The recommended way to run Uptime Kuma is using docker.
To install docker, I largely followed the instructions to install
[docker-engine on Oracle Linux 8 (OL8)](https://oracle-base.com/articles/linux/docker-install-docker-on-oracle-linux-ol8).
However, one can also follow the
[docker instructions for CentOS](https://docs.docker.com/engine/install/centos/).

```shell
dnf install -y dnf-utils zip unzip
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install -y docker-ce
```

Use new block volume mount for storing images:

```shell
mkdir /data/docker
ln -s /data/docker /var/lib/docker
```

Then start docker engine:

```shell
systemctl enable --now docker
docker run hello-world
```

Post-installation steps:

```shell
usermod -aG docker opc
# Log out as root
exit
# Check if things are working as non-root user
newgrp docker
docker run hello-world
```

## Running Uptime Kuma

```shell
mkdir /data/uptime-kuma
cd /data/uptime-kuma
docker run -d --restart=always -p 3001:3001 -v /data/uptime-kuma:/app/data --name uptime-kuma louislam/uptime-kuma:1
```

## Enabling ingress

In the Oracle Cloud Console, navigate to
[https://cloud.oracle.com/networking/vcns](https://cloud.oracle.com/networking/vcns).

- Click on the VCN you created during instance provisioning
- Scroll down, click on Security Lists from the left-hand side menu
- Click the Default Security List for your VCN
- Click Add Ingress Rules and fill in the following:
  - Stateless: No
  - Source CIDR: 0.0.0.0/0
  - IP Protocol: TCP
  - Source Port Range: All
  - Destination Port Range: 443



```shell
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

Download [Caddy](https://caddyserver.com/download).
I downloaded the file on my local machine, then used `scp` to copy it over to
the VM.

```shell
# Make executable
chmod 755 caddy_linux_amd64
# Test if it works
./caddy_linux_amd64
```

Create a configuration file `Caddyfile` for Caddy (adjust for your domain):

```Caddyfile
status.clange.dev

reverse_proxy 127.0.0.1:3001
```

The source for this is the
[Uptime Kuma Wiki](https://github.com/louislam/uptime-kuma/wiki/Reverse-Proxy).

Keep Caddy running: See [caddy docs](https://caddyserver.com/docs/running#linux-service)

```shell
sudo mv caddy_linux_amd64 /usr/bin/caddy
# Test
caddy version
sudo groupadd --system caddy
# Create a caddy user
sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy
```

Download [caddy.service](https://github.com/caddyserver/dist/blob/master/init/caddy.service).

```shell
wget 'https://raw.githubusercontent.com/caddyserver/dist/refs/heads/master/init/caddy.service'
sudo mv caddy.service /etc/systemd/system/caddy.service
sudo mkdir /etc/caddy/
sudo mv Caddyfile /etc/caddy/Caddyfile
# Tag service and binary for SELinux
sudo restorecon -v /etc/systemd/system/caddy.service
sudo semanage fcontext -a -t bin_t /usr/bin/caddy
sudo restorecon -Rv /usr/bin/caddy
sudo systemctl daemon-reload
sudo systemctl enable --now caddy
# Verify
systemctl status caddy
```

There are some complications because of SELinux.
This is also explained under
[SELinux Considerations](https://caddyserver.com/docs/running#selinux-considerations)
on the Caddy webpage.
Instead, I should have installed Caddy using the
[official repositories](https://caddyserver.com/docs/install#fedora-redhat-centos).

On the Cloudflare Dashboard, when creating the DNS A record, make sure to _not_
use the Proxy option.
Instead, let Caddy take care of everything.
More information in
[Sam Mckenzie's blog](https://samjmck.com/en/blog/using-caddy-with-cloudflare/#configuration-with-reverse-proxy-disabled).