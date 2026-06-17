---
title: VulnHub Depth：1
date: 2026-04-16 19:48:00
categories: 
  - VulnHub
tags:
  - RCE
  - SSH 本地登录
  - /etc/sudoers 配置不当
  - 禁用防火墙
---

# VulnHub Depth: 1

项目地址：https://download.vulnhub.com/depth/DepthB2R.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69de4121f0cf5d89b0eca05f.png)

发现 8080 运行了 JSP，访问发现是 Tomcat

![](https://pic1.imgdb.cn/item/69de412ef76f4dd14b4936bd.png)

## RCE

### File listing checker

扫描目录发现 `/test.jsp`，功能是可以执行系统命令 `ls -l /tmp `

![](https://pic1.imgdb.cn/item/69de4181f76f4dd14b493878.png)

## 提取

### SSH 本地登录

执行 `ls -l /home` 命令查看用户家目录，发现 bill

![](https://pic1.imgdb.cn/item/69de4d74f76f4dd14b4984c9.png)

尝试 SSH 本地登录成功，可以使用当前 bill 用户执行命令

![](https://pic1.imgdb.cn/item/69de577af76f4dd14b49f41a.png)

### /etc/sudoers 配置不当

检查权限

```bash
ssh bill@localhost sudo -l
```

发现输出

```bash
(ALL) NOPASSWD: ALL
```

### 禁用防火墙

尝试直接反弹 root Shell 一直收不到，检查下是否开启了防火墙

```bash
ssh bill@localhost sudo iptables -L -n -v
```

虽然 `OUTPUT` 默认策略是 `ACCEPT`，但流量在遍历链的过程中，会被跳转到 `ufw-user-output`

![](https://pic1.imgdb.cn/item/69de6026f0cf5d89b0ecac74.png)

在 `ufw-user-output` 链的末尾，存在一条**无差别丢弃所有流量**的 `DROP` 规则

![](https://pic1.imgdb.cn/item/69de60fdf0cf5d89b0ecaca2.png)

上面这些内容如果因为输出格式看不懂可以扔给 Ai 解析

知道关键点后我们可以关闭防火墙

```bash
ssh bill@localhost sudo ufw disable
```

![](https://pic1.imgdb.cn/item/69de61b4f0cf5d89b0ecaccb.png)