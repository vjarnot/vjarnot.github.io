---
tags: ["github", "go", "hugo"]
title: "Github Pages and Hugo"
date: "2023-02-10"
author: "Voytek Jarnot"
description: "These are the steps I followed to get a basic almost-zero-effort blog published to Github Pages. Adding posts simply requires creating/committing/pushing a new markdown file to your github repo - building the site and publishing it are completely automated."
---

## Extremely-basic Hugo + Github Pages Step-by-step Guide

### Background
* https://pages.github.com/
* https://gohugo.io/getting-started/quick-start/

### Setup

I mostly followed the steps outlined here https://gohugo.io/hosting-and-deployment/hosting-on-github/ with some minor changes.

Here's what I did - this assumes you have a linux environment you're going to work in, and a github account:

1. Create repo for your github pages site
    * If you want your site available at `USERNAME.github.io`, ensure the repo is named `USERNAME.github.io`
2. Clone the repo:
    * `git clone https://github.com/USERNAME/USERNAME.github.io`
3. Install Hugo
    * on my Ubuntu instance, this was a simple `apt install hugo`
4. Generate site (adapted from https://gohugo.io/getting-started/quick-start/ )
    * Assuming you did the `git clone` from ~/projects and you're still in ~/projects
        1. `hugo new site USERNAME.github.io --force`
            * `--force` is here because the `USERNAME.github.io` directory already exists
        2. `cd USERNAME.github.io`
        3. Install a theme
            * `git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke`
        4. Create a hello-world blog post
            * `hugo new posts/hello-world.md`
        5. Edit hello-world.md and remove `draft: true`
        6. Test site
            * run `hugo server`
            * open localhost:1313 in a browser
5. Create a `gh-pages` branch in your repo on github
6. Set up Github actions to auto-publish your site (adapted from https://gohugo.io/hosting-and-deployment/hosting-on-github/ )
    * NOTE: this assumes you want your site available at `USERNAME.github.io` - github expects those type of github pages to be published from the main branch.
    However, we're going to deviate and commit our source to the `main` branch, while publishing from the `gh-pages` branch
    * in ~/projects/USERNAME.github.io
        1. `mkdir -p .github/workflows`
        2. create .github/workflows/gh-pages.yml:
            ```
            name: github pages
            on:
              push:
                branches:
                  - main  # Set a branch that will trigger a deployment
              pull_request:

            jobs:
              deploy:
                permissions:
                  contents: write
                runs-on: ubuntu-22.04
                steps:
                  - uses: actions/checkout@v3
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
                      publish_dir: ./public
            ```
7. Remember how I mentioned that github pages wants to publish from your `main` branch? Let's change that:
    * On your github repo, click Settings, then Pages
    * Under Build and Deployment, select `gh-pages`
8. Success?
    * What you should have now is the following setup:
        1. You checkout, edit, commit, and push to `main`
        2. Your github action executes on push, generates your site, and commits that output to `gh-pages`
        3. Github sees the commit to `gh-pages` and publishes your site

### Troubleshooting

Check the `Actions` tab on your Github repo - you should see an action with the same name as your commit comment - this is running the workflow we defined above.
If that's erroring, you should probably fix it ;) Subsequent to that you should see Github's `pages build and deployment` action - that's the Github Pages magic that actually publishes your site.

