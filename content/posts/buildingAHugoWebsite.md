---
title: "Building A Hugo Website"
date: 2021-11-10T18:23:41-05:00
draft: false
tags:
  - hugo
  - git
  - website
---
## Why am I using `hugo`

This website is a static site built using `hugo`. I made the decision to go with
this approach after spending about 10 mins using the free
[wordpress.com](https://wordpress.com) blogging platform. I found the editor a
nightmare and it felt like my entire experience would be about trying to make it
look good rather than focusing on content. I wanted something that looked and
felt more like [Jonathon Hodgson's](https://jonathanh.co.uk) site: minimalist in
style and content focused. Critically, I wanted to be able to write my content
in markdown because a) I use it a lot in other activities so it comes easily to
me and b) I didn't want to faff around writing `html` directly.

Following some of the links on Jonathon's page showed me that he had developed
bash script for generating a site from markdown. A bit of googling later, I
found that there were lots of ways to do this, and that if I chose wisely I
could have a free site hosted on [github.io](https://github.io) that would be
managed by `git` and would compile automatically on pushing to
[github](https:/github.com). It all sounded perfect.

I considered two possible tools: [jekyll](https://jekyllrb.com) and
[hugo](https://gohugo.io) both of which met my criteria. After what was probably
not enough evaluation I went with `hugo`. _Full disclosure, this was before I
realized that `jekyll` is the website builder that `github.io` recommend._ 

## Installing hugo

Installing `hugo` is very easy on `macos` since you can use the
[homebrew](https://brew.sh/) package manager. (This was was really clinched it
for me in the `hugo` vs `jekyll` consideration.)

All you need to do is

```bash
brew install hugo
```

`homebrew` will do its thing and `hugo` will be installed.

## Configuring the github.io site

To establish a website at `github.io` I made a "special" repository on
`github.com` that has the name `<username>.github.io`. Once that is created
adding a file to the route of the repo called `index.html` will, after a few
minutes syncing, result in a website displaying the content of `index.html` at
`https://<username>.github.io`. This process for doing this as a 'user or
organization site' is described at the bottom of the [github.io landing
page](https://github.io).

Of course, for a `hugo` site, there is no need to create an `index.html` file as
`hugo` will do it for you.

## Establishing a hugo site on github

- Go to `github.com` and establish the repository `<username>.github.io` where
  `<username>` is the username associated with the `github.com` account.
- Clone the repository onto the local machine
```bash
git clone <username>.github.io
```
- Establish an action to build the website on push
  - In the repository, create the directory `.github/workflows`
  - In the `workflows` directory create the file `gh-pages.yml`
  - Add the following content to the file:
  ```yaml
  name: github pages

  on:
    push:
      branches:
        - main  # Set a branch to deploy
    pull_request:

  jobs:
    deploy:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v2
          with:
            submodules: true  # Fetch Hugo themes (true OR recursive)
            fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

        - name: Setup Hugo
          uses: peaceiris/actions-hugo@v2
          with:
            hugo-version: 'latest'
            # extended: true

        - name: Build
          run: hugo --minify

        - name: Deploy
          uses: peaceiris/actions-gh-pages@v3
          if: github.ref == 'refs/heads/main'
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./docs
  ```
  - I used the workflow script from [gohugo.io host on github
    pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/) but
    modified the line `runs-on: ubuntu-20.04` to `runs-on: ubuntu-latest`.
  - Push the file to repository.
- Configure `github.com` to publish the site to `github.io`
  - In the `<username>github.io` repository web view at `github.com` go to
    `settings->Pages`
  - Set the _Source_ to be the `gh-pages` branch and the directory to be `/ (root)`
  - If the `gh-pages` branch is not available, it is because the action script
    has not run yet to create it. Come back and do this step after something has
    been pushed to the repository that triggers the action.
- Back in the command line, in the directory above the repository
  `<username>.github.io`, initialize the repository as a new `hugo` website
```bash
hugo new site ./<usernamei>.github.io -f
```
- This creates this sets up the `hugo` site in the repo. The `-f` argument is
  needed to force `hugo` to make the site in a directory that already contains
  files. (Remember, even an empty repository contains the `.git` directory.)
- Change directory into the repository directory
- Use `git status` and `git add` to add all files. Not all of the directories
  created by the initialization process have files in them, and `git` doesn't
  commit empty directories. This has not been a problem for me.
- Use `git commit` and `git push` to push the files.
  - I found it easier to push the files at this stage than after I added the
    submodules because I could use `git add -A` to add all files. After the
    submodule for the theme is installed, this causes issues as it tries to add
    the submodule files which causes an error.
- Install the theme into the `themes/` directory.
  - Move into the `themes/` directory
  - Install the theme as a `git submodule`
  ```bash
  git submodule add <theme-git-repository>
  ```
  - Move back to the top level of the repository
  - Open the file `config.toml` in text editor
  - Add the line
  ```toml
  theme = '<themename>'
  ```
  - For this website, I used the [mainroad
    theme](https://themes.gohugo.io/themes/mainroad/). There are good theme
    installation instruction on theme webpage. 
- Return to the top level directory of the `<username>.github.io` repository in
  the command line
- Add content to the website, e.g. posts and pages, following the instructions
  on the [gohugo.io quickstart](https://gohugo.io/getting-started/quick-start/)
  guide.
- When ready, set the pages to have
```yaml
draft: false
```
- Add, commit, and push the content to `github`.
  - This should trigger the `gh-pages` action to compile the website on to the
    `gh-pages` branch
  - After a few minutes, the website at `https://<username>.github.io` should update


## Continuing to working with the site

Because the site is synced via `git` and `github` it is straightforward to
develop it from multiple computers. However, because it is common to include the
website theme via a submodule, care must be taken when cloning the repository.

The [git book](https://git-scm.com/book/en/v2/Git-Tools-Submodules) explains the
procedure for cloning a repository and its submoudles as:

```bash
git clone <repo-address>
git submodule init
git submodule update
```

A one line alternative, that is the easiest way to clone a repo with submodules
is:

```bash
git clone --recurse-submodules <repo-address>
```

The above should successfully clone the website and themes installed using the
submodule approach.

