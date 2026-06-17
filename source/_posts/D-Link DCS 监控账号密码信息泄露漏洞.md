---
title: 实战——攻陷海外某政府官员家庭监控
date: 2026-04-09 16:15:00
categories: 
  - 渗透测试
tags:
  - 任意文件读取
  - 弱口令
---

# 实战——攻陷海外某政府官员家庭监控

## 信息收集

对这名官员手下的一台服务器做信息收集，发现开放了 443 端口

访问却需要密码

![](https://pic1.imgdb.cn/item/69d7342815908d0c2fa825d5.png)

使用测绘引擎去指纹识别发现是部署了一个嵌入式 Web 服务器

![](https://pic1.imgdb.cn/item/69d7357b15908d0c2fa82785.png)

## 任意文件读取

找到一个 NDay 直接打，成功拿到账号密码

![](https://pic1.imgdb.cn/item/69d735ed15908d0c2fa828bd.png)

成功黑入监控系统

![](https://pic1.imgdb.cn/item/69d7370215908d0c2fa82bed.png)