---
title: 知攻善防实验室 Linux 应急响应——1
date: 2026-04-11 14:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - Redis 远程连接
---

# 知攻善防实验室 Linux 应急响应——1

## 题目

小王急匆匆地找到小张，小王说：张哥，我 dev 服务器被黑了，快救救我！！

## 应急响应

### 黑客的 IP 地址（Redis 远程连接）

靶机需要关注其公众号获取

账号密码：defend/defend, root/defend

登录后先看看有无 Web 网站，看黑客是不是从 Web 打进来的

很显然是没有的，因为它没有 `/www` 目录

![](https://pic1.imgdb.cn/item/69d9d629757fdade5eace1c5.png)

再查看日志文件

发现 `.log` 的日志文件主要是以 vmware、yum 以及 Xorg 为主，这些主要是系统相关的

![](https://pic1.imgdb.cn/item/69d9d582757fdade5eace0e0.png)

从攻击者角度思考，一台靶机如果没有 Web 网站，那该怎么打进来呢？？？

没错，那就是扫描端口打系统服务

我们这里有两个重要系统服务，一个是 Redis，另一个则是 Samba

查看其中的 Samba 则是空内容

![](https://pic1.imgdb.cn/item/69d9d6ff757fdade5eace266.png)

于是去看 Redis 的，发现有一个日志文件

![](https://pic1.imgdb.cn/item/69d9d71c757fdade5eace26e.png)

打开过了一遍发现里面有记录历史连接，包括了连接的成功失败、IP 地址等等

于是过滤出连接成功的关键字 `Accepted`，发现总共就两个 IP 地址

我们不可能自己打自己吧，所以这题答案就是 `192.168.75.129`

![](https://pic1.imgdb.cn/item/69d9d795757fdade5eace290.png)


## CTF

### 遗留下的三个 flag（关键字过滤）

```bash
grep -inr flag{ /etc/
grep -inr flag{ ~
```

![](https://pic1.imgdb.cn/item/69d9d921757fdade5eace32d.png)