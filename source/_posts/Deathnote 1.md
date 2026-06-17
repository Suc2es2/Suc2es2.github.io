---
title: VulnHub Deathnote：1
date: 2026-06-10 18:40:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - 敏感信息泄露
  - /etc/sudoers 配置不当
---

# Deathnote：1

项目地址：https://download.vulnhub.com/deathnote/Deathnote.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a298829be8f317e8bad01c9.png)

## 弱口令

### SSH

访问 80 端口会自动跳转到一个域名

![](https://pic1.imgdb.cn/item/6a2a1c8a336be91663799056.png)

把域名添加到 HOSTS 中

![](https://pic1.imgdb.cn/item/6a2a1cd9336be9166379906d.png)

随后可以正常访问

![](https://pic1.imgdb.cn/item/6a2a1d3d336be91663799078.png)

扫描后台找到一个用户名字典

![](https://pic1.imgdb.cn/item/6a2a1e30336be916637990aa.png)

`notes.txt` 中则是密码字典

![](https://pic1.imgdb.cn/item/6a2a1e6f336be916637990be.png)

爆破出用户名密码

```
l:death4me
```

登录成功

![](https://pic1.imgdb.cn/item/6a2a1ec4336be916637990c5.png)

## 提权

### 敏感信息泄露

在 `/opt` 目录中拿到 `kira` 用户的提示

![](https://pic1.imgdb.cn/item/6a2a1f5e336be916637990e3.png)

解密得到账号密码

![](https://pic1.imgdb.cn/item/6a2a1fb7336be916637990f9.png)

切换用户

![](https://pic1.imgdb.cn/item/6a2a22c7336be9166379919b.png)

### /etc/sudoers 配置不当

![](https://pic1.imgdb.cn/item/6a2a2348336be916637991a9.png)