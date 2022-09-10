---
author: Davide Quaranta
title: Deploying Hugo on GitHub Pages with GitHub Actions
date: 2022-09-10T08:00:00+02:00
categories: [DevOps]
tags: [software-engineering, webdev, hugo]
description: "A GitHub Action is the automatic execution of a job, after a specified trigger on a GitHub repository. For example, let's say that we want to run a linter on each new commit, we can create a GitHub action to do it. In this post we'll see a workflow to automatically deploy a Hugo site on GitHub Pages."
---

Actually there is a spicy requirement that makes the situation more interessant: **the Hugo theme is a private Git submodule**.

## Development setup

We don't just want to deploy on each commit, but only when we **commit** on the `main` branch. With this regard, let's consider two branches:

* `main`: production branch.
* `dev`: development branch (default).

We want to trigger our GitHub action after new commits on `main`; in this way, we can freely develop and break things on `dev`.

This post hence assumes that we already have these two branches. Another requirement is that your site's repo is named `yoursite.github.io`.

Furthermore, we are assuming to have our Hugo theme as **Git submodule** in `./themes/<theme-name>`. If the theme is not in a submodule, this post is still useful but the PAT steps are not necessary.

## What we need

We mainly need to create:

* A GitHub Action **workflow** file.
* A **Personal Access Token** (PAT).

The workflow file specifies what to do and when; the PAT allows the workflow to access and clone our **private repository** containing the theme.

### Workflow code

This YAML specification is a copy-paste of the sample provided by GitHub. We can just put it in `.github/workflows/hugo.yml`.

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["main"]

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

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.99.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-64bit.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.ACCESS_TOKEN }}
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v2
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

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

Actually the only thing that has been **edited** from the default workflow is the `token` field in the `Checkout` step of the `build` job. See the ([documentation](https://github.com/actions/checkout)) for reference.

The `token` field says that we are using the access token defined in `secrets.ACCESS_TOKEN`.

### Create a Personal Access Token (PAT)

To get a PAT we can:

1. Go to our GitHub **settings**.
2. Head to **Developer Settings**.
3. Move to **Personal access tokens**.

Now we can generate a new PAT, that as how `actions/checkout@v3` currently works, must have full **repo scopes**. 

### Add the PAT to the secrets

Now we just need to add the newly generated PAT to our repository secrets; it's simple:

1. Go to the **settings** of the site's repository.
2. Go to **Secrets** > **Actions**.
3. Click **New repository secret** and paste the PAT, that should be named `ACCESS_TOKEN`.

## Conclusion

That's it. If GitHub didn't automatically create for you a new GitHub Pages with **GitHub Actions** as source, just create it yourself.

Now, each commit in `main` triggers the pipeline, that if successful will deploy the site to GitHub Pages.
