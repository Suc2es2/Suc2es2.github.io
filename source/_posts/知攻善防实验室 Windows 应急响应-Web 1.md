---
title: 知攻善防实验室 Windows 应急响应——Web 1
date: 2026-05-06 10:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - WebShell 查杀
  - Apache 日志
  - Windows 隐藏用户
  - 样本分析
---

# 知攻善防实验室 Windows 应急响应——Web 1

## 题目

小李在值守的过程中，发现有 CPU 占用飙升，出于胆子小，就立刻将服务器关机，并找来正在吃苕皮的 hxd 帮他分析，这是他的服务器系统，请你找出以下内容，并作为通关条件

## 应急响应

### 攻击者的 Shell 密码

还是那句话，有 Web 找 Web，没 Web 看系统服务

打开虚拟机发现有 PHPStudy

![](https://pic1.imgdb.cn/item/69fd7bbf353cb83ac7633c90.png)

打开网站是 EMLOG

![](https://pic1.imgdb.cn/item/69fd89b2353cb83ac7636a3d.png)

找到一个 NDay 复现博客：https://blog.csdn.net/W13680336969/article/details/137267677

定位到具体的上传路径

![](https://pic1.imgdb.cn/item/69fd8a5e353cb83ac7636c84.png)

使用 D 盾查杀到木马

![](https://pic1.imgdb.cn/item/69fd7c37353cb83ac7633df1.png)

打开文件发现是冰蝎的马子，密码是默认密码

![](https://pic1.imgdb.cn/item/69fd7cc7353cb83ac7633fc5.png)

### 攻击者的 IP 地址（Apache 日志）

去翻小皮中 Apache 的日志，发现一个日志存在大量请求访问

![](https://pic1.imgdb.cn/item/69fd7d50353cb83ac763418b.png)

打开就能看到 IP 地址

![](https://pic1.imgdb.cn/item/69fd7dad353cb83ac76342d0.png)

### 攻击者的隐藏账户名称（Windows 隐藏用户）

打开计算机管理，在本地用户和组中找到隐藏用户

![](https://pic1.imgdb.cn/item/69fd8492353cb83ac7635783.png)

### 攻击者挖矿程序的矿池域名（样本分析）

在隐藏用户 hack168 的桌面文件夹中找到一个挖矿程序（一定不要运行！！）

![](https://pic1.imgdb.cn/item/69fd8560353cb83ac7635a93.png)

扔到在线沙箱分析是用的 PyInstaller 工具打包的

![](https://pic1.imgdb.cn/item/69fd8821353cb83ac7636447.png)

使用 Pyinstxtractor 提取 `.pyc` 文件出来

![](https://pic1.imgdb.cn/item/69fd88be353cb83ac7636689.png)

在线网站反编译 `.pyc` 文件拿到域名

![](https://pic1.imgdb.cn/item/69fd8919353cb83ac76367fc.png)

