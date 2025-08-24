# clange.ch

## Creating new content

```shell
hugo new content content/posts/another-blogging-attempt.md
```

## Running the development server

```shell
hugo server --buildDrafts
```

## Updating blowfish theme

See <https://blowfish.page/docs/installation/#update-using-git>:

```shell
git submodule update --remote --merge
```

## Cloudflare deployment

Mind that the `hugo` version might not be compatible when updating the blowfish theme!

The version is currently hard-coded in
<https://dash.cloudflare.com/7cd41f075f99429ec1098decfeeba5ea/pages/view/clange-ch/settings/production>
to `HUGO_VERSION=0.148.2`.

See <https://developers.cloudflare.com/pages/framework-guides/deploy-a-hugo-site/#use-a-specific-or-newer-hugo-version>
for more information on pages customisation.
