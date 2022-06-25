---
title: "Blog Start"
date: 2022-05-10T23:23:35+08:00
weight: 0

toc:
  enable: true

math:
  enable: false

comment: true
hiddenFromHomePage: false
hiddenFromSearch: false

lightgallery: true

seo:
  images: []

summary: "start my own static blog"

description: "description"

keywords: ""

resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

draft: false

subtitle: ""

tags:
- dialy
- blog
- hugo

categories:
- blog

---



# Start Blog 

### Create GitHub Repo 
### Clone to local
```bash
git clone git@github.com:ruo-xi/ruo-xi.github.io.git
cd ruo-xi.github.io
```
### Hugo init 
```bash
hugo new site z
mv z/* .
rm -d z

touch layouts/.gitkeep
cp themes/FixIt/.gitignore .
cp -r themes/FixIt/archetypes .
cp -r themes/FixIt/exampleSite/* .

```
### Clone Themes
```bash
git clone --depth 1 https://github.com/Lruihao/FixIt.git themes/FixIt
git submodule add https://github.com/Lruihao/FixIt.git themes/FixIt 


git clone --depth 1 https://github.com/dillonzq/LoveIt themes/LoveIt
git submodule add https://github.com/dillonzq/LoveIt themes/LoveIt 
```
####  update theme
```bash
git submodule update --remote --merge
```
#### gitalk config (comment)

### deploy 

#### GitHub

1. create GitHub Action file
    ```bash
    touch  .github/workflows/gh-pages.yml
    ```

2. add the follow content
    ```yml
    name: github pages

    on:
      push:
        branches:
          - main  # Set a branch to deploy
      pull_request:

    jobs:
      deploy:
        permissions: 
          contents: write
        runs-on: ubuntu-20.04
        concurrency:
          group: ${{ github.workflow }}-${{ github.ref }}
        steps:
          - uses: actions/checkout@v2
            with:
              submodules: true  # Fetch Hugo themes (true OR recursive)
              fetch-depth: 0    # Fetch all history for .GitInfo and .Lastmod

          - name: Setup Hugo
            uses: peaceiris/actions-hugo@v2
            with:
              hugo-version: 'latest'
              extended: true

          - name: Build
            run: hugo --minify

          - name: Deploy
            uses: peaceiris/actions-gh-pages@v3
            if: ${{ github.ref == 'refs/heads/main' }}
            with:
              github_token: ${{ secrets.GITHUB_TOKEN }}
              publish_dir: ./public
              publish_branch: gh-pages  # default: gh-pages
    ```

3. add a branch gh-pages to GitHub repo
    {{< admonition "Tips" ture>}}
      you can change branch name by change the publish_branch in the next step
    {{< /admonition >}}
4. Push
    ```bash
    git push
    ```


### Reference
* [FixIt](https://github.com/Lruihao/FixIt)
* [hugo](https://gohugo.io)
* [GitHub Pages](https://docs.github.com/en/pages)
* [GitHub Action](https://docs.github.com/en/actions)
* [GitHub Actions for Hugo](https://github.com/marketplace/actions/hugo-setup)
* [GitHub Pages action](https://github.com/marketplace/actions/github-pages-action)
