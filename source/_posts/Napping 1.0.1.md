---
title: VulnHub Napping：1.0.1
date: 2026-06-08 17:14:00
categories: 
  - VulnHub
tags:
  - 重定向漏洞
  - 定时任务
  - /etc/sudoers 配置不当
  - vim 提权
---

# Napping：1.0.1

博客原地址：https://readysetexploit.gitlab.io/home/vulnhub/napping/（靶机没部署好）

项目地址：https://download.vulnhub.com/napping/napping-1.0.1.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a26efdd3c9809430d37c381.png)

## 重定向漏洞

### Blog Link

访问网页要求登录

![](https://pic1.imgdb.cn/item/6a26f0ed3c9809430d37c3d6.png)

接下来，我们创建一个账户，用户名和密码随意

登录后，我们会看到一个免费的博客推广网站

![](https://pic1.imgdb.cn/item/6a26f1023c9809430d37c3e0.png)

看起来我们可以提交链接，提交后，链接就会显示在网站上供我们查看

![](https://pic1.imgdb.cn/item/6a26f1243c9809430d37c3ea.png)

如果我们查看源代码，就会发现网站上的这个特定 URL 链接功能存在重定向漏洞

![](https://pic1.imgdb.cn/item/6a26f23a3c9809430d37c47c.png)

首先，我们来复制登录页面

![](https://pic1.imgdb.cn/item/6a26f2c23c9809430d37c4a3.png)

接下来，我们构建恶意 HTML 有效载荷

![](https://pic1.imgdb.cn/item/6a26f3253c9809430d37c4ca.png)

使用 Python 开启 HTTP 服务

![](https://pic1.imgdb.cn/item/6a26f3973c9809430d37c4e4.png)

成功拿到凭证

![](https://pic1.imgdb.cn/item/6a26f4063c9809430d37c505.png)

```
daniel
C@ughtm3napping123
```

## 弱口令

### SSH

![](https://pic1.imgdb.cn/item/6a26f48d3c9809430d37c526.png)

## 提权

### 定时任务

看到用户是另一个组的成员

![](https://pic1.imgdb.cn/item/6a26f4723c9809430d37c51b.png)

查看有哪些文件可以用

```bash
find / -group administrators -type f 2>/dev/null
```

![](https://pic1.imgdb.cn/item/6a26f4f63c9809430d37d1b4.png)

查看源码，它似乎在检查 Web 服务器的状态，然后将其写入文件 `site_status.txt`

![](https://pic1.imgdb.cn/item/6a26f5013c9809430d37d1b8.png)

作为组成员，我们拥有该文件的写入权限，并且根据 `site_status.txt` 文件显示，该文件似乎每 2 分钟执行一次

![](https://pic1.imgdb.cn/item/6a26f5393c9809430d37d1c8.png)

创建反弹 Shell

![](https://pic1.imgdb.cn/item/6a26f5423c9809430d37d1cb.png)

修改定时脚本的内容

![](https://pic1.imgdb.cn/item/6a26f55e3c9809430d37d1d3.png)

等待一段时间拿到 Shell

![](https://pic1.imgdb.cn/item/6a26f57b3c9809430d37d1db.png)

### /etc/sudoers 配置不当

检查权限发现可以使用 `vim`

![](https://pic1.imgdb.cn/item/6a26f5b03c9809430d37d1e9.png)

### vim 提权

随便写一个文件然后新建一个 Shell 提权

![](https://pic1.imgdb.cn/item/6a26f5d13c9809430d37d1f2.png)