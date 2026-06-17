---
title: VulnHub Noob：1
date: 2026-05-10 18:30:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - CTF
  - Steghide 隐写
---

# Noob：1

项目地址：https://download.vulnhub.com/noob/Noob.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a00ab1f72dba71658f0c450.png)

## 弱口令

### FTP

在上图可以看到，通过 FTP 可以访问两个文件：`welcome` 和 `cred.txt`

把这两个文件 `get` 出来

![](https://pic1.imgdb.cn/item/6a00ac4a72dba71658f0cb5c.png)

查看文件内容得到一组用户名密码

```
champ:password
```

![](https://pic1.imgdb.cn/item/6a00acf672dba71658f0cbdf.png)

访问 80 端口网站，使用上述凭据成功登录系统

点击 “About” 页面触发下载 `downloads.rar`

![](https://pic1.imgdb.cn/item/6a00ae0a72dba71658f0cbed.png)

得到两个图片和一个提示文件

https://pic1.imgdb.cn/item/6a00af1572dba71658f0cc0e.png

### SSH

在下面 CTF 之后利用解密出来的账号密码可以登录（注意改端口）

![](https://pic1.imgdb.cn/item/6a00b46572dba71658f0cc78.png)

## CTF

### Steghide 隐写

`.jpg` 图片解压出一个 `hint.py` 文件

![](https://pic1.imgdb.cn/item/6a00b12572dba71658f0cc39.png)

提示旋转一些单词

![](https://pic1.imgdb.cn/item/6a00b2c872dba71658f0cc4e.png)

解压另一个图片中的隐藏文件

![](https://pic1.imgdb.cn/item/6a00b39172dba71658f0cc5d.png)

从 `funny.bmp` 提取的 `user.txt` 内容为：

```
jgs:guvf bar vf n fvzcyr bar
```

尝试 ROT13 后成功得到

```
wtf:this one is a simple one
```

## 提权

### /etc/sudoers 配置不当

检查权限为三个 ALL，直接切换为 Root 用户

![](https://pic1.imgdb.cn/item/6a00b4fa72dba71658f0cc8d.png)