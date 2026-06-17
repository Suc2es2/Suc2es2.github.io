---
title: 知攻善防实验室 Windows 应急响应——Web 3
date: 2026-05-09 10:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - RDP 溯源
  - Windows 隐藏用户
  - 可疑文件查找
  - 任务计划程序
  - 数据库
---

# 知攻善防实验室 Windows 应急响应——Web 3

## 题目

小苕在省护值守中，在灵机一动情况下把设备停掉了，甲方问：为什么要停设备？小苕说：我第六感告诉我，这机器可能被黑了

## 应急响应

### 攻击者的 IP 地址（RDP 溯源）

分析 Windows 安全日志拿到攻击 IP

![](https://pic1.imgdb.cn/item/69fea0fe353cb83ac7673f19.png)

### 隐藏用户名称（Windows 隐藏用户）

BLA 工具一把梭哈

![](https://pic1.imgdb.cn/item/69fea1c7353cb83ac76741e0.png)

## CTF

### 黑客遗留下的 3 个 flag（可疑文件查找 + 任务计划程序 + 数据库）

翻一下 hack6118$ 的家目录，找到一个名为 system.bat 的批处理文件

![](https://pic1.imgdb.cn/item/69fea499353cb83ac7674c0c.png)

改个名打开拿到 flag

![](https://pic1.imgdb.cn/item/69fea4c7353cb83ac7674cb5.png)

查看任务计划拿到第二个 flag

![](https://pic1.imgdb.cn/item/69fea50b353cb83ac7674d94.png)

最后一个在数据库中

![](https://pic1.imgdb.cn/item/69fea55f353cb83ac7674edb.png)