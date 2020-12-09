---
title: "Golang Private Lab"
date: 2020-10-12T19:28:21+08:00
tags: ["golang","ssh","git"]
categories: ["技术杂文"]
#draft: true
---

# Golang Private Lab

从gitlab私有仓库获取golang依赖包
例如golang go.mod文件中有如下配置信息
```
module agent

go 1.13

require (
        ...
        moove/libvirt/libvirt-go v1.0.0
)

replace moove/libvirt/libvirt-go => 10.0.45.221/moove/libvirt-go v1.0.5 # 替换成本地库
```
此时就需要配置10.0.45.221 这个gitlab仓库作为golang的私有仓库下载libvirt包

## 配置通过ssh 访问gitlab

1. 生成ssh公私钥对。

```
ssh-keygen -t rsa -C "example@example.com" -b 4096 #-t 表示秘钥类型 -C comment信息
```
- 秘钥名称可以默认，也可以自定义，如果自定义参考步骤5
![ssh-keygen](/images/image_2020-10-12-20-05-29.png)
passphrase 是指使用这个秘钥时需要的认证密码，一般我都不会再设置了，省的麻烦，毕竟是自己的电脑相对安全一些。如果非个人电脑建议配置简单秘钥。

- 复制~/.ssh/ 目录下生成的公钥信息到系统剪切板
![publickey](/images/image_2020-10-12-20-13-24.png)

2. 登录gitLab账户，找到SSH选项，配置公钥信息

![ssh秘钥](/images/image_2020-10-12-19-57-18.png)

3. 复制粘贴公钥到空白区域，填写title信息，title不重要，可以随便起

![paste](/images/image_2020-10-12-20-15-00.png)
![paste2](/images/image_2020-10-12-20-15-52.png)

4. 添加秘钥即可Add Key

5. 如果生成的秘钥名称不是默认的秘钥名称，则需要进行如下操作，将秘钥信息纳管到ssh-agent服务中，并且需要在~/.ssh/config 文件中配置服务器地址信息,config文件没有创建
    - 编写config文件如下
    ```
    Host github.com                                                                                                        │~                              │~
	HostName github.com                                                                                                    │~                              │~
	PreferredAuthentications publickey                                                                                     │~                              │~
	IdentityFile ~/.ssh/11111
    ```
	Host: 作为此配置的名称，可以随便命名。
	HostName: 服务地址，如上就是github.com, 也可以直接配置ip地址
	IdentityFile: 表示ssh生成的私钥文件路径
    - 执行如下命令
    ```
    eval $(ssh-agent -s)
    ssh-add ~/.ssh/other_id_rsa
    ```
    other_id_rsa 替换成自己的私钥文件

---

如果通过如下命令能够正确返回登录的用户信息，则说明配置成功了
```
ssh -T git@github.com
```
此时，我们就能够通过git git@example/example.git 命令以ssh的方式下载代码了。

## git 配置

```
git config --global url."git@10.0.45.221:".insteadof "http://10.0.45.221/"
```
此处的--global配置也可以改成--local，只有某个项目使用这个替换方式,即将http访问数据包的方式修改为通过ssh访问

## golang 环境配置

```
go env -w GO111MODULE=on  # 重点，启用go mod
go env -w GOPRIVATE="10.0.45.221" # 使用私有库，
go get -u -v -insecure 10.0.45.221/moove/libvirt-go@latest # 获取libvirt-go的库
```
**提示：go get 操作 -insecure表示使用http方式请求，lastest表示获取最新版本，但是可能go.mod中需要的并不是lastest版本，所以此处要指定对版本。操作的目的是手动先下载下来版本，之后go build时就不会自动下载了。**
**go build去下载时都是采用https的方式，由于private lab可能不支持https所以导致下载失败。提示类似错误：go: moove/uni-network/ovn-store/go-ovn@v1.0.2: unrecognized import path "10.0.45.221/moove/uni-network/ovn-store/go-ovn" (https fetch: Get https://10.0.45.221/moove/uni-network/ovn-store/go-ovn?go-get=1: dial tcp 10.0.45.221:443: connect: no route to host)**

![go env](/images/image_2020-10-12-20-36-49.png)
重点是配置开启MODULE模式，以及配置私有golang仓库的地址
