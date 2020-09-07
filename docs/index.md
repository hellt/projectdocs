![logo](https://gitlab.com/rdodin/pics/-/wikis/uploads/c212a5585037963b29f90075dde399c1/image.png)

---

I am a firm believer that documentation is an integral part of the project. A terse, twisted, incomplete or sometimes even missing documentation penalizes your projects success.  
At the same time clean, concise and comprehensive documentation is not only something worth being proud of, but an opening to a users' appreciation and fame.

In this post I will share the way I build, publish and host documentation sites for my projects, which is a 100% **free** and **automated** process.

My current documentation *stack* of choice is:

* [MkDocs](https://www.mkdocs.org/) + [MkDocs-Material](https://squidfunk.github.io/mkdocs-material/) - engine and theme
* [Github Pages](https://pages.github.com/) - hosting
* [Github Actions](https://github.com/features/actions) - build/publish workflows

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