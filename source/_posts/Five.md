---
title: MazeSec Five
date: 2025-04-01 17:36:00
categories: 
  - MazeSec
tags:
  - 文件上传
  - Python 升级为交互式 Shell
  - /etc/sudoers 配置不当
  - cp 提权
  - man 提权
---

# Five

项目地址：https://downloads.hackmyvm.eu/five.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a222c5ff0d1d8911fab9cc4.png)

## 文件上传

### upload（路径绕过）

扫描目录

![](https://pic1.imgdb.cn/item/6a222df4f0d1d8911fabbae9.png)

访问 `upload.html` 可以上传文件

![](https://pic1.imgdb.cn/item/6a222e69f0d1d8911fabbb02.png)

上传反弹 Shell 成功，但是默认放在了 `uploads/` 目录下面

![](https://pic1.imgdb.cn/item/6a222fecf0d1d8911fabbb52.png)

这个路径访问是 403，抓包看看

![](https://pic1.imgdb.cn/item/6a223057f0d1d8911fabbb59.png)

将这个 `uploads/` 上传位置删除为空，这样子就传到了根目录下面，回显成功

![](https://pic1.imgdb.cn/item/6a22308cf0d1d8911fabbb5e.png)

访问拿到 Shell

![](https://pic1.imgdb.cn/item/6a2230c8f0d1d8911fabbb6a.png)

## 提权

### Python 升级为交互式 Shell

![](https://pic1.imgdb.cn/item/6a22310df0d1d8911fabbb7b.png)

### /etc/sudoers 配置不当

检查权限发现给了 `cp` 命令的 melisa 用户的权限

### cp 提权

由于前面没有扫描出开放了 22 端口，本地检查一下

有一个 4444 端口开放，暂时不知道是干嘛的

![](https://pic1.imgdb.cn/item/6a2231d4f0d1d8911fabbba4.png)

检查家目录看到用户 melisa 是有 `.ssh/` 目录的，说明可以走 SSH 登录

![](https://pic1.imgdb.cn/item/6a223238f0d1d8911fabbbe2.png)

拷贝过来登录 4444 端口发现还是需要密码

```bash
ssh melisa@127.0.0.1 -p 4444 -i id_rsa
```

![](https://pic1.imgdb.cn/item/6a22332af0d1d8911fabbc1f.png)

私钥有了，权限也对，应该是公钥出现了问题

但是因为 `id_rsa` 的权限是 melisa 用户的，我们没法直接查看

解决方法是我们自己创建一个文件，然后用把内容也复制过来

![](https://pic1.imgdb.cn/item/6a22341df0d1d8911fabbc70.png)

然后重新生成公钥

![](https://pic1.imgdb.cn/item/6a223450f0d1d8911fabbc84.png)

完整命令如下

```bash
www-data@five:/$ cd /tmp
www-data@five:/tmp$ touch id_rsa
www-data@five:/tmp$ sudo -u melisa cp /home/melisa/.ssh/id_rsa /tmp/id_rsa
www-data@five:/tmp$ chmod 600 /tmp/id_rsa
www-data@five:/tmp$ ssh-keygen -y -f id_rsa > authorized_keys
www-data@five:/tmp$ sudo -u melisa cp /tmp/authorized_keys /home/melisa/.ssh/authorized_keys
```

最后成功登录

![](https://pic1.imgdb.cn/item/6a223772f0d1d8911fabbcfd.png)

### man 提权

检查权限发现有很多命令可以使用

![](https://pic1.imgdb.cn/item/6a2237aef0d1d8911fabbd27.png)

使用 `man` 提权：

- `-P` 指定选项指定分页器为 less

```bash
sudo /bin/man -P less id
```

![](https://pic1.imgdb.cn/item/6a223a3af0d1d8911fabca27.png)

```
Ilovebinaries
WTFGivemefive
```