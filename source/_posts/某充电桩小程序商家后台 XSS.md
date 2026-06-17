---
title: 某充电桩小程序商家后台 XSS
date: 2026-05-20 00:39:00
categories: 
  - SRC
tags:
  - 微信小程序
  - XSS
---

# 某充电桩小程序 XSS

## 前言

本次测试属于公益类型的 SRC

## XSS

设置管理是查看功能，只能测越权，可惜有鉴权

![](https://pic1.imgdb.cn/item/6a0c9178a0971b8b02a4b3c3.png)

添加站点中有一个上传图片

![](https://pic1.imgdb.cn/item/6a0c90baa0971b8b02a4b3b0.png)

上传 XSS 图片，结果回显路径给咱自动添加了 `.html` 后缀

![](https://pic1.imgdb.cn/item/6a0c90e6a0971b8b02a4b3b8.png)

还有这好事？

![](https://pic1.imgdb.cn/item/6a0c9120a0971b8b02a4b3be.png)