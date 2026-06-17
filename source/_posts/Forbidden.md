---
title: MazeSec Forbidden
date: 2025-04-01 17:36:00
categories: 
  - MazeSec
tags:
  - 弱口令
  - RCE
  - Python 升级为交互式 Shell
  - 敏感文件泄露
  - john 任意文件读取
  - setarch
---

# Forbidden

项目地址：https://downloads.hackmyvm.eu/forbidden.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a23d4010c24090b1026999c.png)

## 弱口令

### FTP

上面扫描出可以匿名登录 FTP

![](https://pic1.imgdb.cn/item/6a23d4450c24090b102699c8.png)

下载 `note.txt` 文件提示有一个 `.jpg` 包含密码

![](https://pic1.imgdb.cn/item/6a23d4450c24090b102699c8.png)

继续下载首页，得到提示

```
无法执行 .php 代码
```

![](https://pic1.imgdb.cn/item/6a23d4fe0c24090b10269a0c.png)

## RCE

### FTP

正常的 `.php` 文件无法执行代码，那就换一些后缀例如 `.phtml`、`.php5` 等等传上去

![](https://pic1.imgdb.cn/item/6a23d6380c24090b10269a9d.png)

`.php5` 可以执行，成功连接上 Shell

![](https://pic1.imgdb.cn/item/6a23d6730c24090b10269ab4.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a23d6bd0c24090b10269af0.png)

根据前面的提示去找一个 `.jpg` 文件，然后通过 Python 开启 HTTP 传过来

![](https://pic1.imgdb.cn/item/6a23d71f0c24090b10269b0c.png)

访问网页

![](https://pic1.imgdb.cn/item/6a23d78f0c24090b10269b2c.png)

### 敏感文件泄露

弄了半天原来密码就是文件名

![](https://pic1.imgdb.cn/item/6a23dbdc0c24090b10269cc6.png)

### /etc/sudoers 配置不当

检查自身权限，给了 `john`

![](https://pic1.imgdb.cn/item/6a23dc4b0c24090b10269cda.png)

### john 任意文件读取

```bash
sudo join -a 2 /dev/null /etc/shadow
```

命令解析如下：

- `join` 尝试将 File 1 (`/dev/null`) 的第一列与 File 2 (`/etc/shadow`) 的第一列进行匹配

- 因为 `/dev/null` 里面没有任何内容，所以 `/etc/shadow` 中的任何一行都不可能在 File 1 中找到匹配项

- `-a` 参数用于指定除了输出匹配成功的行之外，还要输出未匹配成功的行

- `-a 2` 代表同时输出第二个文件中无法匹配的行

- 因为全盘匹配失败，`/etc/shadow` 里的所有行都属于 "未匹配成功的行"。此时 `-a 2` 参数生效，强制要求将 File 2 中所有未匹配的行全部打印到终端

![](https://pic1.imgdb.cn/item/6a23dca70c24090b10269cf4.png)

最后拿到每个用户的 hash

```bash
markos:$6$PTerrFpyfOmkM5Xi$oo8gNZyyxsZbKhOIXrm2w/x.Xvhdr7Ny/4JgLDRLRAxAwEwGtH2kD7PjzeloAstqCPq/KKrqrPioMM8vwWbqZ.:18544:0:99999:7:::
peter:$6$QAeWH9Et9PAJdYz/$/4VhburW9KoVTRY1Ry63wNEfr4rxwQGaRJ3kKW2nEAk0LcqjqZjy/m5rtaCi3VebNu7AaGFhQT4FBgbQVIyq81:18544:0:99999:7:::
marta:$6$h.4ZF5esZ/N1OIcu$8vL1D3iM6iuhniSG8nIz0582atbIV6y/UBl0eks1.Wrd51BqLK8Wqt91WXg0Y2mrdNY4luPQkqUWXFXWxLVwe/:18544:0:99999:7:::
root:$6$8nU2FdqnxRtT9mWF$9q7El.D7BDrlzNyYYPNqjTcwsQEsC7utrzszLgbe9V.3KqYSfx2XgqjIEeToP41TJTiZQOGVsdCzIAYHw5O.51:18544:0:99999:7:::
```

最后 `john` 爆破得到

```bash
peter:boomer
```

切换用户成功，检查自身权限

![](https://pic1.imgdb.cn/item/6a23de3e0c24090b10269d37.png)

### setarch 提权

`setarch` 是 Linux 系统自带的一个系统管理工具

它的核心功能是修改内核的进程人格，并在这个修改后的环境中运行一个新程序

在 `setarch` 的设计中，`-3`（等同于 `--3gb`）是一个完全合法的选项

- 底层原理：它通过调用 Linux 的 `personality()` 系统调用，向内核申请设置 `ADDR_LIMIT_3GB` 标志位

![](https://pic1.imgdb.cn/item/6a23def10c24090b10269d69.png)

```
HMVpussycat
HMVmymymymymind
```