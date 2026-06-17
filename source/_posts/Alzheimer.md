---
title: MazeSec Alzheimer
date: 2025-04-01 17:33:00
categories: 
  - MazeSec
tags:
  - Port Knocking
  - 弱口令
  - SUID 提权
  - casph 提权
---

# Alzheimer

项目地址：https://downloads.hackmyvm.eu/alzheimer.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1fe7509ecef7401784e937.png)

## 弱口令

### FTP

前面 NMap 已经扫描出来了，直接匿名登录

```
229 Entering Extended Passive Mode (|||42234|)
```

这说明 FTP 正在尝试使用被动模式建立数据通道来传输文件列表，端口为 42234 端口

需要先输入 `passive` 关闭被动模式

![](https://pic1.imgdb.cn/item/6a1fea6b9ecef74017853005.png)

## CTF

### Port Knocking

查看隐藏文件内容

```
I need to knock this ports and
one door will be open!
1000
2000
3000
```

我真没招了，刚开始只开 80。重启了都不开了

![](https://pic1.imgdb.cn/item/6a1ffa7e9ecef740178563de.png)

## 弱口令

### SSH

首页回显

```
I dont remember where I stored my password :(
I only remember that was into a .txt file...
-medusa

<!---. --- - .... .. -. --. -->
```

Web 端扫描目录，`/secret/` 说有一个隐藏文件

```
Maybe my password is in this secret folder?
```

猜测是 FTP 端口的 `.secretnote.txt`，将之前的文件删除，再重新 Get 一下

```
I need to knock this ports and
one door will be open!
1000
2000
3000
Ihavebeenalwayshere!!!
Ihavebeenalwayshere!!!
```

多出来的是首页中的用户的密码，SSH 直接登录

## 提权

### SUID 提权 + casph 提权

```bash
medusa@alzheimer:~$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/bin/chsh
/usr/bin/sudo
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/umount
/usr/bin/gpasswd
/usr/sbin/capsh
```

- `/usr/sbin/capsh`：用于操作和测试 Linux Capabilities
- `--gid=0`：将进程的有效 GID 设置为 `0`
- `--uid=0`：将进程的有效 UID 设置为 0
- `--`：参数分隔符，之后执行默认 shell（通常是 /bin/bash）

```
HMVrespectmemories
HMVlovememories
```