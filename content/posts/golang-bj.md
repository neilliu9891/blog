---
title: "Golang Bj"
date: 2020-12-09T09:34:58+08:00
tags: ["golang"]
categories: ["编程语言"]
---

# Golang笔记

## 20201208 记一次golang exec.Command操作

正常在命令行中执行的命令如下：
```
virsh qemu-agent-command ecs-j9v040bxt8pu '{"execute":"guest-network-get-interfaces"}'
```
需要在代码中执行此命令，但是一直报错，不知道为什么，只知道erron=1
编码中进行了各种如下尝试：
```
//output, err := exec.Command("virsh", "qemu-agent-command", vmInst, "\\'{\"execute\":\"guest-network-get-interfaces\"}\\'").Output()
//output, err := exec.Command("sudo", "virsh", "qemu-agent-command", "ecs-j9v040bxt8pu", `'{\"execute\":\"guest-network-get-interfaces\"}'`).Output()
//output, err := exec.Command("ls", "main").Output()
output, err := exec.Command("/bin/bash", "-c", cmd).Output()
//t.Stderr = os.Stdout
//t.Stdout = os.Stdout
//err = t.Start()
//output := ""
//output, err := exec.Command("/bin/bash", "-c", cmd).Stderr = os.Stdin
//exec.Command().Stderr
```
**说重点：可以看到os.Stdout, 但执行命令不知道失败原因时可以使用将Stderr输出到终端的形式，从而了解错误原因，其实就是操作被拒绝，我用的不是root权限执行的程序，导致程序本身没有root权限。**

```
chown root:root main //改变程序的所属用户和用户组均为root

su root //切换到root进行执行
```
由于没有将错误信息打印出来导致各种猜测错误原因浪费了很长时间，下次注意。

## 20201208 golang race操作检查竞态问题

测试命令
```
go build -race main.go

or

go run -race main.go
```
关于race竞争的检查如下文章讲解的很详细

> https://www.cnblogs.com/yjf512/p/5144211.html 

> https://ms2008.github.io/2019/05/12/golang-data-race/https://ms2008.github.io/2019/05/12/golang-data-race/


## 通过创建新的用户隔离golang private lab的配置
需求：Linux编译的环境是共享的，大家如果都用root用户，当涉及到private lab时，需要设计到git 配置和 ssh访问的问题，目前不知道到怎么配置支持多个用户，所以想到通过创建用户隔离git信息和ssh private key等信息。

1. 创建用户, 将用户创建到root组

```
useradd -g root ly //创建ly用户，并添加到root组
passwd ly  //为ly用户分配密码
```

2. 为用户分配root执行权限

root 用户执行vi /etc/sudoers,在 ## Allow root to run any commands anywhere 之下添加用户名，与root类似
```
## Allow root to run any commands anywhere
root     ALL=(ALL)       ALL
ly     ALL=(ALL)       ALL
```
参考：
> http://www.manongjc.com/article/135688.html

3. 配置golang环境
	- 当切换用户之后，golang的环境配置就丢失了，导致执行go命令提示找不到命令,此时只需要修改~/.bash_profile文件内容，将golang 安装之后bin的所在位置添加到PATH变量中即可。
	- 添加golang project的目录位置，放置到用户目录/go 目录中，再~/go/ 目录下创建pkg 、 bin 、 src目录

```
~/.bash_profile 添加内容
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

GOPATH=$HOME/go
PATH=$PATH:$HOME/.local/bin:$HOME/bin:$GOPATH
PATH=$PATH:/usr/local/go/bin

export PATH
```

4. 配置git环境
5. 配置ssh 公钥私钥信息能够通过ssh访问private gitlab仓库

## 关于golang gin框架中使用goroutine的问题

gin框架中如果在goroutine是用gin.context的话，需要使用context.Copy函数copy一份副本进行操作，否则会出现竞争问题。

参考官方文章：
> https://learnku.com/docs/gin-gonic/2019/examples-goroutines-inside-a-middleware/6178 
```go

func main() {
    r := gin.Default()

    r.GET("/long_async", func(c *gin.Context) {
        // 创建在 goroutine 中使用的副本
        cCp := c.Copy()
        go func() {
            // 用 time.Sleep() 模拟一个长任务。
            time.Sleep(5 * time.Second)

            // 请注意您使用的是复制的上下文 "cCp"，这一点很重要
            log.Println("Done! in path " + cCp.Request.URL.Path)
        }()
    })

    r.GET("/long_sync", func(c *gin.Context) {
        // 用 time.Sleep() 模拟一个长任务。
        time.Sleep(5 * time.Second)

        // 因为没有使用 goroutine，不需要拷贝上下文
        log.Println("Done! in path " + c.Request.URL.Path)
    })

    // 监听并在 0.0.0.0:8080 上启动服务
    r.Run(":8080")
}
```



