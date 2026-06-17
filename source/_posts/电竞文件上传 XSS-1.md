---
title: 某电竞俱乐部小程序文件上传 XSS
date: 2026-04-27 12:32:00
categories: 
  - SRC
tags:
  - XSS
  - 微信小程序
---

# 某电竞俱乐部小程序文件上传 XSS

## 前言

本次测试属于公益类型的 SRC

## XSS

点击头像处

![](https://pic1.imgdb.cn/item/69ef08b0fc83efdcc4c097a7.png)

点击修改头像

![](https://pic1.imgdb.cn/item/69ef08d8fc83efdcc4c097e4.png)

如果直接传 `.html` 后缀会被拦截，应该是限制了黑名单

![](https://pic1.imgdb.cn/item/69ef095bfc83efdcc4c09852.png)

将其后缀改为 PNG 绕过，上传成功回显路径

![](https://pic1.imgdb.cn/item/69ef090bfc83efdcc4c09810.png)

访问弹窗

![](https://pic1.imgdb.cn/item/69ef0936fc83efdcc4c0983a.png)