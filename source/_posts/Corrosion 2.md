---
title: VulnHub Corrosion：2
date: 2026-06-09 18:31:00
categories: 
  - VulnHub
tags:
  - 敏感信息泄露
  - 弱口令
  - RCE
  - Python 升级为交互式 Shell
  - look 提权
  - 环境变量劫持
---

# Corrosion：2

博客地址：https://medium.com/@ozkirdenizenes/vulnhub-corrosion-2-walkthrough-c3e2f6057574

项目地址：https://download.vulnhub.com/digitalworld/ELECTRICAL.7z（占用太大懒得下）

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a28c54bedae85a628529e3e.png)

## 敏感信息泄露

### backup.zip

访问 8080 端口是 Tomcat

![](https://pic1.imgdb.cn/item/6a28cc58edae85a62852a0a7.png)

扫描目录发现 `/backup.zip`

![](https://pic1.imgdb.cn/item/6a28ccebedae85a62852a0d9.png)

解压需要密码

![](https://pic1.imgdb.cn/item/6a28cd6cedae85a62852a102.png)

提取出哈希值

![](https://pic1.imgdb.cn/item/6a28ce16edae85a62852d903.png)

然后去破解

![](https://pic1.imgdb.cn/item/6a28ce26edae85a62852d907.png)

解压文件后找到用户名密码

```bash
admin
melehifokivai
```

![](https://pic1.imgdb.cn/item/6a28cebeedae85a62852d941.png)

## 弱口令

### Tomcat

成功登录进去

![](https://pic1.imgdb.cn/item/6a28cf85edae85a62852d9a7.png)

## RCE

### War 包部署

生成 War 包部署上去拿 Shell

![](https://pic1.imgdb.cn/item/6a28cfe3edae85a62852d9b9.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a28d06aedae85a62852d9c1.png)

### 敏感信息泄露

查看家目录有两个用户

![](https://pic1.imgdb.cn/item/6a28d0d6edae85a62852d9c8.png)

尝试之前的密码切换成功

![](https://pic1.imgdb.cn/item/6a28d0f2edae85a62852d9cd.png)

### SUID 提权

查找具有 SUID 权限的文件，发现 `/home/jaye/Files/look`

![](https://pic1.imgdb.cn/item/6a28d111edae85a62852d9e5.png)

### look 提权

| 组成部分 | 含义 |
|---|---|
| `look` | 系统自带的二进制程序（通常在 `/usr/bin/look`） |
| `''` | 第一个参数：空字符串 |
| `/path/to/input-file` | 第二个参数：要读取的目标文件 |

它的正常语法是：`look <你要查的词头> <字典文件>`

**在 Linux 中，任何字符串，都被认为是 "以空字符串开头的"**

由于每一行都以空字符串 "开头"，因此 `look ''` 会匹配文件中的所有行，等同于读取并输出整个文件内容，效果类似于 `cat`

我们将 `LFILE` 变量设置为目标文件（`LFILE=/etc/shadow`），然后执行漏洞利用命令：`./look '' $LFILE`

![](https://pic1.imgdb.cn/item/6a28d2c1edae85a62852da57.png)

成功破解出 `randy` 用户的密码

![](https://pic1.imgdb.cn/item/6a28d2daedae85a62852da5a.png)

### 环境变量劫持

SSH 登录后检查权限发现可以执行 `randombase64.py`

![](https://pic1.imgdb.cn/item/6a28d30dedae85a628530041.png)

我们使用 `python` 命令检查脚本的源代码 `cat /home/randy/randombase64.py`

我们发现脚本非常简单，但其中包含一行关键代码：`python -f 'lib' import base64`

我们找到它的位置并运行命令 `ls -alh /usr/lib/python3.8/base64.py` 来检查其权限为 777

![](https://pic1.imgdb.cn/item/6a28d37cedae85a628530070.png)

我们添加一行代码 `os.system("/bin/bash")` 劫持，成功提到 root 用户

![](https://pic1.imgdb.cn/item/6a28d3b3edae85a62853008b.png)