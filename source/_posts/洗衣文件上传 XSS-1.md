---
title: 某洗衣小程序文件上传 XSS
date: 2026-03-01 16:39:00
categories: 
  - SRC
tags:
  - XSS
  - 微信小程序
---

# 某洗衣小程序文件上传 XSS

## 前言

本次测试属于公益类型的 SRC

## XSS

在上传头像处上传一个 PNG 类型的文件，里面是 HTML 类型的 XSS 代码并抓取上传头像的流量包

![](https://pic1.imgdb.cn/item/69d74a2f15908d0c2fa869ee.png)

改为后缀 `.htm` 发现可以上传

![](https://pic1.imgdb.cn/item/69d74a8a15908d0c2fa86a53.png)

在历史请求包中可以看到文件上传的路径

分别是 `ip` 字段和 `fileUrl` 字段

![](https://pic1.imgdb.cn/item/69d74ac315908d0c2fa86dd3.png)

拼接起来到网页上访问弹窗

![](https://pic1.imgdb.cn/item/69d74aef15908d0c2fa86e0d.png)
