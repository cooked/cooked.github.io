---
layout: single
title: Deploy to GitHub Pages using Actions 
tags: [GitHub Pages, GitHub Actions, Jekyll]

header:
    teaser: /assets/img/ghpges-actions.png

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

The build and deploy method can be changed to GitHub Actions in the repository **Settings > Pages > Build and Deploy** settings.

![gh-actions](/assets/img/gh_actions.png){: .align-center}

## Starter workflows

GitHub Actions provides a set of [starter workflows](https://github.com/actions/starter-workflows) to be used as a base to craft your own.

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
