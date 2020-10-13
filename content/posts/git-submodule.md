---
title: "Git Submodule简单应用"
date: 2020-10-13T08:01:32+08:00
tags: ["git"]
categories: ["技术杂谈"]
draft: false
---

# Git Submodule 命令

问题：如果我们的项目中需要其他git项目作为我们的lib库使用，且这个lib库也是我们来维护，如何让我们在项目修改中不用关心是lib库修改，还是本项目修改，而是都作为本次的修改一并提交呢？
答：使用git submodule，这个命令能在一个git项目中引用另一个git项目，将所有修改一并提交到两个git仓库中。
### 命令
```
git submodule add <url> <dir>
```
- url: 表示子git的仓库地址
- dir：下载到本项目的目录位置

### example
假设有一个blog项目，其中themes/blog-themes主题作为子git
```
git clone git@blog.git
cd blog
git submodule add git@blog-themes.git themes/blog-themes
```
提交效果展示
![submodule](/images/image_2020-10-13-08-12-18.png)
