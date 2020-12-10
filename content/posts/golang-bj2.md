---
title: "Golang 记一次log日志不能输出的问题"
date: 2020-12-10T09:28:45+08:00
tags: ["golang"]
categories: ["编程语言"]
draft: false
---

# Golang 记一次log日志不能输出的问题

由于线上运行程序过程中出现了log日志打印不出来的情况，分析代码模拟测试复现问题。由于没有锁住log变量的问题

目录结构如下

```
.
├── ccc
│   └── ccc.go
├── go.mod
├── go.sum
├── hhh
│   └── hhh.go
├── log
│   └── log.go
├── main.go
└── network-agent
    ├── network-agent.log -> network-agent.log.2020-11-06
    └── network-agent.log.2020-11-06

4 directories, 8 files
```

ccc.go
```
package ccc

import (
        "demo/log"
        "sync"
)

var logger = log.GetNetworkLog("ccc")

//var logger = capnslog.NewPackageLogger("network", "ccc")

//var logger = log.GetNetworkLog()

func PrintI(i int, wg *sync.WaitGroup) {
        defer wg.Done()
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Infof("INFO :CCC", i)
}

func PrintE(i int, wg *sync.WaitGroup) {
        defer wg.Done()
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Errorf("ERROR :CCC", i)
}
func PrintII(i int) {
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Infof("INFO :CCC", i)
}

func PrintEE(i int) {
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Errorf("ERROR :CCC", i)
}
```

hhh.go
```
package hhh

import (
        "demo/log"
        "sync"
)

var logger = log.GetNetworkLog("hhh")

//var logger = capnslog.NewPackageLogger("network", "hhh")

//var logger = log.GetNetworkLog()

func PrintI(i int, wg *sync.WaitGroup) {
        defer wg.Done()
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Infof("INFO :HHH", i)
}
func PrintE(i int, wg *sync.WaitGroup) {
        defer wg.Done()
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Errorf("ERROR :HHH", i)
}
func PrintII(i int) {
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Infof("INFO :HHH", i)
}
func PrintEE(i int) {
        //fmt.Printf("%p, %p \n", &logger, &logger.Mutex)
        logger.Errorf("ERROR :HHH", i)
}
```

log.go
```
package log

import (
        "fmt"
        "io"
        "os"
        "path/filepath"
        "sync"
        "time"

        "github.com/coreos/pkg/capnslog"
        rotatelogs "github.com/lestrrat-go/file-rotatelogs"
)

type NetworkLog struct {
        sync.Mutex
        pkg    string
        format capnslog.Formatter
}

var networkLog = new(NetworkLog)

func init() {
        SetupLogging()
}

type stop struct {
        error
}

func Retry(attempts int, sleep time.Duration, fn func() error) error {
        if err := fn(); err != nil {
                if e, ok := err.(stop); ok {
                        return e.error
                }
                if attempts--; attempts > 0 {
                        fmt.Printf("retry func error: %s. attempts #%d after %s.", err.Error(), attempts, sleep)
                        time.Sleep(sleep)
                        return Retry(attempts, sleep, fn)
                }
                return err
        }
        return nil
}

func IsPathExist(path string) bool {
        _, err := os.Stat(path)
        if err == nil {
                return true
        }
        if os.IsNotExist(err) {
                return false
        }
        return false
}

func EnsureTree(path string) error {
        err := Retry(10, 2*time.Second, func() error {
                if !IsPathExist(path) {
                        err := os.MkdirAll(path, os.ModePerm)
                        if err != nil {
                                return err
                        }
                        return nil
                }
                return nil
        })
        return err
}

func GetWriter(filename string) io.Writer {
        writer, err := rotatelogs.New(
                filename+".%Y-%m-%d",
                rotatelogs.WithLinkName(filename),         // 生成软链，指向最新日志文
                rotatelogs.WithMaxAge(365*24*time.Hour),   // 文件最大保存时间
                rotatelogs.WithRotationTime(24*time.Hour), // 日志切割时间间隔
        )

        if err != nil {
                //ulog.Fatalf("config local file system logger error. %+v", errors.WithStack(err))
                fmt.Println("config local file system logger error")
        }
        return writer
}

func SetupLogging() {
        logfile := "/home/neil/go/src/demo/network-agent/network-agent.log"
        dirname := filepath.Dir(logfile)
        EnsureTree(dirname)
        //capnslog.SetFormatter(capnslog.NewPrettyFormatter(writer, true))
        networkLog.format = capnslog.NewPrettyFormatter(GetWriter(logfile), true)
        //capnslog.SetFormatter(capnslog.NewPrettyFormatter(GetWriter(logfile), true))
}

func (log *NetworkLog) Infof(format string, args ...interface{}) {
        fmt.Println("begin lock")
        log.Lock()
        fmt.Println("locked")
        defer func() {
                fmt.Println("begin unlock")
                log.Unlock()
                fmt.Println("unlocked")
        }()
        if log.format != nil {
                log.format.Format(log.pkg, capnslog.INFO, 2, fmt.Sprintf(format, args...))
        }
        //time.Sleep(time.Duration(2) * time.Second)
}

func (log *NetworkLog) Errorf(format string, args ...interface{}) {
        fmt.Println("begin lock")
        log.Lock()
        fmt.Println("locked")
        defer func() {
                fmt.Println("begin unlock")
                log.Unlock()
                fmt.Println("unlocked")
        }()
        if log.format != nil {
                log.format.Format(log.pkg, capnslog.ERROR, 2, fmt.Sprintf(format, args...))
        }
        //time.Sleep(time.Duration(2) * time.Second)
}

func (log *NetworkLog) Fatalf(format string, args ...interface{}) {
        fmt.Println(fmt.Sprintf(format, args...))
}

// 正确
func GetNetworkLog(pkg string) *NetworkLog {
        networkLog.pkg = pkg
        return networkLog
}

// 错误
//func GetNetworkLog() NetworkLog {
//return *networkLog
//}

```

main.go
```
package main

import (
        "demo/ccc"
        "demo/hhh"
        "fmt"
        "sync"
)

func main() {
        var wg sync.WaitGroup
        fmt.Println("---")

        //log.SetupLogging()
        for j := 0; j < 100; j++ {
                for i := 0; i < 4; i++ {
                        wg.Add(1)
                        switch i {
                        case 0:
                                fmt.Println("---", i)
                                go ccc.PrintI(i, &wg)
                        case 1:
                                fmt.Println("---", i)
                                go hhh.PrintI(i, &wg)
                        case 2:
                                fmt.Println("---", i)
                                go ccc.PrintE(i, &wg)
                        case 3:
                                fmt.Println("---", i)
                                go hhh.PrintE(i, &wg)
                        }
                }
        }

        wg.Wait()
}
```


