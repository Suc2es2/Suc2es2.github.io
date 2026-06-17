---
title: VulnHub Thoth Tech：1
date: 2026-06-15 18:44:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - /etc/sudoers 配置不当
  - find 提权
---

# Thoth Tech：1

项目地址：https://download.vulnhub.com/thothtech/Thoth-Tech.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a317696b67c7e4f4a875bb1.png)

## 弱口令

### FTP

前面 NMap 扫描出来了，使用 `ftp:ftp` 登录即可

![](https://pic1.imgdb.cn/item/6a3176e2b67c7e4f4a875bd6.png)

`get` 下来后查看拿到用户名提示

![](https://pic1.imgdb.cn/item/6a317739b67c7e4f4a875bfc.png)

### SSH

爆破 SSH，拿到密码是 `babygirl1`

![](https://pic1.imgdb.cn/item/6a31777cb67c7e4f4a875c19.png)

## 提权

### /etc/sudoers 配置不当

检查权限，可以使用 `find` 命令

### find 提权

```bash
sudo find . -exec /bin/sh \; -quit
```

- `-exec /bin/sh \`;：对每一个查找到的结果，执行 `/bin/sh`

- `-quit`：关键参数，告诉 `find` 在处理完第一个匹配结果后立即退出

如果不加 `-quit`，它会为当前目录及子目录下的每一个文件都启动一个 Shell，导致终端被刷屏甚至卡死

![](https://pic1.imgdb.cn/item/6a3177d3b67c7e4f4a875c3c.png)