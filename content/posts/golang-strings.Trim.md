---
title: "Golang strings Trim"
date: 2020-10-18T20:50:56+08:00
tags: ["golang"]
categories: ["编程语言"]
draft: true
---

> 有时候不是自己做的不够多，而是自己总结的不够多；走的太快容易忘记迷失方向，也不会留下脚印。

---

> 只想给自己留下点儿什么！

# Golang strings.Trim函数 

## TrimSpace 清除空白符
Q:what's the white space?
A: \t, \n, \v, \f, \r, ' ' and utf-8 space

example:
```golang
package main

import (
	"fmt"
	"strings"
)

func main() {
	var str = " \n\r\t helloworld \n\r\t\f  "
	fmt.Println(strings.TrimSpace(str))

	fmt.Println("vim-go")
}
```
## TrimSpace源码解析


