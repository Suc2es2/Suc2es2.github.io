---
title: VulnHub Node：1
date: 2026-05-03 18:30:00
categories: 
  - VulnHub
tags:
  - JavaScript 接口泄露
  - CTF
  - ZIP 密码爆破
  - 内核提权
---

# Node：1

项目地址：https://download.vulnhub.com/node/Node.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69f72e4d98d7db9e978282a5.png)

访问 3000 端口

![](https://pic1.imgdb.cn/item/69f7a13f98d7db9e9783ec4c.png)

## JS 接口泄露

### /api/users

通过插件 XMCVE-WebRecon 扫描发现

![](https://pic1.imgdb.cn/item/69f7a40198d7db9e9783ec76.png)

访问拿到用户名以及密码，但是都不是管理员账号

![](https://pic1.imgdb.cn/item/69f7a46d98d7db9e9783ec7d.png)

访问另一个接口拿到管理员用户名 myP14ceAdm1nAcc0uNT

![](https://pic1.imgdb.cn/item/69f7a5bd98d7db9e9783ec8c.png)

成功破解出账号密码

![](https://pic1.imgdb.cn/item/69f7a62c98d7db9e9783ec91.png)

## ZIP 密码爆破

### myplace.backup

成功登录后台，有一个下载备份按钮

![](https://pic1.imgdb.cn/item/69f7a67798d7db9e9783ec97.png)

下载文件发现是 Base64 编码了

![](https://pic1.imgdb.cn/item/69f7a77198d7db9e9783eca6.png)

解码发现是 ZIP 压缩文件

![](https://pic1.imgdb.cn/item/69f7a7b198d7db9e9783ecaa.png)

通过 CTF 的一些压缩包密码爆破工具爆出密码是 `magicword`

![](https://pic1.imgdb.cn/item/69f7a88b98d7db9e9783ecb8.png)

拿到网站源码

![](https://pic1.imgdb.cn/item/69f7a8c398d7db9e9783ecba.png)

在 `app.js` 中翻找到系统用户的密码

![](https://pic1.imgdb.cn/item/69f7a96698d7db9e978402d6.png)

成功连接登录上

```bash
mark
5AYRft73VtFpc84k
```

![](https://pic1.imgdb.cn/item/69f7aa0b98d7db9e978402df.png)

## 提权

### BPF（CVE-2016-4557）

查看其内核版本是 Linux 4.4

![](https://pic1.imgdb.cn/item/69f7aa5898d7db9e978402e2.png)

搜索到一个内核提权漏洞

![](https://pic1.imgdb.cn/item/69f7ab6098d7db9e978402f3.png)

提权成功

![](https://pic1.imgdb.cn/item/69f7abda98d7db9e978402f8.png)