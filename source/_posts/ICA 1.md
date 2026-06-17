---
title: VulnHub ICA：1
date: 2026-06-08 17:18:00
categories: 
  - VulnHub
tags:
  - 
---

# ICA：1

项目地址：https://download.vulnhub.com/ica/ica1.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a27ad22edae85a6284e6f34.png)

## 敏感信息泄露

### qdPM 9.2 - Password Exposure

访问网页，运行的是 qdPM 9.2

![](https://pic1.imgdb.cn/item/6a27b6eaedae85a6284e92c2.png)

找到一个密码泄露的 NDay

![](https://pic1.imgdb.cn/item/6a27b76aedae85a6284e930b.png)

查看文件内容是让我们访问一个路径

![](https://pic1.imgdb.cn/item/6a27b7f2edae85a6284e9344.png)

访问过去拿到数据库账号密码

![](https://pic1.imgdb.cn/item/6a27b82dedae85a6284e9357.png)

## 弱口令

### MySQL

远程连接数据库

![](https://pic1.imgdb.cn/item/6a27b957edae85a6284e93b7.png)

### SSH

在数据库中翻出用户名密码

![](https://pic1.imgdb.cn/item/6a27ba30edae85a6284ec6c9.png)

最终确认这组用户名密码可以登录 SSH

![](https://pic1.imgdb.cn/item/6a27babaedae85a6284ec70c.png)

## 提权

### SUID 提权

找到一个 SUID 文件

![](https://pic1.imgdb.cn/item/6a27bb39edae85a6284ed7f7.png)

直接运行好像没啥有用的东西输出

![](https://pic1.imgdb.cn/item/6a27bbebedae85a6284ed841.png)

### 环境变量劫持

查看可识别的字符串，执行了命令 `cat /root/system.info`

![](https://pic1.imgdb.cn/item/6a27bc34edae85a6284ed863.png)

我们进入 `/tmp` 创建一个同名的 `cat` 文件，然后打环境变量劫持

```bash
export PATH=/tmp:$PATH
```

![](https://pic1.imgdb.cn/item/6a27bdb7edae85a6284ed8da.png)