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

draft: true

subtitle: ""

tags:
- dialy
- blog
- hugo
categories:

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
```
####  update theme
```bash
git submodule update --remote --merge
```
#### gitalk config (comment)

### deploy
