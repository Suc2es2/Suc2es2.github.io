---
title: MazeSec Hommie
date: 2025-04-01 17:32:00
categories: 
  - MazeSec
tags:
  - 弱口令
  - 环境变量劫持
---

# Hommie

项目地址：https://downloads.hackmyvm.eu/hommie.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1f90fdb69adaba3f8af68d.png)

## 弱口令

### FTP

FTP 存在匿名访问

![](https://pic1.imgdb.cn/item/6a1f925eb69adaba3f8af6e9.png)

下载文件下来得到这样一条信息

![](https://pic1.imgdb.cn/item/6a1f935eb69adaba3f8af759.png)

### TFTP

使用 nmap 再扫描一遍 UDP 协议的，但是因为 UDP 很慢，所以加上端口限制

- `--top-ports 40`：仅扫描 Nmap 内置列表中最常见的 40 个端口

```bash
nmap -sU --top-ports 40 192.168.125.12
```

可以看到开放了 TFTP

![](https://pic1.imgdb.cn/item/6a1f965ab69adaba3f8af83a.png)

连接上去了根据前面的提示信息直接获取该文件

![](https://pic1.imgdb.cn/item/6a1f9746b69adaba3f8af89e.png)

### SSH

使用私钥连接该用户

![](https://pic1.imgdb.cn/item/6a1f9897b69adaba3f8b177a.png)

## 提权

### 环境变量劫持

查找具有 SUID 权限的文件，发现 `/opt/showMetheKey`

![](https://pic1.imgdb.cn/item/6a1f98f1b69adaba3f8b179a.png)

运行拿到密钥

![](https://pic1.imgdb.cn/item/6a1f992fb69adaba3f8b17ab.png)

经过验证发现这就是我们当前用户的密钥

![](https://pic1.imgdb.cn/item/6a1f99b5b69adaba3f8b17df.png)

Python 开启 HTTP 服务

![](https://pic1.imgdb.cn/item/6a1f9a4eb69adaba3f8b1966.png)

把文件拷贝下来

![](https://pic1.imgdb.cn/item/6a1f9a69b69adaba3f8b1969.png)

反编译可以看到该文件执行的命令是读取 `$HOME/.ssh/id_rsa`

![](https://pic1.imgdb.cn/item/6a1f9ab3b69adaba3f8b1980.png)

字符串提取也能识别到，应该是我看漏了

![](https://pic1.imgdb.cn/item/6a1f9b24b69adaba3f8b1999.png)

所以我们可以打环境变量劫持，让它去读取 root 用户的密钥

![](https://pic1.imgdb.cn/item/6a1f9b5db69adaba3f8b19a7.png)

成功提权

![](https://pic1.imgdb.cn/item/6a1f9b97b69adaba3f8b19b8.png)

```
Imnotroot
Imnotbatman
```