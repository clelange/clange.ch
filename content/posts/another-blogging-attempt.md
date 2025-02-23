+++
date = '2025-02-23T12:28:50+01:00'
title = 'Another Blogging Attempt'
+++
I have had a homepage for ages, but after some initial enthusiasm,
it quickly turned out to be too painful to add to it.
For my previous attempt, I had also used
[Hugo](https://gohugo.io/),
but then after some Hugo updates the theme broke.
Furthermore, I had to manually synchronise the pages to the FTP server
and my web hoster back made it somewhat complicated and at some point
went completely out of business.

I am now starting another attempt to start blogging on a more regular basis.
Since I am used to writing Markdown and I like Hugo, I am again using this
approach.
However, as a first step, I automated the deployment of my homepage using
[Cloudflare Pages](https://gohugo.io/hosting-and-deployment/hosting-on-cloudflare-pages/).
The source of this page is at

{{< github repo="clelange/clange.ch" >}}

--- and with this you already see that I am now using a new theme called
[Blowfish](blowfish.page/).
This theme makes it easy to make my webpage look nice and it seems
well-maintained so that it should continue to work in the future.
It also has a lot of nice
[Shortcodes](https://blowfish.page/docs/shortcodes/).

Besides filling this webpage with content, I will now have to look into
automatically updating this theme that I have added as a git submodule.
Otherwise, I risk that things break again over time.
Maybe some GitHub actions such as
[update-git-submodules](https://github.com/marketplace/actions/update-git-submodules)
running on a schedule will help with that.
