---
title: "Go Ovn源码阅读思考"
date: 2021-01-05T17:50:43+08:00
tags: ["ovn"]
categories: ["编程语言"]

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

json-rpc通信(来自百度百科)：
> https://baike.baidu.com/item/json%20rpc/13675431?fr=aladdin

rpc例子：

> https://github.com/cenkalti/rpc2  
> https://github.com/neilliu9891/examples/tree/main/rpc-demo

### interface实现小技巧
1. 使用_ 变量提前暴露struct是否完全实现interface所定义的函数.
  - 例如当我们的接口有很多的定义方法时，如果不采用强制暴露的方式，很难在第一时间发现错误。
  - 代码中的var _ Client = &ovndb{}, 目的是校验ovndb struct是否全部实现了Client接口的所有方法，否则编译报错。

2. 如何保证struct不需要实现全部的interface方法，同样能够转换成Interface类型呢？将interface作为struct的匿名变量

```golang
type TestI1 interface {
	Test1() error
}

type TestS1 struct {
	TestI1
}

var t1 TestI1 = &TestS1{}

func main() {
	fmt.Println("vim-go")
	fmt.Printf("%v\n", t1)
}
```

### golang 中function的定义

1. 经常看到函数的返回值被定义了名称，此时可以直接使用此名称而不用在函数内部定义

```
func main() {
	r1, r2 := NameReturn()
	fmt.Printf("%s, %d\n", r1, r2)
}

func NameReturn() (r1 string, r2 int) {
	r1 = "r1"
	r2 = 2
	return r1, r2
}
```
  - > https://www.jianshu.com/p/a5bc8add7c6e, struct中嵌入结构体的解答
2. 函数定义应该知道的几个内容

  - 函数无须前置声明
  - 不支持命名嵌套定义，支持匿名嵌套
  - 函数只能判断是否为nil，不支持其它比较操作
  - 支持多返回值
  - 支持命名返回值
  - 支持返回局部变量指针
  - 支持匿名函数和闭包

