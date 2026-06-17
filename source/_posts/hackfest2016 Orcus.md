---
title: VulnHub hackfest2016：Orcus
date: 2026-04-01 16:38:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - 文件上传
  - Python 升级为交互式 Shell
  - NFS 配置不当
---

# hackfest2016: Orcus

项目地址：https://download.vulnhub.com/hackfest2016/Orcus.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69c7acb4f2426dbb8efa297d.png)

## 弱口令

### phpMyAdmin

扫描目录

![](https://pic1.imgdb.cn/item/69c7ac97f2426dbb8efa2978.png)

访问 `/backups` 目录拿到源码

![](https://pic1.imgdb.cn/item/69c7adc5f2426dbb8efa29bd.png)

翻到了数据库账号密码

![](https://pic1.imgdb.cn/item/69c7aea1f2426dbb8efa29f7.png)

成功登录到了前面扫出来的 phpMyAdmin 后台

![](https://pic1.imgdb.cn/item/69c7aeddf2426dbb8efa2a03.png)

在数据库中找到一个和之前扫出的目录同名的

![](https://pic1.imgdb.cn/item/69c7b220f2426dbb8efa2b00.png)

## 文件上传

### Zenphpoto（CVE-2020-36079）

访问 Web 页面发现需要数据库的配置信息

![](https://pic1.imgdb.cn/item/69c7b28ff2426dbb8efa2b10.png)

使用上面的账号密码初始化进去

![](https://pic1.imgdb.cn/item/69c948b4e4fb544c99bb1047.png)

完成安装

![](https://pic1.imgdb.cn/item/69ca1b4a9547e6ce4e3e7a86.png)

再完成后续的部署步骤后登录进去

![](https://pic1.imgdb.cn/item/69ca1d2f9547e6ce4e3e7e15.png)

在插件模块中开启这个功能

![](https://pic1.imgdb.cn/item/69ca2e789547e6ce4e3eb5f7.png)

上传 PHP 木马

![](https://pic1.imgdb.cn/item/69ca311f9547e6ce4e3ebcc3.png)

连接成功

![](https://pic1.imgdb.cn/item/69ca31049547e6ce4e3ebc97.png)

## 提权

### Python 升级为交互式 Shell

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
```

### NFS 配置不当

因为之前 NMap 扫描出挂载了 NFS，所以我们先看看共享列表

```
showmount -e localhost
```

![](https://pic1.imgdb.cn/item/69ca34da9547e6ce4e3ecaf6.png)

这里我们先创建一个目录

然后将靶机的共享的 `/tmp` 目录挂载到创建的目录中

再随便创建一个文件并查看权限是不是 root

如果文件的属主是 **root**，说明 `no_root_squash` 生效（客户端的 root 映射为服务器端的 root）

![](https://pic1.imgdb.cn/item/69ca370b9547e6ce4e3ed257.png)

我们可以将靶机的 `/bin/bash` 拷贝到 `/tmp` 目录中

```bash
cp /bin/bash pwn
```

然后在 Kali 上修改权限并设置 SUID

![](https://pic1.imgdb.cn/item/69ca39c09547e6ce4e3edd07.png)

靶机运行提权成功

![](https://pic1.imgdb.cn/item/69ca39fe9547e6ce4e3eddfc.png)