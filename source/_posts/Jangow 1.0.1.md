---
title: VulnHub Jangow：1.0.1
date: 2026-06-08 16:16:00
categories: 
  - VulnHub
tags:
  - RCE
  - 弱口令
  - eBPF 提权（CVE-2017-16995）
---

# Jangow: 1.0.1

项目地址：https://download.vulnhub.com/jangow/jangow-01-1.0.1.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a2686b23c9809430d35b475.png)

## RCE

### buscar

扫描出 `/site` 目录，访问过去

![](https://pic1.imgdb.cn/item/6a2687033c9809430d35b4bd.png)

点击右上角的 Buscar 会跳转到一个空白页面

并且也会多一个参数，但是值是空的

![](https://pic1.imgdb.cn/item/6a2687373c9809430d35b4e1.png)

输入 `id` 竟然会执行，存在 RCE 漏洞

![](https://pic1.imgdb.cn/item/6a26876f3c9809430d35b507.png)

但是无论换成哪种 Shell 都弹不回来，查看当前目录所有文件发现 Wordpress

![](https://pic1.imgdb.cn/item/6a2687d43c9809430d35b522.png)

在其目录下有一个 `config.php` 文件，查看拿到数据库的配置信息

![](https://pic1.imgdb.cn/item/6a2688123c9809430d35b530.png)

但是查看家目录没有这个用户

![](https://pic1.imgdb.cn/item/6a2688673c9809430d35b554.png)

在 `/html` 目录下发现 `.backup`

![](https://pic1.imgdb.cn/item/6a26887d3c9809430d35b55e.png)

查看拿到账号密码

![](https://pic1.imgdb.cn/item/6a2688e73c9809430d35b580.png)

## 弱口令

### FTP

登录 FTP，但是不能上传文件

![](https://pic1.imgdb.cn/item/6a2689e73c9809430d35b610.png)

## 提权

### eBPF 提权（CVE-2017-16995）

没想到居然要手动登录靶机回弹 Shell……

![](https://pic1.imgdb.cn/item/6a268afa3c9809430d35b646.png)

后续是利用 FTP 从家目录上传（我这里传 PHP 只是为了测试）

![](https://pic1.imgdb.cn/item/6a268b573c9809430d35b64e.png)

传漏洞利用脚本提权成功

![](https://pic1.imgdb.cn/item/6a268b8f3c9809430d35b66b.png)