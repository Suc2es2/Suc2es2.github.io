---
title: 某电竞俱乐部小程序投诉陪玩与客服对话文件上传 XSS（两个）
date: 2026-05-11 12:32:00
categories: 
  - SRC
tags:
  - XSS
  - 微信小程序
---

# 某电竞俱乐部小程序投诉陪玩与客服对话文件上传 XSS（两个）

## 前言

本次测试属于公益类型的 SRC

## XSS

在投诉页面发现可以上传图片

![](https://pic1.imgdb.cn/item/6a01a929514fff0f73776d8e.png)

选择上传一个含有 XSS 代码的 HTML 文件，将后缀和类型改为 PNG

![](https://pic1.imgdb.cn/item/6a01a96c514fff0f73776da6.png)

上传成功，访问响应包里的回显路径

![](https://pic1.imgdb.cn/item/6a01a997514fff0f73776daa.png)

成功弹窗

![](https://pic1.imgdb.cn/item/6a01a9de514fff0f73776dae.png)

另外一个则是在与客服对话中

![](https://pic1.imgdb.cn/item/6a01ac5f514fff0f73776e1e.png)

也是一样的，在响应包中可以看到 URL

![](https://pic1.imgdb.cn/item/6a01ac7b514fff0f73776e27.png)