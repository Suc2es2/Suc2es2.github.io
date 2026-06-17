---
title: 知攻善防实验室 Windows 应急响应——挖矿事件
date: 2026-05-05 10:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - RDP 溯源
  - 后台进程找恶意文件
  - 任务计划程序
  - 配置文件
---

# 知攻善防实验室 Windows 应急响应——挖矿事件

## 题目

机房运维小陈，下班后发现还有工作没完成，然后上机器越用越卡，请你帮他看看原因

## 应急响应

### 攻击者的 IP 地址（RDP 溯源）

在桌面上没有看到部署网站类的软件，所以先排除掉 Web

![](https://pic1.imgdb.cn/item/69fa12382356c9fc91939464.png)

查看开放端口情况发现有开启 445 以及 3389

![](https://pic1.imgdb.cn/item/69fa12992356c9fc91939484.png)

找事件 ID 为 4625 的登录成功日志，成功拿到 IP

![](https://pic1.imgdb.cn/item/69fa3fd82356c9fc919400d7.png)

### 攻击者开始攻击的时间（RDP 溯源）

攻击者开始攻击的时间即为 RDP 爆破开始的时间

这里我用自己二开的 BLA 扫描出时间戳

```
2024-05-21 20:25:22
```

![](https://pic1.imgdb.cn/item/69fac5845b6fdfa53a246fd1.png)

### 攻击者攻击的端口（RDP 溯源）

RDP 默认端口为 3389

### 挖矿程序的 md5（后台进程找恶意文件）

挖矿病毒特征就是 CPU 占用率 100%，任务管理器里找占用最高的就是

![](https://pic1.imgdb.cn/item/69fac61e5b6fdfa53a24703d.png)

### 后门脚本的 md5（任务计划程序）

重启靶机会执行一段 cmd 命令

![](https://pic1.imgdb.cn/item/69fafe9d4498ed47aaabc4ee.png)

打开任务计划程序找到恶意的 `.bat` 文件

![](https://pic1.imgdb.cn/item/69fb03394498ed47aaabc943.png)

定位到具体文件

![](https://pic1.imgdb.cn/item/69fb08d84498ed47aaabceb4.png)

### 矿池地址（配置文件）

在挖矿病毒的根目录中有一个 `config,json` 文件，打开就能看到地址

`c3pool.org` 也叫猫池

![](https://pic1.imgdb.cn/item/69fb09cd4498ed47aaabcf98.png)

### 钱包地址（配置文件）

同样也是在这个文件中找到，去猫池上查找这个地址

![](https://pic1.imgdb.cn/item/69fb0a374498ed47aaabd032.png)