---
title: MazeSec suidy
date: 2025-04-01 17:29:00
categories: 
  - MazeSec
tags:
  - 弱口令
  - SUID 提权
  - 定时任务
---

# suidy

项目地址：https://downloads.hackmyvm.eu/connection.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a1e390db3f09b19e9d3a6fc.png)

## 弱口令

### SSH

扫描目录在 `robots.txt` 中发现 `/shehatesme/` 路径，于是去访问

![](https://pic1.imgdb.cn/item/6a1e3a3eb3f09b19e9d3a71a.png)

给出了用户密码 `theuser/thepass`，连接成功

![](https://pic1.imgdb.cn/item/6a1e3aaeb3f09b19e9d3a72b.png)

## 提权

### SUID 提权

发现一个家目录下面的文件有 Suid

![](https://pic1.imgdb.cn/item/6a1e3b9ab3f09b19e9d3a751.png)

直接执行拿到 suidy 用户权限

![](https://pic1.imgdb.cn/item/6a1e3d0eb3f09b19e9d3a777.png)

### 定时任务

上传 pspy64 运行，发现了 Root 定时任务

每分钟执行 `/root/timer.sh`

![](https://pic1.imgdb.cn/item/6a1e4035b3f09b19e9d3a7dc.png)

查看提示推测 root 用户会定期检查二进制文件 `suidyyyy` 的 SUID 权限

![](https://pic1.imgdb.cn/item/6a1e4158b3f09b19e9d3a87c.png)

但是因为该文件我们可以编辑，所以直接修改源码（需要切换到 theuser 用户）

```c
#include <stdlib.h>
#include <unistd.h>

int main() {
    setuid(0);
    setgid(0);
    system("/bin/bash");
    return 0;
}
```

创建一个 `.c` 文件编译下，然后替换掉原来的程序

![](https://pic1.imgdb.cn/item/6a1e4399b3f09b19e9d408dc.png)

等待一段时间后提权成功

![](https://pic1.imgdb.cn/item/6a1e43ceb3f09b19e9d40909.png)

```
HMV2353IVI
HMV0000EVE
```