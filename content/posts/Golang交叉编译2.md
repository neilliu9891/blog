---
title: "Golang交叉编译2"
date: 2020-10-30T09:52:44+08:00
tags: ["golang", "交叉编译"]
categories: ["编程语言"]

---

# Golang交叉编译必知必会

## 需求
在Linux x86的操作系统上编译 Linux Aarch64平台的golang程序，程序中引用的cgo代码。
典型的交叉编译需求，Golang支持的平台列表参见官方文档：

> https://golang.google.cn/doc/install/source

## 实现

编译golang程序，如果是交叉编译只需要指定GOOS，GOARCH平台即可，根据需求编译如下：

```
env GOOS=linux GOARCH=arm64 go build -ldflags $(LDFLAGS) -o ./bin/$(BIN_NAME)-aarch64 *.go
```

由于代码中包含了cgo程序， CGO_ENABLED=1程序默认开启，此时golang编译时会查找本地的gcc进行编译，由于gcc并不是目标平台的gcc。所以我们在编译时要指定对应平台的gcc进行编译。指令如下：

```
env CGO_ENABLED=1 GOOS=linux GOARCH=arm64 CC=aarch64-linux-gnu-gcc-10 go build -ldflags $(LDFLAGS) -o ./bin/$(BIN_NAME)-aarch64 *.go
```

如何现在安装对应平台的gcc呢？
ubuntu有自己的安装包管理器，所以通过如下命令查询并安装即可：
查询aarch64的编译器有哪些？
```
sudo apt-cache search aarch64 | grep "GNU C"

cpp-9-aarch64-linux-gnu - GNU C preprocessor
cpp-aarch64-linux-gnu - GNU C preprocessor (cpp) for the arm64 architecture
g++-9-aarch64-linux-gnu - GNU C++ compiler (cross compiler for arm64 architecture)
g++-aarch64-linux-gnu - GNU C++ compiler for the arm64 architecture
gcc-9-aarch64-linux-gnu - GNU C compiler (cross compiler for arm64 architecture)
gcc-9-aarch64-linux-gnu-base - GCC, the GNU Compiler Collection (base package)
gcc-aarch64-linux-gnu - GNU C compiler for the arm64 architecture
cpp-10-aarch64-linux-gnu - GNU C preprocessor
cpp-8-aarch64-linux-gnu - GNU C preprocessor
g++-10-aarch64-linux-gnu - GNU C++ compiler (cross compiler for arm64 architecture)
g++-8-aarch64-linux-gnu - GNU C++ compiler (cross compiler for arm64 architecture)
gcc-10-aarch64-linux-gnu - GNU C compiler (cross compiler for arm64 architecture)
gcc-10-aarch64-linux-gnu-base - GCC, the GNU Compiler Collection (base package)
gcc-8-aarch64-linux-gnu - GNU C compiler (cross compiler for arm64 architecture)
gcc-8-aarch64-linux-gnu-base - GCC, the GNU Compiler Collection (base package)
```
通过查询知道支持的gcc包括gcc-aarch64-linux-gnu , gcc-10-aarch64-linux-gnu等，我们就安装这个gcc-10*编译器。

```
sudo apt-get install gcc-10-aarch64-linux-gnu
```
被安装在/usr/bin/目录下

```
/usr/bin/aarch64-linux-gnu-gcc-10
```
查看可以编译的目标平台是否正确
```
neil@TJ-YF-11W16P:~$ aarch64-linux-gnu-gcc-10 -v
Using built-in specs.
COLLECT_GCC=aarch64-linux-gnu-gcc-10
COLLECT_LTO_WRAPPER=/usr/lib/gcc-cross/aarch64-linux-gnu/10/lto-wrapper
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 10-20200411-0ubuntu1' --with-bugurl=file:///usr/share/doc/gcc-10/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-10 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --without-target-system-zlib --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=aarch64-linux-gnu --program-prefix=aarch64-linux-gnu- --includedir=/usr/aarch64-linux-gnu/include
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 10.0.1 20200411 (experimental) [master revision bb87d5cc77d:75961caccb7:f883c46b4877f637e0fa5025b4d6b5c9040ec566] (Ubuntu 10-20200411-0ubuntu1)
```

Target表示目标平台是aarch64 架构，Linux系统，符合预期。

## 问题
1. 出现'GLIBC_2.14' not found 问题
原因是由于编译的cgo引用的glibc的版本与运行环境的glibc的版本不匹配，由于是动态链接所以找不到对应的版本报错。
解决：通过静态链接将glibc的版本链接进去。

```
LDFLAGS := "-s -w -extldflags "-static""

# -extldflags  "-static" 就表示指定静态链接
```

## 参考链接

> http://blog.sina.com.cn/s/blog_43a59c7a0102wz7w.html


*名词解释*：
GNU:GNU不是一个公司名，而是一个软件项目名。它开发了许多应用程序。
GCC:GCC全称是 GNU C Compiler, 最早的时候就是一个c编译器。但是后来因为这个项目里边集成了更多其他不同语言的编译器，GCC就代表 the GNU Compiler Collection，所以表示一堆编译器的合集。
G++:G++则是GCC的c++编译器


