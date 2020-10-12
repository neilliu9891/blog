---
title: "Github ShowPicture"
date: 2020-10-05T13:18:30+08:00
tags: ["git"]
categories: ["技术杂文"]
draft: false
---

# github 显示照片
在访问别人的github项目时，阅读Readme经常发现不能显示照片。后来查看发现是由于picture所存储的图床地址不能通过dns解析

## 修改方法
1. 查找picture对应的服务器地址
![searchServerName](/images/image_2020-10-05-21-14-27.png)
2. 根据服务器地址，通过https://site.ip138.com/ 网址查询网页获取这个服务器地址所对应的ip地址
![showip](/images/image_2020-10-05-21-11-51.png)
3. 更新/etc/hosts 文件，将ip地址与服务器地址对应，写入/etc/hosts中。如果是windows系统则修改C:\Windows\System32\drivers\etc\hosts地址
![hosts](/images/image_2020-10-05-21-17-02.png)

参考链接：
> https://blog.csdn.net/qq_38232598/article/details/91346392
> https://blog.csdn.net/dplovel/article/details/107356603

查询网址：
> https://site.ip138.com/
