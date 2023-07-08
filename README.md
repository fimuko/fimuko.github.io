# My personal portfolio / blog

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

Or you can copy the [`config.yml` file](https://github.com/adityatelange/hugo-PaperMod/blob/exampleSite/config.yml) of the [theme's example site](https://adityatelange.github.io/hugo-PaperMod/).

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
- [ ] search does not seem to work
- [ ] table of contents seems to start from h2 and does not include h1
- [ ] enable comments with giscus