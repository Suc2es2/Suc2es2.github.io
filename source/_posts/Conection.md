---
title: MazeSec Conection
date: 2025-04-01 16:29:00
categories: 
  - MazeSec
tags:
  - RCE
  - SUID 提权
  - GDB
---

# Conection

项目地址：https://downloads.hackmyvm.eu/connection.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1d3d74dd21e9856bd4b4b6.png)

## RCE

### SMB

139 445 端口全开，使用 enum4linux 枚举信息

![](https://pic1.imgdb.cn/item/6a1d3ed2dd21e9856bd4dc33.png)

发现一个共享文件夹 `share`

![](https://pic1.imgdb.cn/item/6a1d3f5ddd21e9856bd4dcc2.png)

使用 smbclient 加上 `-N` 参数匿名访问发现是分享的 Web 目录

![](https://pic1.imgdb.cn/item/6a1d47ecdd21e9856bd4e27a.png)

上传一个反弹 Shell PHP 上去

![](https://pic1.imgdb.cn/item/6a1d48f2dd21e9856bd4e335.png)

## 提权

### /etc/sudoers 配置不当

寻找提权点

```bash
find / -user root -perm /4000 2>/dev/null
```

发现 `/usr/bin/gdb` 可以使用

### GDB 提权

提权命令解析：

- `nx`：不加载任何 `.gdbinit` 初始化文件，防止被干扰

- `ex`：启动时立即执行后面的命令

- `import os; os.setuid(0)`：调用 Linux 底层的 `setuid()` 系统调用，将当前进程的真实用户 ID 强制修改为 0

- `!bash`：启动一个新的 bash 终端

- `-ex quit`：启动 bash 后，立即退出 gdb 调试器本身

```bash
gdb -nx -ex 'python import os; os.setuid(0)' -ex '!bash' -ex quit
```

最后的 flag

```
a7c6ea4931ab86fb54c5400204474a39
3f491443a2a6aa82bc86a3cda8c39617
```