---
title: VulnHub digitalworld.local：electrical
date: 2026-06-09 18:30:00
categories: 
  - VulnHub
tags:
  - 敏感信息泄露
  - 弱口令
  - echo 逃逸
  - /etc/sudoers 配置不当
  - vim 提权
---

# digitalworld.local：electrical

博客地址：https://novasky.medium.com/digitalworld-local-development-walkthrough-vulnhub-story-of-obscurity-oscp-practice-5f52486f581b

项目地址：https://download.vulnhub.com/digitalworld/ELECTRICAL.7z（占用太大懒得下）

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

```bash
nmap -sC -sV -p- 192.168.217.134
Starting Nmap 7.94SVN ( https://nmap.org ) at 2023-12-17 12:09 +0545
Nmap scan report for 192.168.217.134
Host is up (0.00023s latency).
Not shown: 65530 closed tcp ports (conn-refused)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 79:07:2b:2c:2c:4e:14:0a:e7:b3:63:46:c6:b3:ad:16 (RSA)
|   256 c2:b6:8c:36:a6:dd:9b:17:bb:4f:0e:0f:16:89:d6:4b (ECDSA)
|_  256 24:6b:85:e3:ab:90:5c:ec:d5:83:49:54:cd:98:31:95 (ED25519)
113/tcp  open  ident?
|_auth-owners: oident
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
|_auth-owners: root
445/tcp  open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
|_auth-owners: root
8080/tcp open  http-proxy  IIS 6.0
|_http-open-proxy: Proxy might be redirecting requests
|_http-title: DEVELOPMENT PORTAL. NOT FOR OUTSIDERS OR HACKERS!
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Date: Sun, 17 Dec 2023 06:25:03 GMT
|     Server: IIS 6.0
|     Last-Modified: Wed, 26 Dec 2018 01:55:41 GMT
|     ETag: "230-57de32091ad69"
|     Accept-Ranges: bytes
|     Content-Length: 560
|     Vary: Accept-Encoding
|     Connection: close
|     Content-Type: text/html
|     <html>
|     <head><title>DEVELOPMENT PORTAL. NOT FOR OUTSIDERS OR HACKERS!</title>
|     </head>
|     <body>
|     <p>Welcome to the Development Page.</p>
|     <br/>
|     <p>There are many projects in this box. View some of these projects at html_pages.</p>
|     <br/>
|     <p>WARNING! We are experimenting a host-based intrusion detection system. Report all false positives to patrick@goodtech.com.sg.</p>
|     <br/>
|     <br/>
|     <br/>
|     <hr>
|     <i>Powered by IIS 6.0</i>
|     </body>
|     <!-- Searching for development secret page... where could it be? -->
|     <!-- Patrick, Head of Development-->
|     </html>                                                                                                                                                                                                                               
|   HTTPOptions:                                                                                                                                                                                                                            
|     HTTP/1.1 200 OK                                                                                                                                                                                                                       
|     Date: Sun, 17 Dec 2023 06:25:03 GMT                                                                                                                                                                                                   
|     Server: IIS 6.0                                                                                                                                                                                                                       
|     Allow: GET,POST,OPTIONS,HEAD                                                                                                                                                                                                          
|     Content-Length: 0                                                                                                                                                                                                                     
|     Connection: close                                                                                                                                                                                                                     
|     Content-Type: text/html                                                                                                                                                                                                               
|   RTSPRequest:                                                                                                                                                                                                                            
|     HTTP/1.1 400 Bad Request                                                                                                                                                                                                              
|     Date: Sun, 17 Dec 2023 06:25:03 GMT
|     Server: IIS 6.0
|     Content-Length: 294
|     Connection: close
|     Content-Type: text/html; charset=iso-8859-1
|     <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
|     <html><head>
|     <title>400 Bad Request</title>
|     </head><body>
|     <h1>Bad Request</h1>
|     <p>Your browser sent a request that this server could not understand.<br />
|     </p>
|     <hr>
|     <address>IIS 6.0 Server at 192.168.217.134 Port 8080</address>
|_    </body></html>
|_http-server-header: IIS 6.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.94SVN%I=7%D=12/17%Time=657E943F%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,330,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x2017\x20Dec\
SF:x202023\x2006:25:03\x20GMT\r\nServer:\x20IIS\x206\.0\r\nLast-Modified:\
SF:x20Wed,\x2026\x20Dec\x202018\x2001:55:41\x20GMT\r\nETag:\x20\"230-57de3
SF:2091ad69\"\r\nAccept-Ranges:\x20bytes\r\nContent-Length:\x20560\r\nVary
SF::\x20Accept-Encoding\r\nConnection:\x20close\r\nContent-Type:\x20text/h
SF:tml\r\n\r\n<html>\r\n<head><title>DEVELOPMENT\x20PORTAL\.\x20NOT\x20FOR
SF:\x20OUTSIDERS\x20OR\x20HACKERS!</title>\r\n</head>\r\n<body>\r\n<p>Welc
SF:ome\x20to\x20the\x20Development\x20Page\.</p>\r\n<br/>\r\n<p>There\x20a
SF:re\x20many\x20projects\x20in\x20this\x20box\.\x20View\x20some\x20of\x20
SF:these\x20projects\x20at\x20html_pages\.</p>\r\n<br/>\r\n<p>WARNING!\x20
SF:We\x20are\x20experimenting\x20a\x20host-based\x20intrusion\x20detection
SF:\x20system\.\x20Report\x20all\x20false\x20positives\x20to\x20patrick@go
SF:odtech\.com\.sg\.</p>\r\n<br/>\r\n<br/>\r\n<br/>\r\n<hr>\r\n<i>Powered\
SF:x20by\x20IIS\x206\.0</i>\r\n</body>\r\n\r\n<!--\x20Searching\x20for\x20
SF:development\x20secret\x20page\.\.\.\x20where\x20could\x20it\x20be\?\x20
SF:-->\r\n\r\n<!--\x20Patrick,\x20Head\x20of\x20Development-->\r\n\r\n</ht
SF:ml>\r\n")%r(HTTPOptions,A6,"HTTP/1\.1\x20200\x20OK\r\nDate:\x20Sun,\x20
SF:17\x20Dec\x202023\x2006:25:03\x20GMT\r\nServer:\x20IIS\x206\.0\r\nAllow
SF::\x20GET,POST,OPTIONS,HEAD\r\nContent-Length:\x200\r\nConnection:\x20cl
SF:ose\r\nContent-Type:\x20text/html\r\n\r\n")%r(RTSPRequest,1CD,"HTTP/1\.
SF:1\x20400\x20Bad\x20Request\r\nDate:\x20Sun,\x2017\x20Dec\x202023\x2006:
SF:25:03\x20GMT\r\nServer:\x20IIS\x206\.0\r\nContent-Length:\x20294\r\nCon
SF:nection:\x20close\r\nContent-Type:\x20text/html;\x20charset=iso-8859-1\
SF:r\n\r\n<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//IETF//DTD\x20HTML\x202\.0//E
SF:N\">\n<html><head>\n<title>400\x20Bad\x20Request</title>\n</head><body>
SF:\n<h1>Bad\x20Request</h1>\n<p>Your\x20browser\x20sent\x20a\x20request\x
SF:20that\x20this\x20server\x20could\x20not\x20understand\.<br\x20/>\n</p>
SF:\n<hr>\n<address>IIS\x206\.0\x20Server\x20at\x20192\.168\.217\.134\x20P
SF:ort\x208080</address>\n</body></html>\n");
Service Info: Host: DEVELOPMENT; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## 敏感信息泄露

### SiTeFiLo 1.0.6 - 文件泄露（CVE-2008-5762）

访问 8080 端口

![](https://pic1.imgdb.cn/item/6a2808fbedae85a628504e40.png)

如果你尝试用 gobuster 测试它，它会将你锁定 10 分钟

我们在 `/development.html` 和 `/development` 目录下找到了一些有用的信息

![](https://pic1.imgdb.cn/item/6a28098dedae85a628504e70.png)

这段文字不知所云，似乎隐藏着某种秘密，我们下载 test.pcap 文件

在其中找到一段访问路径

![](https://pic1.imgdb.cn/item/6a2809bbedae85a628504e7f.png)

访问页面

![](https://pic1.imgdb.cn/item/6a2809e8edae85a628504e8a.png)

点击注销按钮后，我们会重定向到登录页面

![](https://pic1.imgdb.cn/item/6a280a05edae85a628504e8e.png)

尝试使用一些随机字符登录，将会收到错误提示

```
Deprecated: Function ereg_replace() is deprecated in /var/www/html/developmentsecretpage/slogin_lib.inc.php on line 335
```

Google 得到该框架的 CVE 漏洞，拿到用户名以及密码哈希

![](https://pic1.imgdb.cn/item/6a283f5dedae85a62851890e.png)

## 弱口令

### SSH

爆破密码得到 `12345678900987654321`

![](https://pic1.imgdb.cn/item/6a28417dedae85a62851899f.png)

尝试使用 SSH 内部连接登录后，虽然成功了，我们获得了一个 shell，但权限有限（被限制），如果我们胡乱操作就会被踢出去

![](https://pic1.imgdb.cn/item/6a28417dedae85a62851899f.png)

## 提权

### echo 逃逸

所以我们需要逃离这个牢笼，了解它的工作原理将有助于我们逃离它

尝试传递几个字符后，它在处理数字时返回错误，错误信息显示它是用 Python 构建的，再次产生错误信息对我们有所帮助

![](https://pic1.imgdb.cn/item/6a2841f9edae85a6285189d4.png)

在 bash 中，`echo` 命令会返回脚本的执行结果

我们可以通过 Python OS 库生成一个 shell

```bash
echo os.system('/bin/bash')
```

![](https://pic1.imgdb.cn/item/6a284348edae85a628518a68.png)

检查结果后我们发现，使用 intern 用户无法使用 sudo，但使用 patrick 用户可以

所以切换为 patrick 用户

### /etc/sudoers 配置不当

检查权限该用户可以使用 `vim` 命令

### vim 提权

直接提权到 root

```bash
sudo vim 
:shell
```