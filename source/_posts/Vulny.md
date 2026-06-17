---
title: MazeSec Vulny
date: 2025-04-01 17:38:00
categories: 
  - MazeSec
tags:
  - RCE
  - Python 升级为交互式 Shell
  - 敏感信息泄露
  - /etc/sudoers 配置不当
  - flock 提权
---

# Vulny

项目地址：https://downloads.hackmyvm.eu/vulny.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a261ef167f23bae9b003dfc.png)

## RCE

### WP-file-manager v6.9（CVE-2020-25213）

只开放了 80 端口，扫下后台

![](https://pic1.imgdb.cn/item/6a26258467f23bae9b006579.png)

有个 `/secret` 目录，继续扫描发现有安装 Wordpress

![](https://pic1.imgdb.cn/item/6a2625e967f23bae9b006587.png)

安装了一个插件是 `wp-file-manager`

![](https://pic1.imgdb.cn/item/6a26262867f23bae9b006598.png)

搜索插件找到一个 RCE 漏洞

![](https://pic1.imgdb.cn/item/6a26266f67f23bae9b0065a3.png)

运行脚本

![](https://pic1.imgdb.cn/item/6a26270967f23bae9b0065cd.png)

查看源码可以看到其上传的脚本密码以及名称

![](https://pic1.imgdb.cn/item/6a2627d767f23bae9b006634.png)

网上搜索漏洞相关信息找到路径 `/wp-content/plugins/wp-file-manager/lib/files/shell.php?cmd=`

开启 HTTP 服务让其下载 PHP 反弹 Shell，然后运行

![](https://pic1.imgdb.cn/item/6a26293367f23bae9b006757.png)

拿到 Shell

![](https://pic1.imgdb.cn/item/6a26296067f23bae9b00677c.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a2629be67f23bae9b0067bf.png)

### 敏感信息泄露

得到一组假密码，但是给出了一个路径

![](https://pic1.imgdb.cn/item/6a262a3867f23bae9b006837.png)

进入路径中，这是 Wordpress 源码所在位置

![](https://pic1.imgdb.cn/item/6a262a6e67f23bae9b006869.png)

查看数据库配置文件，找到一个注释

![](https://pic1.imgdb.cn/item/6a262ae667f23bae9b0068d5.png)

查看家目录，推测这是该用户的密码

![](https://pic1.imgdb.cn/item/6a262b0067f23bae9b0068ea.png)

登录成功

![](https://pic1.imgdb.cn/item/6a262b2967f23bae9b006912.png)

### /etc/sudoers 配置不当

查看自身权限

![](https://pic1.imgdb.cn/item/6a262b7867f23bae9b00696a.png)

### flock 提权

flock 是 Linux 自带的文件锁工具

- `-u`：释放文件锁

- `/`：锁操作的目标文件（这里选根目录，因为它永远存在）

- `/bin/sh`：flock 要执行的子命令

![](https://pic1.imgdb.cn/item/6a262bd167f23bae9b0069e9.png)

```
HMViuploadfiles
HMVididit
```