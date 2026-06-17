---
title: 2020 BJDCTF WriteUp
date: 2020-04-14 13:18:00
categories: 
  - CTF
tags:
  - CTF
---

# 2020 BJDCTF WriteUp

主办方：由江苏科技大学、北京工业大学、西南民族大学、杭州师范大学、江苏大学、湖南工业大学、新疆大学七校联合举办。北京工业大学的官方信息也确认参与了本次赛事的组织

## Misc

### just a rar（RAR 爆破 + Hex 隐写）

![](https://pic1.imgdb.cn/item/6a30c746fdc5e0ecc37770fc.png)

rar的名字是四位数，那么密码应该也是四位数

![](https://pic1.imgdb.cn/item/6a30c653fdc5e0ecc3777008.png)

WinHex 打开拿到 flag

![](https://pic1.imgdb.cn/item/6a30c722fdc5e0ecc37770ed.png)

### 你猜我是个啥（Hex 隐写）

![](https://pic1.imgdb.cn/item/6a30fc19b67c7e4f4a84a7f7.png)

WinHex 打开拿到 flag

![](https://pic1.imgdb.cn/item/6a30fb9db67c7e4f4a847e1a.png)

### 藏藏藏（文件附加隐写）

![](https://pic1.imgdb.cn/item/6a30fd71b67c7e4f4a84a908.png)

010 看到文件中有 PK 头，推测还附加有压缩文件

![](https://pic1.imgdb.cn/item/6a30fd9eb67c7e4f4a84a91b.png)

使用 BinWalk 提取出来后有个名为 "福利.doc" 的文件，打开发现有个二维码

![](https://pic1.imgdb.cn/item/6a30fdc7b67c7e4f4a84a936.png)

### 认真你就输了（文件附加隐写）

![](https://pic1.imgdb.cn/item/6a30fe0db67c7e4f4a84a982.png)

解压缩文件，给了一个 XLS 文件，打开该文件，发现是 PK 头

![](https://pic1.imgdb.cn/item/6a30fe4eb67c7e4f4a84a9f0.png)

改为 `.zip` 后缀解压缩拿到 flag

![](https://pic1.imgdb.cn/item/6a30fe7fb67c7e4f4a84aa2f.png)

![](https://pic1.imgdb.cn/item/6a30fe8cb67c7e4f4a84aa6a.png)

### 签个到（文件后缀名欺骗）

![](https://pic1.imgdb.cn/item/6a30fee3b67c7e4f4a84aae7.png)

下载压缩包解压报错，WinHex 打开发现是 PNG 文件头

![](https://pic1.imgdb.cn/item/6a30ff18b67c7e4f4a84ab31.png)

改后缀为 PNG 是一张二维码，扫码得到 flag

![](https://pic1.imgdb.cn/item/6a30ff3cb67c7e4f4a84ab68.png)

## Crypto

### sign in（签到）

![](https://pic1.imgdb.cn/item/6a310015b67c7e4f4a84ac90.png)

直接 Hex 转 ASCII 码即可

![](https://pic1.imgdb.cn/item/6a30fff9b67c7e4f4a84ac70.png)