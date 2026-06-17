---
title: 知攻善防实验室 Windows 应急响应——近源渗透
date: 2026-04-25 10:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - 病毒文件分析
  - 可疑文件查找
---

# 知攻善防实验室 Windows 应急响应——近源渗透

## 题目

小王从某安全大厂被优化掉后，来到了某私立小学当起了计算机老师。某一天上课的时候，发现鼠标在自己动弹，又发现除了某台电脑，其他电脑连不上网络。感觉肯定有学生捣乱，于是开启了应急

## 应急响应

### 攻击者的外网 IP 地址（病毒文件分析）

在桌面上发现可疑文件，题目提示了是近源，所以可以排除 Web

同时靶机又没有通讯软件和邮件，排除钓鱼

![](https://pic1.imgdb.cn/item/69ec2a5d673fb9750a2f9825.png)

桌面上有 Word，结合学生上传 Word 交作业场景推测有宏病毒

将桌面上的 Word 文件全部打包成压缩包共享出来

![](https://pic1.imgdb.cn/item/69ec2b1e673fb9750a2f988b.png)

访问复制

![](https://pic1.imgdb.cn/item/69ec34c2673fb9750a2fc490.png)

微步云沙箱检测识别出木马

![](https://pic1.imgdb.cn/item/69ec3526673fb9750a2fc557.png)

拿到其 IP 地址

![](https://pic1.imgdb.cn/item/69ec3583673fb9750a2fc61c.png)

### 攻击者的内网跳板 IP 地址（病毒文件分析）

在桌面上发现一个图标为 bat 的 `phpStudy-修复` 文件

![](https://pic1.imgdb.cn/item/69ec3781673fb9750a2fcc64.png)

右键属性发现是链接的一个 `test.bat`

![](https://pic1.imgdb.cn/item/69ec379c673fb9750a2fcc75.png)

复制路径去查找，发现是隐藏文件，右键属性设置下就好

![](https://pic1.imgdb.cn/item/69ec37c2673fb9750a2fcc86.png)

右键编辑拿到 IP 地址

![](https://pic1.imgdb.cn/item/69ec37e5673fb9750a2fcc98.png)

### 攻击者使用的限速软件（可疑文件查找）

在 C 盘 `/Perflogs` 文件夹中发现有个文件夹一直循环，一直往里翻找到限速软件

![](https://pic1.imgdb.cn/item/69ec392c673fb9750a2fcde0.png)

### 攻击者的后门（可疑文件查找）

在 `C:\Windows\System32` 目录下发现了一个用 python 写的 exe 文件

![](https://pic1.imgdb.cn/item/69ec39af673fb9750a2fce2f.png)

微步识别为可疑文件

![](https://pic1.imgdb.cn/item/69ec39bd673fb9750a2fce4c.png)


## CTF

### 攻击者留下的 flag（可疑文件查找）

![](https://pic1.imgdb.cn/item/69ec3a4a673fb9750a2fcf4b.png)