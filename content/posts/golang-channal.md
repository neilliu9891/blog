---
title: "Golang Channal"
date: 2020-12-10T10:49:54+08:00
tags: ["golang"]
categories: ["编程语言"]
draft: true
---

# golang channel

## channal结构体

## channal 单方向
- 只读channal <-chan TYPE
```
var c chan int

func readc(c <-chan int){
    x <- c
}
```
- 只写channal chan<- TYPE
```
var c chan int

func writec(c chan<- int){
    c <- x
}
```
1. 关闭的channal可以读，不阻塞，但返回状态fasle
2. 向关闭的channal写会panic 
3. 关闭已经关闭的channal会panic
3. 关闭已经关闭的channal会panic
3. 关闭已经关闭的channal会panic

## selete channal运用
1. selete 匹配不上则阻塞
2. selete 随机选择case
3. default为特殊的case如果都比配不上匹配default
4. 
