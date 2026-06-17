---
title: 知攻善防实验室 Windows 应急响应——Web 2
date: 2026-05-07 10:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - Apache 日志
  - WebShell 查杀
  - RDP 溯源
  - QQ 溯源
  - 可疑文件查找
  - Windows 隐藏用户
---

# 知攻善防实验室 Windows 应急响应——Web 2

## 题目

小李在某单位驻场值守，深夜 12 点，甲方已经回家了，小李刚偷偷摸鱼后，发现安全设备有告警，于是立刻停掉了机器开始排查

## 应急响应

### 攻击者的两个 IP 地址（Apache 日志 + RDP 溯源）

桌面看到 PHPStudy，分析其 Apache 日志拿到第一个 IP 地址

工具：https://github.com/duckpigdog/blueteam-log-analyzer

![](https://pic1.imgdb.cn/item/69fd90ba353cb83ac7637f91.png)

排查安全日志拿到第二个 IP，同时发现的隐藏用户 hack887$

![](https://pic1.imgdb.cn/item/69fd941a353cb83ac76389c6.png)

### 攻击者的 WebShell 文件名（WebShell 查杀）

用 BLA 找一个访问 `system.php` 的 POST 请求

![](https://pic1.imgdb.cn/item/69fd97bc353cb83ac7639531.png)

访问文件，看内容很明显就是哥斯拉的木马

![](https://pic1.imgdb.cn/item/69fd9817353cb83ac76396dc.png)

### 攻击者的 webshell 密码（WebShell 查杀）

在上面已经给出，hack6618

### 攻击者的伪 QQ 号（QQ 溯源）

攻击者可能在系统中登陆过 QQ，而 QQ 会在每个用户登录后自动往当前用户的 `Documents\Tencent Files\` 目录下创建一个以 QQ 号命名的文件夹

因此可以搜索进入该目录，查看是否存在以 QQ 号命名的文件夹

![](https://pic1.imgdb.cn/item/69fd98c2353cb83ac7639a3f.png)

### 攻击者的伪服务器 IP 地址（可疑文件查找）

搜索 `*.exe` 发现内网穿透工具 frp

![](https://pic1.imgdb.cn/item/69fd9e2b353cb83ac763ec45.png)

打开文件所在目录，查看 frp 的配置文件拿到 IP 地址

![](https://pic1.imgdb.cn/item/69fd9e3d353cb83ac763ec76.png)

### 攻击者的服务器端口（可疑文件查找）

也都在上面

### 攻击者是如何入侵的（攻击链路溯源）

其 PHPStudy 部署的是 WordPress，但是在 Apache 日志中没有看到任何登录请求

所以排除 Web

![](https://pic1.imgdb.cn/item/69fd9f95353cb83ac763f118.png)

所以只能是 RDP 或者是通过其他服务进来的

![](https://pic1.imgdb.cn/item/69fda2c5353cb83ac763fb0a.png)

在小皮中看到有 FTP 服务，进去看看日志发现有内容

![](https://pic1.imgdb.cn/item/69fda19e353cb83ac763f767.png)

利用 BLA 分析看到就是从这里上传的 `system.php`

![](https://pic1.imgdb.cn/item/69fda48f353cb83ac76400b5.png)

### 攻击者的隐藏用户名（Windows 隐藏用户）

前面分析日志已经找到了