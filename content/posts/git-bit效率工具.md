---
title: "Git Bit效率工具"
date: 2020-10-26T16:16:54+08:00
tags: ["git"]
categories: ["效率工具"]

---

# Bit之git效率工具

bit: 建立再git之上的实验性现代化代码git'CLI
项目地址：https://github.com/chriswalz/bit

## Install

```
$ curl -sf https://gobinaries.com/chriswalz/bit | sh; curl -sf https://gobinaries.com/chriswalz/bit/bitcomplete | sh && echo y | COMP_INSTALL=1 bitcomplete
$ bit
```

2. 从如下地址下载安装

```
https://github.com/chriswalz/bit/releases
```

3. Go安装

确保使用了go module
```
$ go get github.com/chriswalz/bit
$ bit
```
## Platform

```
iTerm2 (macOS)
Terminal.app (macOS)
Command Prompt (Windows)
WSL/Windows Subsystem for Linux (Windows)
gnome-terminal (Ubuntu)
```

## Special Cmd
```
1、bit save [commit message]

创建一个新的提交。

2、bit sync

同步对 origin 分支的更改。

大部分时候，bit sync 相当于 bit commit -m "I can still use git commands", bit pull -r origin master
```

## Normal Work Flow

1. 切换分支
```
$ bit
> bit switch example-branch
? Branch does not exist. Do you want to create it? Yes
Switched to a new branch 'example-branch'
```
2. 做些改动
```
$ bit add *
$ bit save "add important feature"
```

3. 做其他改动

```
$ bit save
```
4. push 改变到 origin

```
$ bit sync
```
5. 一段时间后，可以再同步别人的修改

```
$ bit sync
```

一般都直接输入 bit，然后回车。接着输入会自动提示，如开始的动图

