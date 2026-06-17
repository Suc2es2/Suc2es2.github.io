---
title: 知攻善防实验室 Linux 应急响应——2
date: 2026-04-11 18:18:00
categories: 
  - 应急响应
  - 知攻善防
tags:
  - 宝塔重置密码
  - 宝塔日志
  - PHPEMS 弱口令
  - PHPEMS RCE
  - 蚁剑流量特征
  - WebShell 流量特征
  - 关键字过滤
  - 环境变量
---

# 知攻善防实验室 Linux 应急响应——2

## 题目

前景需要：看监控的时候发现 webshell 告警，领导让你上机检查你可以救救安服仔吗！！

## 应急响应

### 破解 CentOS 7

官方提供的靶机用户名和密码无效，需要我们重置密码

![](https://pic1.imgdb.cn/item/69d9f265757fdade5ead4b4d.png)

重启来到这个页面按 e

![](https://pic1.imgdb.cn/item/69d9f362757fdade5ead4b92.png)

进入到编辑页面，找到这个 `ro`

![](https://pic1.imgdb.cn/item/69d9f3cc757fdade5ead4bc2.png)

改为 `rw init=/sysroot/bin/sh`，再按 Ctrl + X 使用单用户模式启动

![](https://pic1.imgdb.cn/item/69d9f3f1757fdade5ead4bec.png)

使用下面的命令访问系统

```bash
chroot /sysroot
```

然后更新 root 密码重启系统

![](https://pic1.imgdb.cn/item/69d9f46d757fdade5ead4c1d.png)

成功登录进去

![](https://pic1.imgdb.cn/item/69d9f4b4757fdade5ead4c26.png)

### 攻击者 IP（宝塔重置密码 + 宝塔日志）

提示了 WebShell，那就看看 `/var`

随便没有 `www` 目录，但是有 `bt_setupPath.conf` 文件，这是宝塔所用的

![](https://pic1.imgdb.cn/item/69d9f54d757fdade5ead4c51.png)

宝塔默认安装在 `/www` 下面，去看看发现有这个目录

![](https://pic1.imgdb.cn/item/69d9f5b4757fdade5ead4c6f.png)

这里看不到中文字体，可以让 AI 指导操作就行

```
uysycv5w
123456
```

![](https://pic1.imgdb.cn/item/69d9f61a757fdade5ead4c89.png)

再获取下宝塔的 Web 端位置

![](https://pic1.imgdb.cn/item/69d9f673757fdade5ead4ca1.png)

浏览器访问登录

![](https://pic1.imgdb.cn/item/69d9f6e6757fdade5ead4cc6.png)


成功拿到攻击者 IP 地址

```
192.168.20.1
```

![](https://pic1.imgdb.cn/item/69d9f759757fdade5ead4ceb.png)

### 攻击者修改的管理员明文密码（PHPEMS 弱口令）

在宝塔中发现部署了一个网站

![](https://pic1.imgdb.cn/item/69d9fab8757fdade5ead4e16.png)

且里面有完整的文件内容

![](https://pic1.imgdb.cn/item/69d9faf1757fdade5ead4e25.png)

进入宝塔的站点配置页，将靶机 IP 添加进域名列表，使站点能被访问

![](https://pic1.imgdb.cn/item/69d9f95c757fdade5ead4d8a.png)

测试访问成功

![](https://pic1.imgdb.cn/item/69d9fa83757fdade5ead4de0.png)

登录需要账号密码，但是我们没有，所以选择从数据库里面去找

![](https://pic1.imgdb.cn/item/69d9fb4f757fdade5ead4e43.png)

点击管理

![](https://pic1.imgdb.cn/item/69da0221757fdade5ead8278.png)

使用刚刚修改的账号密码登录 phpMyAdmin 后台

![](https://pic1.imgdb.cn/item/69d9fe66757fdade5ead4f10.png)

查看用户名拿到账号密码，PHPEMS 系统默认账号是：peadmin:peadmin

参考代审文章：https://zone.huoxian.cn/d/876-phpems

![](https://pic1.imgdb.cn/item/69d9fed6757fdade5ead4f33.png)

破解 MD5 即可

```
Network@2020
```

![](https://pic1.imgdb.cn/item/69da00df757fdade5ead821c.png)

### 提交第一次连接 WebShell 的 URL（PHPEMS RCE）

使用前面拿到的账号密码登录后台

![](https://pic1.imgdb.cn/item/69da0246757fdade5ead8282.png)

点击后台管理后根据前面的代审文章找到了 WebShell

![](https://pic1.imgdb.cn/item/69da0277757fdade5ead8293.png)

该标签对应的位置是 "注册页面"，因此 "注册页面" 的 url 就是 webshell 的 url

即 `http://10.0.6.150/index.php?user-app-register`


### 提交 Webshell 连接密码（PHPEMS RCE）

上面已经给出了，就是 `Network2020`

### 提交数据包的 flag1（蚁剑流量特征）

在 `/root` 目录下找到对应的数据包，下载下来

![](https://pic1.imgdb.cn/item/69da0436757fdade5ead8336.png)

过滤 HTTP 协议并追踪流

包含 `@ini_set("display_errors", "0")、set_time_limit(0)、@ini_get("open_basedir")`

很明显的蚁剑特征

![](https://pic1.imgdb.cn/item/69da04b5757fdade5ead8364.png)

慢慢翻数据包找到 flag1

```
flag1{Network@_2020_Hack}
```

![](https://pic1.imgdb.cn/item/69da05d8757fdade5ead83bb.png)

### 提交攻击者使用的后续上传的木马文件名称（WebShell 流量特征）

访问 `version.php` 返回 404，访问 `version2.php` 返回 200

所以木马文件名称为 `version2.php`

![](https://pic1.imgdb.cn/item/69da0632757fdade5ead83db.png)

## CTF

### 提交攻击者隐藏的 flag2（关键字过滤）

在 `/www/wwwroot/127.0.0.1/.api/alinotify.php` 中

```
flag{bL5Frin6JVwVw7tJBdqXlHCMVpAenXI9In9}
```

![](https://pic1.imgdb.cn/item/69da071f757fdade5ead841e.png)


### 提交攻击者隐藏的 flag3（环境变量）

```
flag{5LourqoFt5d2zyOVUoVPJbOmeVmoKgcy6OZ}
```

![](https://pic1.imgdb.cn/item/69da0749757fdade5ead8433.png)