---
title: VulnHub EvilBox：One
date: 2026-06-15 18:44:00
categories: 
  - VulnHub
tags:
  - 
---

# EvilBox：One

项目地址：https://download.vulnhub.com/evilbox/EvilBox---One.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a31183ab67c7e4f4a852e32.png)

## 文件包含

### evil.php

扫描目录发现 `/secret`

![](https://pic1.imgdb.cn/item/6a3121ccb67c7e4f4a857304.png)

继续扫描出 `/secret/evil.php` 文件，FUZZ 出参数是 `command`，测试存在文件包含漏洞

![](https://pic1.imgdb.cn/item/6a3122c8b67c7e4f4a8573bc.png)

利用 PHP 伪协议读取源码

![](https://pic1.imgdb.cn/item/6a31235bb67c7e4f4a85742c.png)

解码得到

```php
<?php
    $filename = $_GET['command'];
    include($filename);
?>
```

尝试读取用户私钥

```php
php://filter/read=convert.base64-encode/resource=/home/mowree/.ssh/id_rsa
```

![](https://pic1.imgdb.cn/item/6a3123b4b67c7e4f4a85747f.png)

解码保存私钥后连接发现需要密码

![](https://pic1.imgdb.cn/item/6a312421b67c7e4f4a857502.png)

利用 `ssh2john` 生成 hash 然后使用 `john` 爆破出来是 `unicorn`

![](https://pic1.imgdb.cn/item/6a31245cb67c7e4f4a857551.png)

## 提取

### 权限配置不当

上传 `linpeas.sh` 发现 `/etc/passwd` 可写

![](https://pic1.imgdb.cn/item/6a31259ab67c7e4f4a857747.png)

按照 root 用户的格式新增一个用户写入到 `/etc/passwd` 文件中，权限改为 `0` 即可