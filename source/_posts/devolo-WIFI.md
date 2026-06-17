---
title: 实战——攻陷海外某单位监控及 WIFI 设备后台
date: 2026-04-06 16:15:00
categories: 
  - 渗透测试
tags:
  - 任意文件读取
  - 弱口令
---

# 实战——攻陷海外某单位监控及 WIFI 设备后台

## 信息收集

信息收集找到目标站点的一个监控后台

![](https://pic1.imgdb.cn/item/69d4ca9fcfae5fcb267a69ee.png)

网上找弱口令无果，于是搜索 NDay 成功找到一个任意文件读取漏洞

![](https://pic1.imgdb.cn/item/69d4c981cfae5fcb267a68bb.png)

## 任意文件读取

尝试读取 `/etc/shadow` 文件，没想到成功了，可以看出这个监控系统身份是 root 级别的

![](https://pic1.imgdb.cn/item/69d4cb17cfae5fcb267a6a97.png)

## 弱口令

后续爆破 admin 的哈希登录进监控系统，同时意外发现该站点还有一个 WIFI 后台

登录进去可以修改 WIFI 密码等等

![](https://pic1.imgdb.cn/item/69d4cbc5cfae5fcb267a6bac.png)
![](https://pic1.imgdb.cn/item/69d4cc06cfae5fcb267a6bee.png)
![](https://pic1.imgdb.cn/item/69d4cc1acfae5fcb267a6c10.png)
