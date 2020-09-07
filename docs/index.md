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