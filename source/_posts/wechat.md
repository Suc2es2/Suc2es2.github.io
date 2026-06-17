---
title: 微信小程序抓包教程
date: 2026-02-01 16:39:00
categories: 
  - SRC
tags:
  - 微信小程序
  - 抓包
---

# 微信小程序抓包教程

## 前言
本次使用的工具是 Proxifier + Yakit

## Proxifier
去官网下载安装就行，然后注册码谷歌一下，邮箱随便填就可以破解了
![](https://pic1.imgdb.cn/item/69d7558115908d0c2fa87bcc.png)

运行程序，选择 Profile --> Proxy Servers，配置如下
![](https://pic1.imgdb.cn/item/69d7565515908d0c2fa87da0.png)
![](https://pic1.imgdb.cn/item/69d7564015908d0c2fa87d6d.png)

然后再添加规则，Profile --> Proxification Rules，配置如下
![](https://pic1.imgdb.cn/item/69d7569315908d0c2fa87e54.png)
![](https://pic1.imgdb.cn/item/69d756cb15908d0c2fa87edb.png)

## Yakit
首先要启动劫持，下载证书
![](https://pic1.imgdb.cn/item/69d755c215908d0c2fa87c57.png)

下载到本地保存
![](https://pic1.imgdb.cn/item/69d755d115908d0c2fa87c75.png)

删除 `.pem` 后缀
![](https://pic1.imgdb.cn/item/69d755eb15908d0c2fa87cad.png)

双击安装然后选择安装到受信任的机构中
![](https://pic1.imgdb.cn/item/69d7561715908d0c2fa87d0a.png)

然后重启所有上面两个应用 + 微信即可抓包了（注意：必须关梯子才行！！！）
![](https://pic1.imgdb.cn/item/69d756f815908d0c2fa87f45.png)
