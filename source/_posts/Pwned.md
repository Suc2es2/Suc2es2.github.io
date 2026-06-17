---
title: MazeSec Pwned
date: 2025-04-01 16:29:00
categories: 
  - MazeSec
tags:
  - 敏感信息泄露
  - 弱口令
  - /etc/sudoers 配置不当
  - Python 升级为交互式 Shell
  - Docker 组权限提权
---

# Pwned

项目地址：https://downloads.hackmyvm.eu/pwned.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a14026f72662027ad94299c.png)

## 敏感信息泄露

### /pwned_vuln

访问首页

![](https://pic1.imgdb.cn/item/6a14027b72662027ad94299e.png)

扫描目录发现一个 `.dic` 文件

![](https://pic1.imgdb.cn/item/6a14027b72662027ad94299e.png)

打开是一些路径

![](https://pic1.imgdb.cn/item/6a14036272662027ad9429d2.png)

对路径做扫描发现 `/pwned_vuln`

![](https://pic1.imgdb.cn/item/6a14037a72662027ad9429d8.png)

前端页面发现隐藏账号密码

![](https://pic1.imgdb.cn/item/6a14039d72662027ad9429e2.png)

## 弱口令

### FTP

FTP 登录成功

![](https://pic1.imgdb.cn/item/6a1403d972662027ad9429f7.png)

### SSH

发现有私钥

![](https://pic1.imgdb.cn/item/6a1403f772662027ad9429f8.png)

```
Wow you are here 
ariana won't happy about this note 
sorry ariana :(
```

拿到用户名后登录

![](https://pic1.imgdb.cn/item/6a14047372662027ad942a31.png)

## 提权

### /etc/sudoers 配置不当

检查权限发现给了一个 `.sh`

![](https://pic1.imgdb.cn/item/6a14042872662027ad9429fe.png)

这里看脚本内容是存在命令注入漏洞，使用 `selena` 运行脚本拿到 Shell

![](https://pic1.imgdb.cn/item/6a1404cd72662027ad942a4c.png)

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a14052d72662027ad942a87.png)

### Docker 组权限提权

继上面发现于 Docker 是属于同一用户组

组内用户运行 `docker run -v /:/mnt` 时，Docker 会将宿主机的整个根目录 `/` 挂载到容器内的 `/mnt` 目录

最后，在容器内执行 `chroot /mnt sh`，直接将当前运行的根目录切换到宿主机的根目录，从而获得对宿主机的完全控制权

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

![](https://pic1.imgdb.cn/item/6a14074b72662027ad942bae.png)

```
fb8d98be1265dd88bac522e1b2182140
4d4098d64e163d2726959455d046fd7c
```