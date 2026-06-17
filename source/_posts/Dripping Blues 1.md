---
title: VulnHub Dripping Blues：1
date: 2026-06-09 18:32:00
categories: 
  - VulnHub
tags:
  - 文件包含
  - 弱口令
  - SUID 提权
  - PwnKit (CVE-2021-3560)
---

# Dripping Blues：1

项目地址：https://download.vulnhub.com/drippingblues/drippingblues.ova

是一个打包好的镜像文件

**注意：导入 ovf 时一定要选择我已移动，否则没网**

使用无影扫描出开放服务及系统版本

![](https://pic1.imgdb.cn/item/6a29010deae595505fc067c0.png)

## 文件包含

### drip

访问 80 端口首页没东西，扫描出 `robots.txt` 文件

![](https://pic1.imgdb.cn/item/6a2901e2eae595505fc06890.png)

这里提示 SSH 连接密码，但试过了没用

![](https://pic1.imgdb.cn/item/6a2901f5eae595505fc0689a.png)

匿名访问 FTP 服务有一个压缩文件，爆破出密码是 `072528035`

![](https://pic1.imgdb.cn/item/6a2903a8eae595505fc0698a.png)

解压之后获得 `respectmydrip.txt` 文件和 `secret.zip` 文件

`respectmydrip.txt` 文件是一条信息

```
just focus on "drip"
```

结合之前 `robots.txt` 给出的路径，打一个文件包含漏洞去读取 `/etc/dripispowerful.html` 下的内容

![](https://pic1.imgdb.cn/item/6a2904d2eae595505fc06b5c.png)

查看源代码发现注释，给出了密码

![](https://pic1.imgdb.cn/item/6a2904e6eae595505fc06b69.png)

以及用户名

![](https://pic1.imgdb.cn/item/6a290b24eae595505fc0c099.png)

## 弱口令

### SSH

thugger 用户可以连接

![](https://pic1.imgdb.cn/item/6a290b0aeae595505fc0c078.png)

## 提权

### SUID 提权

Polkit (PolicyKit)：Linux 系统中用于控制系统范围权限的组件，它允许非特权进程与特权进程进行安全的通信

`pkexec`：Polkit 提供的一个命令行工具，功能类似于 `sudo`。它允许授权用户以其他用户的身份执行程序

![](https://pic1.imgdb.cn/item/6a290c27eae595505fc0f6d9.png)

### PwnKit (CVE-2021-3560)

一个普通用户想要执行特权操作（例如：创建一个新用户、修改系统时间）时，流程如下：

- 用户进程通过 D-Bus（Linux 的进程间通信总线）向特权服务发送请求

- 特权服务自己没有权限决定，于是向 Polkit 守护进程 询问：“这个用户有权做这件事吗？”

- Polkit 查询规则，发现需要密码验证

- Polkit 启动一个认证代理，弹出一个对话框要求输入密码。

- 用户输入正确的密码，Polkit 验证通过，通知特权服务执行操作

如果攻击者在第 2 步（Polkit 正在向系统查询该用户的 UID 和权限）时，突然强行杀掉（kill -9）了自己发起的请求进程，会发生什么？

- 正常的预期： Polkit 发现请求者死了，应该取消这次操作，返回错误

- 实际的代码缺陷： 当 `polkit-daemon` 收到 "客户端已断开连接" 的信号时，它会中断正在进行的身份验证查询，但错误地将这次中断的请求标记为 "已授权"，并通知特权服务继续执行

POC：https://github.com/Almorabea/Polkit-exploit

上传到靶机上执行并拿到 root 权限

![](https://pic1.imgdb.cn/item/6a290e7eeae595505fc0fcf6.png)

成功

![](https://pic1.imgdb.cn/item/6a290e97eae595505fc0fcff.png)