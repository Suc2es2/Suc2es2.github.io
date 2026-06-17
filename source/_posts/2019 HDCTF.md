---
title: 2019 HDCTF WriteUp
date: 2019-05-22 14:18:00
categories: 
  - CTF
tags:
  - CTF
---

# 2019 HDCTF WriteUp

## 前言

主办方：海南大学

## Misc

### 你能发现什么蛛丝马迹吗（Windows pslist 排查进程）

![](https://pic1.imgdb.cn/item/6a02de62c05e72d9171c06bf.png)

下载附件是一个 `.img`，使用 vol2 进行取证，先查看操作系统

```bash
python2 vol.py -f memory.img imageinfo
```

发现是 Windows 2003，给出了三个具体的版本

![](https://pic1.imgdb.cn/item/6a02d943c05e72d9171bc434.png)

查看进程，Server 2003 生命周期中 SP2 是最普遍的版本，所以选择它

```bash
python2 vol.py -f memory.img --profile=Win2003SP2x86 pslist
```

![](https://pic1.imgdb.cn/item/6a02dd35c05e72d9171bd193.png)

dump 出 `explorer.exe`，它的内存里主要是与桌面、任务栏、文件夹浏览相关的代码和数据

```bash
python2 vol.py -f memory.img --profile=Win2003SP2x86 memdump -p 1992 -D ../ 
```

检查字符串发现有一个 `flag.png`

![](https://pic1.imgdb.cn/item/6a02e1e5c05e72d9171c1713.png)


foremost 提取出两个图片

```
key：Th1s_1s_K3y00000
iv：1234567890123456
jfXvUoypb8p3zvmPks8kJ5KtOvmEwOxUZyRG0icraY4=
```

解密拿到 flag

```bash
flag(FOuNd_s0m3th1ng_1n_M3mory}
```

### 信号分析（无线电固定码信号分析）

![](https://pic1.imgdb.cn/item/6a02f8cfc05e72d9171cb2a1.png)

固定码遥控信号的构成

钥匙信号(PT224X) = 同步引导码(8bit) + 地址位(20bit) + 数据位(4bit) + 停止码(1bit)

钥匙信号(PT226X) = 同步引导码(4bit) + 地址位(8bit) + 数据位(4bit) + 停止码(1bit)

![](https://pic1.imgdb.cn/item/6a02f9d7c05e72d9171cb467.png)

![](https://pic1.imgdb.cn/item/6a02f9e1c05e72d9171cb47a.png)

PT2262 是一种常用的无线遥控编码芯片（与 PT2272 配对），广泛应用在车库门、报警器等 315/433MHz 设备上

其编码特点：每个数据位可以有三种状态——0、1、F（浮空）

```
一长一短 → F
两个都短 → 0
两个都长 → 1
```

位宽表示：每位由两个脉冲周期组成

用 Audacity 打开文件

![](https://pic1.imgdb.cn/item/6a02faa8c05e72d9171ce996.png)

```
地址位 + 数据位 = FFFFFFFF0001
所以：flag{FFFFFFFF0001}
```

## Crypto

### bbbbbbrsa（爆破 e）

题目源代码如下：

```python
from base64 import b64encode as b32encode
from gmpy2 import invert,gcd,iroot
from Crypto.Util.number import *
from binascii import a2b_hex,b2a_hex
import random

flag = "******************************"

nbit = 128

p = getPrime(nbit)
q = getPrime(nbit)
n = p*q

print p
print n

phi = (p-1)*(q-1)

e = random.randint(50000,70000)

while True:
	if gcd(e,phi) == 1:
		break;
	else:
		e -= 1;

c = pow(int(b2a_hex(flag),16),e,n)

print b32encode(str(c))[::-1]

# 2373740699529364991763589324200093466206785561836101840381622237225512234632

```

已知条件：

- 已知 `p` 和 `n`，可以直接求出 `q = n // p`

- `e` 为 50000~70000 之间的随机数，并与 `φ(n)` 互素

反转字符串，得到正常 Base64 编码结果

```python
c_rev = "==gMzYDNzIjMxUTNyIzNzIjMyYTM4MDM0gTMwEjNzgTM2UTN4cjNwIjN2QzM5ADMwIDNyMTO4UzM2cTM5kDN2MTOyUTO5YDM0czM3MjM"
c_b64 = c_rev[::-1]
```

Base64 解码得到 `c` 的十进制字符串，再转整数

```python
from base64 import b64decode
c_str = b64decode(c_b64).decode()
c = int(c_str)
```

计算基本参数

```python
p = 177077389675257695042507998165006460849
n = 37421829509887796274897162249367329400988647145613325367337968063341372726061
q = n // p
assert p * q == n
phi = (p-1)*(q-1)
```

爆破 e 并解密

```python
import gmpy2
from Crypto.Util.number import long_to_bytes

for e in range(50000, 70001):
    if gmpy2.gcd(e, phi) != 1:
        continue
    d = gmpy2.invert(e, phi)
    m = pow(c, d, n)

    # 还原 flag：整数 -> 十六进制字符串 -> 字节
    try:
        m_hex = hex(m)[2:]
        if len(m_hex) % 2 == 1:
            continue
        flag_bytes = bytes.fromhex(m_hex)
        if b'flag' in flag_bytes or b'HDCTF' in flag_bytes:
            print("Found e:", e)
            print(flag_bytes.decode())
            break
    except:
        continue
```

得到 `flag{rs4_1s_s1mpl3!}`

### basic rsa（RSA）

![](https://pic1.imgdb.cn/item/6a035bb6c05e72d9171f369f.png)

```python
import gmpy2
from Crypto.Util.number import *
from binascii import a2b_hex,b2a_hex

flag = "*****************"

p = 262248800182277040650192055439906580479
q = 262854994239322828547925595487519915551

e = 65533
n = p*q


c = pow(int(b2a_hex(flag),16),e,n)

print c

# 27565231154623519221597938803435789010285480123476977081867877272451638645710
```

加密流程

- flag 字符串 → `b2a_hex(flag)` 转换为十六进制字符串

- 再 `int(..., 16)` 转为大整数 m

- 计算 `c = m^e mod n`

所以解密只需逆回去

```python
import gmpy2
from Crypto.Util.number import long_to_bytes
from binascii import a2b_hex, b2a_hex

p = 262248800182277040650192055439906580479
q = 262854994239322828547925595487519915551
e = 65533
c = 27565231154623519221597938803435789010285480123476977081867877272451638645710

n = p * q
phi = (p - 1) * (q - 1)
d = gmpy2.invert(e, phi)   # 计算私钥 d
m = pow(c, d, n)            # 解密

# 将整数 m 还原为十六进制字符串，再解码得到 flag
m_hex = hex(m)[2:]          # 去掉 '0x' 前缀
if len(m_hex) % 2 == 1:     # 奇数位则补零
    m_hex = '0' + m_hex
flag = bytes.fromhex(m_hex).decode()
print(flag)
```

得到 `flag{B4by_Rs4_1s_3asy}`

### together（共模攻击）

![](https://pic1.imgdb.cn/item/6a035f2fc05e72d9171f4c7d.png)

拿到题目后，解压附件会得到 4 个文件：

- `pubkey1.pem` 和 `pubkey2.pem`公钥文件

- `myflag1` 和 `myflag2` 密文文件

解题的关键突破口在于对比两个公钥文件

使用 openssl 命令分别提取两个公钥的模数 `N` 和指数 `e`

```bash
openssl rsa -pubin -text -modulus -in pubkey1.pem
openssl rsa -pubin -text -modulus -in pubkey2.pem
```

经过对比，得到两条核心信息：

- 模数 `N` 完全相同

- 公钥指数 `e` 不同（例如 `e1=2333`, `e2=23333`）

两条不同的消息使用了相同的模数 `N` 进行加密，这是典型的**共模攻击**场景

密文其实是 Base64 字符串，进行解码才能得到真正的密文数值 `c1` 和 `c2`

```python
import base64
import gmpy2
from Crypto.PublicKey import RSA

# ----------------- 1. 提取 n, e1, e2 -----------------
with open('pubkey1.pem', 'r') as f:
    key1 = RSA.import_key(f.read())
with open('pubkey2.pem', 'r') as f:
    key2 = RSA.import_key(f.read())

n = key1.n
e1 = key1.e
e2 = key2.e

# ----------------- 2. 读取密文并还原为整数 -----------------
with open('myflag1', 'r') as f:
    f1_str = f.read().strip()
with open('myflag2', 'r') as f:
    f2_str = f.read().strip()

c1_bytes = base64.b64decode(f1_str)
c2_bytes = base64.b64decode(f2_str)
c1 = int.from_bytes(c1_bytes, byteorder='big')
c2 = int.from_bytes(c2_bytes, byteorder='big')

# ----------------- 3. 共模攻击 -----------------
g, s1, s2 = gmpy2.gcdext(e1, e2)   # g=1
if s1 < 0:
    s1 = -s1
    c1 = gmpy2.invert(c1, n)       # 求 c1 模 n 的逆元
elif s2 < 0:
    s2 = -s2
    c2 = gmpy2.invert(c2, n)

m = (gmpy2.powmod(c1, s1, n) * gmpy2.powmod(c2, s2, n)) % n

# ----------------- 4. 整数转 flag -----------------
flag_bytes = int(m).to_bytes((m.bit_length() + 7) // 8, byteorder='big')
print(flag_bytes.decode())
```

得到 `flag{23re_SDxF_y78hu_5rFgS}`

## Reverse

### Maze（花指令绕过 + 迷宫）

![](https://pic1.imgdb.cn/item/6a037595c05e72d9171fafeb.png)

使用 Exeinfo PE 发现有 UPX 壳

![](https://pic1.imgdb.cn/item/6a037660c05e72d9171fb549.png)

脱壳后用 IDA 分析，有花指令

![](https://pic1.imgdb.cn/item/6a03e947c05e72d91720d8f0.png)

关键点：`jnz +1` 的跳转目标

- `jnz short` 指令本身长 2 字节（75 操作码 + 01 偏移量）

- 偏移量是相对下一指令地址的位移。当前 `jnz` 位于 `0x40102C`，下一指令地址是 `0x40102E`

- 加上 `+1`，目标地址 = `0x40102E + 1` = `0x40102F`

- 因此，无论标志位如何，CPU 下一步必然从 `0x40102F` 开始执行

所以，`call 0EC85D78Bh` 这条 5 字节指令的唯一目的就是干扰 IDA 反编译

CPU 实际会执行的部分是从 `0x40102F` 开始的每一个字节，所以这些字节一个都不能动

唯一需要填掉的是 IDA 会顺序解析，但 CPU 永不执行的部分，即：

- `75 01` 这 2 字节的 `jnz`（它虽然会执行，但它的存在让 IDA 沿着 `0x40102E` 往下分析。如果我们 `NOP` 掉它，IDA 就不会进入那条错误的分支了）

- `E8` 这 1 字节的 `call` 操作码

因此，我们只需把 0x40102C 开始的 3 个字节 改为 90：

```
原始： 75 01 E8 58 C7 45 EC ...
修改： 90 90 90 58 C7 45 EC ...
```

保存后再次反编译可以看到伪代码，接受用户输入的 awsd 推测为迷宫类题目

![](https://pic1.imgdb.cn/item/6a03ed3cc05e72d91720d96c.png)

Shift + F12 打开字符串列表

![](https://pic1.imgdb.cn/item/6a03f0c4c05e72d91720d9ee.png)

正好是 70 个字符，对应一个 7 行 × 10 列 的迷宫

```
*******+**
******* **
****    **
**   *****
** **F****
**    ****
*******  *
```

s s a a a s a a s s d d d w → 刚好能走到 F 的位置，所以 flag 就是 `flag{ssaaasaassdddw}`

### MFC（MFC）

![](https://pic1.imgdb.cn/item/6a03793cc05e72d9171fd473.png)

使用 Exeinfo PE 发现有 VMP 壳

![](https://pic1.imgdb.cn/item/6a037929c05e72d9171fd463.png)

这道题是一个 MFC，你可以把 Windows 的图形界面程序想象成一个“事件驱动”的机器，按钮点击、文本输入等所有操作都会被转化为系统消息

而 MFC 框架就像一个黑箱，把这些底层的消息处理过程层层封装，让你很难在 IDA 等静态工具里，通过简单的关键API 或者搜索字符串定位到核心逻辑

再加上程序还加了 VMProtect 壳，雪上加霜

所以这里我要利用到另一个工具——xspy

xspy 是一款专门为逆向分析 MFC 程序设计的辅助工具，它的核心能力是监控目标程序的消息和控件信息

下面开始逆向，运行程序

![](https://pic1.imgdb.cn/item/6a03f3d8c05e72d91720dbf1.png)

用 xspy 的 "放大镜" 拖拽到程序窗口

![](https://pic1.imgdb.cn/item/6a03f535c05e72d91720de52.png)

现在我们来解析下回显内容

**窗口身份确认**

- 窗口类型：主窗口是一个 `CDialog`（对话框），继承自 `CWnd` → `CCmdTarget` → `CObject`

- 句柄值：当前该对话框的窗口句柄（HWND）是 `0x000708FE`。这个值后面可以直接用来发消息

```
class:001AFE60(CDialog,size=0x98)
CDialog:CWnd:CCmdTarget:CObject
HWND: 0x000708FE
```

**框架与链接方式**

- MFC 版本：120，对应 VS2013 的 MFC 库

- 静态链接：程序把 MFC 库静态编译进了自身，因此看不到 `mfc120.dll`，所有逻辑都在同个 exe 里

```
mfc version:120, static linked?: true, debug?: false
```

**消息映射表**

- 标准消息：`WM_SYSCOMMAND`、`WM_PAINT`、`WM_QUERYDRAGICON` 都是系统预设消息，对应普通的功能

- 自定义消息 `0x0464`：`OnMsg:0464` 是程序作者自己定义的，对应的处理函数地址是 `0x00402170`。这意味着，只要向这个窗口发送 `0x0464` 号消息，程序就会自动调用 `0x00402170` 处的函数

```
message map=0x00588CF4(mfc.exe+ 0x188cf4 )
msg map entries at 0x00588D00(mfc.exe+ 0x188d00 )
OnMsg:WM_SYSCOMMAND(0112),func= 0x00401FD0
OnMsg:WM_PAINT(000f),func= 0x00402080
OnMsg:0464,func= 0x00402170        ← 这就是题眼！
OnMsg:WM_QUERYDRAGICON(0037),func= 0x00402160
```

编写代码编译成 exe 发送消息

```c
#include <windows.h>
#include <stdio.h>

int main()
{
    // 查找窗口标题为 "Flag就在控件里" 的窗口
    HWND hWnd = FindWindowA(NULL, "Flag就在控件里");
    if (hWnd == NULL)
    {
        printf("未找到窗口，请确保程序正在运行。\n");
        return 1;
    }

    printf("找到窗口句柄: 0x%X\n", (DWORD_PTR)hWnd);

    // 发送自定义消息 0x0464
    SendMessage(hWnd, 0x0464, 0, 0);

    printf("消息发送成功，请观察目标程序窗口标题变化。\n");

    // 再获取一次窗口标题，看是否变化
    char title[256] = { 0 };
    GetWindowTextA(hWnd, title, sizeof(title));
    printf("当前窗口标题: %s\n", title);

    return 0;
}
```

可以看到控件更新了

![](https://pic1.imgdb.cn/item/6a03f9e56f6a2d474b48dc9b.png)

再次用 xspy 的放大镜拖到窗口上可以看到密文

![](https://pic1.imgdb.cn/item/6a03fa7b6f6a2d474b48dd59.png)

DES 解密得到 `flag{thIs_Is_real_kEy_hahaaa}`