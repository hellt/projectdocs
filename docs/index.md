![logo](https://gitlab.com/rdodin/pics/-/wikis/uploads/c212a5585037963b29f90075dde399c1/image.png)

---

I am a firm believer that documentation is an integral part of the project. A terse, twisted, incomplete or sometimes even missing documentation penalizes your projects success.  
At the same time clean, concise and comprehensive documentation is not only something worth being proud of, but an opening to a users' appreciation and fame.

In this post I will share the way I build, publish and host documentation sites for my projects, which is a 100% **free** and **automated** process.

My current documentation *stack* of choice is:

* [MkDocs](https://www.mkdocs.org/) + [MkDocs-Material](https://squidfunk.github.io/mkdocs-material/) - engine and theme
* [Github Pages](https://pages.github.com/) - hosting
* [Github Actions](https://github.com/features/actions) - build/publish workflows

The projects built with that stack: [gnmic](http://gnmic.kmrd.dev/), [yangpath](http://yangpath.netdevops.me/).

## Engine and theme
In this day and age, most projects documentation requirements can be satisfied with a Static Site Generator (SSG) engines. SSG sites can be hosted for free with Github/Gitlab Pages service, are fast to build and render and all you end up working with are version controlled text files.

I've tried many engines and themes and ended up using [MkDocs+Material](https://squidfunk.github.io/mkdocs-material/) combination which satisfies most of my needs.

<p align=center><a href=https://gnmic.kmrd.dev><img src=https://gitlab.com/rdodin/pics/-/wikis/uploads/b3aa7da230a915f3210e6923677b15eb/image.png/></a>
<small><a href=https://gnmic.kmrd.dev>gnmic</a> site build with mkdocs+material</small></p>

My favorite features of Mkdocs-material:

* clean, nice looking and modern responsive interface
* highly customizable in every aspect (icons, fonts, custom css)
* supports tree-like menu and scroll-spy table of contents out of the box
* bundled with lots of markdown extensions so you can create rich documentation
* has embedded full-text search
* has official docker images
* maintained and extensively documented thanks to [@squidfunk](https://twitter.com/squidfunk)

## Repository
As stated in the introduction, I treat documentation as an integral part of the projects codebase and therefore I store it in the same repository where I keep my code - Github.

For example, this very site you are reading is mimicking a projects documentation portal for a project named [`projectdocs`](https://github.com/hellt/projectdocs).

Following a common convention, my documentation is kept within the `docs` directory of a code repository:

```
❯ tree
.
├── docs
│   ├── CNAME
│   └── index.md
├── mkdocs.yml
└── my_big_source_code_file.txt
```

I will not go into the details on how to create the [mkdocs config file](https://github.com/hellt/projectdocs/blob/master/mkdocs.yml), as its quite self-explanatory and is extensively documented in the mkdocs-material project site.

What is important at this stage is that I keep my documentation source in the same Github repo as my project's code.

## Local testing
So once you start to write your documentation its always nice to fire up a development web server to see the state of your work rendered with the mkdocs-material engine and theme.

Since the material theme has lots of nice UI/UX features and elements, I always use the embedded server to track and render the changes as I hack my documentation. I especially enjoy the fact that mkdocs-material comes with official docker images that allow me to use it without installing any Python:

```bash
# being in the projects directory 
docker run --rm -it -p 8000:8000 -v $(pwd):/docs squidfunk/mkdocs-material:5.5.12
INFO    -  Building documentation... 
WARNING -  Config value: 'dev_addr'. Warning: The use of the IP address '0.0.0.0' suggests a production environment or the use of a proxy to connect to the MkDocs server. However, the MkDocs' server is intended for local development purposes only. Please use a third party production-ready server instead. 
INFO    -  Cleaning site directory 
INFO    -  Documentation built in 0.27 seconds 
[I 200907 18:14:19 server:335] Serving on http://0.0.0.0:8000
```

## Deploying documentation
As promised, the documentation built with mkdocs-material will be stored for free thanks to Github Pages offering unlimited sites for public projects.

Although its completely possible to manually trigger the documentation build and publish process with [`mkdocs gh-deploy`](https://www.mkdocs.org/user-guide/deploying-your-docs/) command we are better than that. Instead we will create the Github Action workflow that will build and publish documentation for us exactly when we need it.

This workflow is, again, stored in the code repository - [publish.yml](https://github.com/hellt/projectdocs/blob/master/.github/workflows/publish.yml):

```yaml
name: publish
on:
  push:
    branches:
      - "master"
    paths:
      - "docs/**"

# this commented out example triggers workflows on tagged release
# OR pushes to a specific documentation branches which name starts with `docs-`
# on:
#   push:
#     branches:
#       - "refs/tags/v*"
#       - "docs-*"

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: docker run -v $(pwd):/docs --entrypoint mkdocs squidfunk/mkdocs-material:5.5.12 gh-deploy --force --strict
```

With this workflow this documentation site is built and published when the `push` event happens on master branch and anything under the `docs` directory is changed. The commented part is for the case where I want to sync my documentation to a release cycle.

Once pushed, the workflow process can be monitored in [Actions panel](https://github.com/hellt/projectdocs/actions).

Note, that the `publish` job runs mkdocs-material docker container and deploys the documentation. With that command mkdocs takes the source documentation from the `/docs` directory, builds the static site and pushes it to the remote repository at a specific branch named `gh-pages`[^1].

## Configuring github pages
Once you did your first push and the `publish` workflow has been completed successfully you need to configure Github Pages service to serve your documentation from the [`gh-pages` branch](https://github.com/hellt/projectdocs/tree/gh-pages) where your static site is.

To do so, you need to drill down to the project's Settings->GitHub Pages section and select that branch, keeping the `root` folder:

![pages](https://gitlab.com/rdodin/pics/-/wikis/uploads/12130dec0bdf794359d36e6b7f39f8d6/image.png)

???info "How to find the Github Pages settings"
    ![animation](https://gitlab.com/rdodin/pics/-/wikis/uploads/f25b6fca7df6ab636427dbbffffd49bf/CleanShot_2020-09-07_at_21.43.57.gif)


After this step your documentation will be served at `https://<username>.github.io/<project-name>` with TLS certificates provided by Github. Life is good, you can stop here, unless you want to add a custom domain name.

## Custom domain
Hands down, but `project-name.your-domain.com` is 3.14 times better than `<username>.github.io/<project-name>`. So lets see how can we add subdomain to our documentation site hosted by Github pages.

!!!note
    you need to have a domain registed in your name to proceed. It doesn't matter which DNS manager you will user.

I own `netdevops.me` domain and my goal is to serve this documentation from `projectdocs.netdevops.me`. Here is what I needed to do.

### Creating a CNAME DNS record
In my DNS provider (Cloudflare in my case) I first needed to create a CNAME record that will point the chosen subdomain (`projectdocs`) to the Github Pages site for my user (`hellt.github.io`). The Github Pages site for a user follows a simple pattern of `<username>.github.io`

![cname](https://gitlab.com/rdodin/pics/-/wikis/uploads/15ec6e8ff2a7d35baa1f69ee3c6f3ea9/image.png)

### Adding CNAME file to docs dir
Once the DNS record is created add a [CNAME](https://github.com/hellt/projectdocs/blob/master/docs/CNAME) file to the docs directory which will contain the desired FQDN for a doc site. This file will be used by `mkdocs gh-deploy` command at publish stage to configure Github Pages to take this custom domain name into consideration.

### Trigger documentation build
When the `CNAME` is ready, trigger the documentation build by adding documentation. Once the workflow completes, you should notice that Github Pages settings section reflects the custom domain you provided within `CNAME` file:

![custom_domain](https://gitlab.com/rdodin/pics/-/wikis/uploads/ac5909bb6bb8f9f8fec38955051139d9/image.png)

### TLS for custom domain
If the configuration sequence I explained here was respected, you will notice that "Enforce HTTPS" checkmark is available to select. That means that Github created certificats for your custom domain and is ready to use them and redirect HTTP requests to HTTPS schema. Check this mark and celebrate!

![tls](https://gitlab.com/rdodin/pics/-/wikis/uploads/2b8a85813cac4b8569c8b18c4c048671/image.png)

If "Enforce HTTPS" renders non-selectable, remove the custom domain, hit "Save", add it again, click Save. You should see the certificates regeneration animation to appear.

Now you are officially done!

---

> If you like what I'm doing here and in a mood for sending a token of appreciation, you can leave a comment, or use one of the buttons below  
> <iframe src="https://github.com/sponsors/hellt/button" title="Sponsor hellt" height="35" width="107" style="border: 0;"></iframe>

> <style>.bmc-button img{height: 20px !important;width: 20px !important;margin-bottom: 1px !important;box-shadow: none !important;border: none !important;vertical-align: middle !important;}.bmc-button{padding: 7px 15px 7px 10px !important;line-height: 20px !important;text-decoration: none !important;display:inline-flex !important;color:#FFFFFF !important;background-color:#FF813F !important;border-radius: 5px !important;border: 1px solid transparent !important;padding: 7px 15px 7px 10px !important;font-size: 20px !important;letter-spacing:-0.08px !important;margin: 0 auto !important;font-family:'Lato', sans-serif !important;-webkit-box-sizing: border-box !important;box-sizing: border-box !important;}.bmc-button:hover, .bmc-button:active, .bmc-button:focus {-webkit-box-shadow: 0px 1px 2px 2px rgba(190, 190, 190, 0.5) !important;text-decoration: none !important;box-shadow: 0px 1px 2px 2px rgba(190, 190, 190, 0.5) !important;opacity: 0.85 !important;color:#FFFFFF !important;}</style><link href="https://fonts.googleapis.com/css?family=Lato&subset=latin,latin-ext" rel="stylesheet"><a class="bmc-button" target="_blank" href="https://www.buymeacoffee.com/ntdvps"><img src="https://cdn.buymeacoffee.com/buttons/bmc-new-btn-logo.svg" alt="Buy me a coffee"><span style="margin-left:5px;font-size:14px !important;">For a coffee</span></a>

[^1]: as explained in the [docs](https://www.mkdocs.org/user-guide/deploying-your-docs/#project-pages)