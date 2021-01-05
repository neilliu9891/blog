---
title: "Go Ovn源码阅读思考"
date: 2021-01-05T17:50:43+08:00
tags: ["ovn"]
categories: ["技术杂谈"]
draft: true
---

# go-ovn源码阅读思考

最近在学习ovn相关的内容，需要通过go-ovn库实现向ovn写入信息。
go-ovn的源码实现大致实现功能简单梳理一下：
go-ovn 代码的核心思想就是封装了libovsdb库，将rpc接口修改为了api接口，定义了marshal unmarshal的转换。而libovsdb则基于RFC7047协议实现了RPC接口的基本功能，包括双方通信的方法。

底层思想：
1. 需要理解ovsdb manager protocol协议的内容，即协议中定义了操作的方法包括update、notify等等。
2. 需要理解ovsdb表的内容便于理解json是如何转换的，对应的转换名称是什么。

##  学到内容：

### json-rpc

json-rpc通信(来自百度百科)：> https://baike.baidu.com/item/json%20rpc/13675431?fr=aladdin

rpc例子：

> https://github.com/cenkalti/rpc2
> https://github.com/neilliu9891/examples/tree/main/rpc-demo

### interface实现小技巧


### golang 中function的定义
