+++
date = '2025-02-23T13:55:11+01:00'
title = 'SSHFS on Mac'
tags = ['Mac', 'computing']
+++
Using `scp` or `rsync` requires knowledge of file and directory locations.
Sometimes, it is easier to mount an entire area to interact with it.
This can also be used to edit files "locally".
The example below is given for connecting to the Swiss CMS Tier-3 cluster.

## Setup

```shell
brew install macfuse
brew install gromgit/fuse/sshfs-mac
mkdir ~/t3home
```

Reboot computer.

## Connecting

```shell
sshfs -f -o volname=t3home,\
            reconnect,\
            ServerAliveInterval=15,\
            ServerAliveCountMax=3,\
            idmap=user,\
            auto_xattr,\
            dev,\
            suid,\
            defer_permissions,\
            noappledouble,\
            noapplexattr\
    lange_c@t3ui03.psi.ch:/t3home/lange_c ~/t3home
```

Remove the `-f` option to connect in background. In this case, remember to unmount afterwards:

```shell
umount ~/t3home
```
Some recommendations can be found at <https://sbgrid.org/corewiki/faq-sshfs.md>.
Also, `-ocache=no -onolocalcaches` might be useful, to be checked.
