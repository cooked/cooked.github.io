---
layout: single
title: Deploy to GitHub Pages using Actions 
tags: [GitHub Pages, GitHub Actions, Jekyll]

header:
    teaser: /assets/img/ghpages-actions.png

---

Custom Actions workflow for Jekyll and GitHub Pages that actually works.

## Context

A GitHub Page site can be set up for either one of the following scenario:

1. a repository named **<<user>>.github.io** containing only the site itself in the main
2. a repository containing a **project** and **gh-pages** branch for the site  

The notes below apply to the former case only, which I refer to as *stand-alone* site. In this case GitHub by default builds and deploy the Jekyll site every time you push new commits to the repo.  
This approach works fine for the majority of the cases especially when using one of the [GitHub Pages supported themes](https://pages.github.com/themes/) and [whitelisted plugins](https://pages.github.com/versions/). However, this list is pretty slim compared for example to the whole [rubygems.org](https://rubygems.org/) and if your site uses a plugin not included you can't use the default GitHub deploy method.  

If this is the case you're left with two options, both of which rely on custom GitHub Actions workflows:

1. build the static site on your local machine, push the files to the remote (usually inside *_site* folder) and then use GitHub Actions only to publish it
2. create an end-to-end GitHub Actions workflow to automatically build and deploy the site for you on remote

Going forward I analyze the **end-to-end method** (for a "standalone" site) since it is the most similar to the default deployment with advantage of not being limited to whitelisted plugins (i.e. any plugin from any supported source, even own repos, are allowed).

{: .notice--info}
 **NOTE** - [Since August 2022](https://github.blog/2022-08-10-github-pages-now-uses-actions-by-default/) the very own GitHub Pages deployment framework is based upon GitHub Actions.  

{: .notice--warning}
**CAUTION** - Many of the articles I found online refer to 'gh-pages' branch without providing too much details on the context. As discussed above this is a very specific scenario of *project* and *site* contained in (different branches of) the same repository. This scenario might require a specialized workflow NOT DESCRIBED HERE.

## Configure the use of Actions

The build and deploy method can be changed to 'GitHub Actions' in the repository **Settings > Pages > Build and Deploy** settings.

![gh-actions](/assets/img/gh_actions.png){: .align-center}

## Starter workflows

GitHub Actions provides a set of [starter workflows](https://github.com/actions/starter-workflows) to be used as a base to craft your own. The workflow I picked has a basis is [jekyll.yml](https://github.com/actions/starter-workflows/tree/main/pages)

``` yaml
# Sample workflow for building and deploying a Jekyll site to GitHub Pages
name: Deploy Jekyll site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: [$default-branch]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Setup Ruby
        uses: ruby/setup-ruby@0a29871fe2b0200a17a4497bae54fe5df0d973aa # v1.115.3
        with:
          ruby-version: '3.0' # Not needed with a .ruby-version file
          bundler-cache: true # runs 'bundle install' and caches installed gems automatically
          cache-version: 0 # Increment this number if you need to re-download cached gems
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v2
      - name: Build with Jekyll
        # Outputs to the './_site' directory by default
        run: bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
        env:
          JEKYLL_ENV: production
      - name: Upload artifact
        # Automatically uploads an artifact from the './_site' directory by default
        uses: actions/upload-pages-artifact@v1

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

## Fixes

The Jekyll starter workflow did not work out of the box for me. Here are the changes I made.

### Branch to use

The starter workflow uses the 'default branch' variable to listen to pushes. That didn't work for me so I hardcoded the name of the branch to listen to

``` yaml
on:
  push:
    branches: ['main']
```

### Workflow permissions

I adjusted the GITHUB_TOKEN permissions removing 'content: read'

``` yaml
# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  pages: write
  id-token: write
```

### Update _config.yml

Fine tuning the jekyll config file is possibly the most important step of the process.
Running my custom workflow I stumbled upon the **'invalid date' issue** documented in [Jekyll issue #5267](https://github.com/jekyll/jekyll/issues/5267)

Following the solution in the [Jekyll issue #2938](https://github.com/jekyll/jekyll/issues/2938) I went on and updated my _config.yml file to exclude 'vendor'

``` yaml
exclude: 
- vendor

## add here other paths to exclude
```

## Other references

See also the following links for actions related to GitHub Pages build and deployment:

- [GitHub Actions for GitHub Pages](https://github.com/marketplace/actions/github-pages-action)