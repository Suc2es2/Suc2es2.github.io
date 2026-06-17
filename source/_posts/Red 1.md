---
title: VulnHub Red：1
date: 2026-06-08 17:14:00
categories: 
  - VulnHub
tags:
  - 文件包含
  - 弱口令
  - /etc/sudoers 配置不当
  - time 提权
  - 反弹 Shell 稳定化
---

# Red：1

项目地址：https://download.vulnhub.com/red/Red.ova

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a26c7923c9809430d36cda0.png)

## 文件包含

### NetworkFileManagerPHP

发现 `/wp-admin/` 目录，打开首页没啥东西

![](https://pic1.imgdb.cn/item/6a26c8623c9809430d36ce21.png)

查看源代码，我们发现主机名为 redrocks.win

![](https://pic1.imgdb.cn/item/6a26c8d13c9809430d36ce69.png)

所以让我们将其添加到 hosts 文件中

![](https://pic1.imgdb.cn/item/6a26c91e3c9809430d36ce8e.png)

嘲讽我们，暗示我们的网站被黑了，他甚至提到网站里有某种后门

![](https://pic1.imgdb.cn/item/6a26c9393c9809430d36ce99.png)

点进这篇博客

![](https://pic1.imgdb.cn/item/6a26ca153c9809430d36cee8.png)

查看源代码找到注释

![](https://pic1.imgdb.cn/item/6a26ca303c9809430d36cefd.png)

AI 梭哈一下，提示扫描后台

![](https://pic1.imgdb.cn/item/6a26cbc03c9809430d371828.png)

发现一个 `NetworkFileManagerPHP.php` 文件

![](https://pic1.imgdb.cn/item/6a26cbea3c9809430d37183e.png)

FUZZ 出参数是 `key`，测试发现不是 RCE，而是文件包含

![](https://pic1.imgdb.cn/item/6a26cd9c3c9809430d3718e1.png)

## 弱口令

### SSH

PHP 协议查看网页源码

![](https://pic1.imgdb.cn/item/6a26d69b3c9809430d371ca8.png)

翻看源码，正常的 LFI 代码，但是有一段 Base64 注释

![](https://pic1.imgdb.cn/item/6a26d7053c9809430d371cbc.png)

AI 梭哈注释，提示是 Hashcat Rules

![](https://pic1.imgdb.cn/item/6a26d7a53c9809430d371cdc.png)

查看 `wp-config.php`

![](https://pic1.imgdb.cn/item/6a26d7b83c9809430d371ce4.png)

Hashcat 的规则引擎可以对字典中的每个基础单词进行系统化变形，自动生成大量密码变体

```
基础单词: "password"
     │
     ▼ 应用规则
     ├── "Password"      (首字母大写)
     ├── "PASSWORD"      (全大写)
     ├── "password123"   (追加数字)
     ├── "p@ssw0rd"      (字符替换 l33t)
     ├── "password!"     (追加特殊字符)
     ├── "drowssap"      (反转)
     └── "P@$$w0rd!2024" (组合变形)
```

将上面的数据库密码保存然后进行变形

```bash
# --stdout：不破解哈希，仅输出结果
# -r：加载一个规则文件
hashcat --stdout passwd.txt -r /usr/share/hashcat/rules/best66.rule > passlist.txt
```

![](https://pic1.imgdb.cn/item/6a26d9213c9809430d3768de.png)

然后使用九头蛇爆破密码

![](https://pic1.imgdb.cn/item/6a26d9583c9809430d3768ee.png)

## 提权

### /etc/sudoers 配置不当

连接上后检查自身权限，可以使用 `time` 提权

![](https://pic1.imgdb.cn/item/6a26d9d93c9809430d37690d.png)

但是过一段时间就会踢掉我们

![](https://pic1.imgdb.cn/item/6a26d9f13c9809430d376911.png)

并且还会换密码

![](https://pic1.imgdb.cn/item/6a26da4c3c9809430d376925.png)

### time 提权

快速提权到 `ippsec` 用户

```bash
sudo -u ippsec /usr/bin/time /bin/bash
```

`/usr/bin/time`	用于测量并报告另一个程序执行所消耗的系统资源（时间、内存等），它接收一个命令作为参数并执行它

![](https://pic1.imgdb.cn/item/6a26da8c3c9809430d376934.png)

### 反弹 Shell 稳定化

快速创建一个反弹 Shell 脚本并运行

![](https://pic1.imgdb.cn/item/6a26dd843c9809430d3769f1.png)

思路如下：

- 使用 Python 升级 PTY

    - `python3 -c 'import pty;pty.spawn("/bin/bash")'`

- 按下 Ctrl+Z，将 `nc` 进程放入后台

- `stty raw -echo;fg`：将你 Kali 本地的终端设置为 "原始模式"

    - `-echo`：关闭本地回显，因为远端的 Bash 已经会把你输入的命令回显给你了，如果本地再回显一次，你打的每个字母都会显示两遍

    - `fg`：将刚才挂起的 nc 进程重新调回前台

- `export TERM=xterm`：修复终端显示

防掉线原理：

- 在 Raw 模式下，本地终端驱动程序不再拦截和处理任何特殊控制字符（如 Ctrl+C, Ctrl+Z, Ctrl+\）

- 当你在键盘上按下 Ctrl+C 时，Kali 不会去杀 `nc` 进程，而是将 Ctrl+C 的原始字节流（`0x03`）直接通过网络透传给远端的目标机器

- 远端的 Bash 接收到 `0x03`，只会终止远端正在运行的命令（比如 `ping` 或 `find`），而不会杀死你的 `nc` 连接

### 定时任务

快速定位属于特定用户组（ippsec）的目录

```bash
find / -group ippsec -type d 2>/dev/null | grep -v proc
```

- `-v`：表示反向匹配（排除）。这里的作用是排除路径中包含 `proc` 字样的结果（主要是为了过滤掉 `/proc` 伪文件系统）

![](https://pic1.imgdb.cn/item/6a26e3043c9809430d376f17.png)

我们拥有其写入权限

![](https://pic1.imgdb.cn/item/6a26e3153c9809430d376f19.png)

在这里面有两个文件

![](https://pic1.imgdb.cn/item/6a26e3283c9809430d376f1e.png)

上传 `psps64s` 文件

![](https://pic1.imgdb.cn/item/6a26e3433c9809430d376f25.png)

运行 `psps64s` 发现这两文件有定时任务

![](https://pic1.imgdb.cn/item/6a26e36b3c9809430d376f31.png)

删除后重新写成一个反弹 Shell

![](https://pic1.imgdb.cn/item/6a26e37e3c9809430d376f3b.png)

等待一段时间即可拿到 Root Shell

![](https://pic1.imgdb.cn/item/6a26e3cb3c9809430d376f48.png)