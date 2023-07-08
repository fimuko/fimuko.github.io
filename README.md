# My personal portfolio / blog

フィムコ

Built with Hugo static sites generator, and PaperMod theme.
## Installation steps

0. This repo was already existing on remote (github), cloned this repo to local 

1. Installed go, hugo from binaries, etc. as described [here](https://gohugo.io/installation/windows/)

As per the [hugo quickstart](https://gohugo.io/getting-started/quick-start/):

2. In bash (git bash on Win) navigated to the project
```bash
cd ~/workspace-git/personal/fimuko.github.io
cd /c/workspace-git/personal/fimuko.github.io
```

3. Initialized hugo project
```bash
hugo new site . -f yml
```

As per the installation steps outline in the [theme's installation manual](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-installation/):

4. Added the theme
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

5. Removed .toml file...

```bash
rm hugo.tml
```
and replaced it with the `config.yml` from [here](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-installation/#sample-configyml)

OR you can copy the [`config.yml` file](https://github.com/adityatelange/hugo-PaperMod/blob/exampleSite/config.yml) of the [theme's example site](https://adityatelange.github.io/hugo-PaperMod/).

```bash
curl https://raw.githubusercontent.com/adityatelange/hugo-PaperMod/exampleSite/config.yml -o config.yml
```

6. Created `archetypes/posts.md` with the content from [here](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-installation/#sample-pagemd)

7. Created `content/archives.md` with the following content:

```yml
---
title: "Archive"
layout: "archives"
url: "/archives/"
summary: archives
---
```
8. Created `content/search.md` with the following content:

```yml
---
title: "Search" # in any language you want
layout: "search" # is necessary
# url: "/archive"
# description: "Description for Search"
summary: "search"
placeholder: "..."
---
```

9. Adjusted the `config.yml`, `archetypes/posts.md`, `archetypes/default.md` as per my needs, as described in the [theme's webpage](https://adityatelange.github.io/hugo-PaperMod/posts/papermod/papermod-features/), or in [this video](https://www.youtube.com/watch?v=sm3IuE7zkYQ&list=PLeiDFxcsdhUrzkK5Jg9IZyiTsIMvXxKZP)

10. For automated build on github pages, I added `.github/workflows/hugo.yaml` with the following content:

11. We are now ready to blog, you add content like this:

```bash
hugo new posts/my-first-post.md
```

Test serving the site locally:
```bash
hugo server
```

Building locally with `hugo` (to `public` folder) is not necessary, because we are using CI/CD with github pages as described [here](https://gohugo.io/hosting-and-deployment/hosting-on-github/)


## TODO:
- [X] search does not seem to work > solved when removed cache
    - https://github.com/adityatelange/hugo-PaperMod/discussions/1258
    - solutions? look here:
        - theme's github issues: https://github.com/adityatelange/hugo-PaperMod/issues?q=is%3Aissue+search
            - https://github.com/adityatelange/hugo-PaperMod/discussions/731
            - https://github.com/adityatelange/hugo-PaperMod/issues/556
        - example site code https://github.com/adityatelange/hugo-PaperMod/blob/exampleSite/
        - https://github.com/adityatelange/hugo-PaperMod/discussions/1175
        - https://discord.com/channels/971046860317921340/1008546760156790815 and this persons site: https://github.com/PlanterApp/planter_blog/tree/main
        - 
- [X] table of contents seems to start from h2 and does not include h1 > solved with setting UseHugoToc to false
- [X] tags and categories counts include posts in draft state > solved when removed cache
- [x] mermaid does not work > solved it like here https://www.bodunhu.com/blog/posts/add-mermaid-to-hugo-with-dark-mode/ (not ideal as i will have to switch from code blocks to shorcode)
    - https://gohugo.io/content-management/diagrams/#mermaid-diagrams > solving it like here in fact
    - not much in webpage wiki
    - github issues / discussions:
        - https://github.com/adityatelange/hugo-PaperMod/discussions/763 , https://github.com/binick/binick.github.io/tree/main/src , code he allegedly added: https://github.com/search?q=repo%3Abinick%2Fbinick.github.io+mermaid&type=code
        - https://github.com/adityatelange/hugo-PaperMod/discussions/850
        - https://github.com/bee-san/hugo-PaperMod/commit/c5c9ffc3006f5fb499b2a8c66b94bdd3800433eb
        - https://github.com/adityatelange/hugo-PaperMod/discussions/853
- [ ] enable comments with giscus