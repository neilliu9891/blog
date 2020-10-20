---
title: "Golang交叉编译"
date: 2020-10-20T17:05:34+08:00
tags: ["golang"]
categories: ["编程语言"]

---

# Golang 交叉编译

由于自己的开发环境是Linux开发环境，但自己编写的工具需要运行在windows的环境中，所以需要用到交叉编译工具.

## Ubuntu下编译windows 程序

参考: > https://studygolang.com/articles/8167

- Install gcc-mingw-w64:gcc编译器

```
apt-get install gcc-mingw-w64
```		

- Cross platform compiler

64bit
```
env CGO_ENABLED=1 GOOS=windows GOARCH=amd64 CC=x86_64-w64-mingw32-gcc go build -o main.exe main.go
```
32bit
```
env CGO_ENABLED=1 GOOS=windows GOARCH=386 CC=i686-w64-mingw32-gcc go build -o main.exe main.go
```

- Common Error

```
Q:gcc: error: unrecognized command line option ‘-mthreads’; did you mean ‘-pthread’?
A:CGO_ENABLED=1但是未指定CC编译器

Q:运行时出错(error="Binary was compiled with 'CGO_ENABLED=0', go-sqlite3 requires cgo to work.)
A:CGO_ENABLED未设置成1
```



