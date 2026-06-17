---
title: VulnHub Empire：Breakout
date: 2026-05-04 22:20:00
categories: 
  - VulnHub
tags:
  - 敏感信息泄露
  - Brainfuck 解密
  - CTF
  - SMB 用户枚举
  - Capabilities 提权
---

# Empire：Breakout

项目地址：https://download.vulnhub.com/empire/02-Breakout.zip

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69f8b00cb681ce9bfd1fb9e0.png)

80、10000、20000 都是 Web 服务

![](https://pic1.imgdb.cn/item/69f8b031b681ce9bfd1fb9e1.png)

## 敏感信息泄露

### Apache2 Debian Default Page: It works

在 80 端口首页中发现注释

![](https://pic1.imgdb.cn/item/69f8b05fb681ce9bfd1fb9e8.png)

### SMB 用户枚举

前面扫描发现一个开放的 SMB 端口，使用 enum4linux 进行用户枚举

找到用户 cyber

![](https://pic1.imgdb.cn/item/69f8b520b681ce9bfd1ff917.png)

## CTF

### Brainfuck 解密

前面的注释一眼 Brainfuck，去在线网站解密

![](https://pic1.imgdb.cn/item/69f8b0e2b681ce9bfd1fb9fe.png)

拿到 `.2uqPEfj3D<P'a-3` 去登录

发现 20000 端口可以登录进去

![](https://pic1.imgdb.cn/item/69f8b56eb681ce9bfd1ff921.png)

## RCE

### Usermin 1.830

发现在最下面有一个终端，点击后可以执行任何命令

![](https://pic1.imgdb.cn/item/69f8b647b681ce9bfd1ff936.png)

直接反弹 Shell

![](https://pic1.imgdb.cn/item/69f8b6f2b681ce9bfd1ff949.png)

## 提权

### Capabilities 提权

`sudo -l` 查看权限无果

![](https://pic1.imgdb.cn/item/69f8b71eb681ce9bfd1ff950.png)

执行命令 `getcap -r / 2>/dev/null`

- `getcap`：用于查看文件的 "能力"（capabilities）

- `-r`：递归查询

- `/`：指定搜索的根目录

- `2>/dev/null`：将所有报错信息直接丢弃

当一个文件被设置了能力，它的行为会受到特殊影响。例如：

- `cap_setuid`：允许程序修改自己的UID，即把自己变成 root

- `cap_net_bind_service`：允许程序绑定小于1024的特权端口

- `cap_sys_ptrace`：允许程序读写其他进程的内存，可用于注入代码

- `cap_dac_read_search`：允许程序绕过文件系统的所有读权限检查，直接读取任何文件

`/usr/bin/ping` 带有 `cap_net_raw` 是正常系统配置，用于发送 ICMP 包

`/home/cyber/tar` 带有 `cap_dac_read_search=ep` 则是极度高危信号——这个自定义的 `tar` 程序可以绕过所有文件读权限检查

![](https://pic1.imgdb.cn/item/69f8b94bb681ce9bfd1ff986.png)


在备份目录中找到一个密码的备份文件

![](https://pic1.imgdb.cn/item/69f8be25b681ce9bfd1ff9fb.png)

在家目录中可以以 Root 身份运行 `tar`

![](https://pic1.imgdb.cn/item/69f8be95b681ce9bfd1ff9fc.png)

先将 `/var/backups/.old_pass.bak` 打包，`/home/cyber/tar` 拥有 `cap_dac_read_search`，可以直接忽略权限检查，将它打包进归档

然后解压上一步生成的 `passwd`

即在当前目录下创建了 `var/backups/` 目录，并把旧的密码备份文件还原了出来

![](https://pic1.imgdb.cn/item/69f8c051b681ce9bfd1ffa1f.png)

拿到密码

```
Ts&4&YurgtRX(=~h
```

![](https://pic1.imgdb.cn/item/69f8c096b681ce9bfd1ffa20.png)

最后提权成功

![](https://pic1.imgdb.cn/item/69f8c146b681ce9bfd1ffa23.png)