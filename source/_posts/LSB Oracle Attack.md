---
title: LSB Oracle Attack
date: 2026-06-11 14:18:00
categories: 
  - 现代密码
tags:
  - 现代密码
  - LSB Oracle Attack
---

# LSB Oracle Attack

## 原理

LSB Oracle Attack（最低有效位预言机攻击）是一种针对非对称加密（特别是 RSA）的侧信道攻击。其核心是利用一个能够返回明文最低有效位（即奇偶性，`LSB = m mod 2`）的解密预言机，逐步恢复完整明文

攻击原理：

- 假设有密文 `c = m^e mod N`，攻击者拥有一个 Oracle，对任意输入的密文 `c'` 可以返回 `m' mod 2`

- RSA 具有同态性质：`(c * 2^e) mod N` 对应的明文是 `(2m) mod N`

- 如果 Oracle 返回 `(2m mod N) mod 2 = 0`，说明 `2m < N`，即 `m < N/2`；如果返回 1，则 `2m >= N`，即 `m >= N/2`

- 反复构造 `c * (2^k)^e`，每次缩小明文的可能范围（二分法），经过 `log2(N)` 次查询即可恢复完整明文

## 实战案例

### parityOracle（LSB Oracle Attack）

传统的 LSB Oracle 每次泄露 1 个比特，而本题的 Oracle 每次泄露 2 个比特

```python
# 关键约束
while p*q%4-3:
    p, q = getPrime(1024), getPrime(1024)
...
# Oracle 行为
result = pow(cip, self.d, self.n) % 4
self.send(hex(result)[2:].encode())
```

这段代码暴露了两个致命的问题：

- 循环条件确保了 `p × q mod 4 ≠ 3` 时不断重试，直到 `p × q mod 4 == 3`。也就是说，该分发实例中的 RSA 模数 `n ≡ 3 (mod 4)`

- 用户输入任意密文 `cip`，服务端用私钥 `d` 将其解密为 `m'`，但只返回 `m' mod 4` 的值（结果为 `0, 1, 2, 3` 中的一个）

**第一轮交互（`i=1`）的完整代数推导**

我们向服务器发送构造密文 `c1 = c * 4^e (mod n)`。下面是服务器内部解密并取模的完整步骤

**步骤 1：解密同态变换**

服务器计算 `m1 ≡ (c1)^d (mod n)`。将 `c1` 的定义代入：

```
m1 ≡ (c * 4^e)^d (mod n)
```

根据模幂的积同态性质 `(A * B)^d ≡ A^d * B^d (mod n)`，将指数 `d` 分配进去：

```
m1 ≡ (c^d) * (4^e)^d (mod n)
```

因为 `c ≡ m^e (mod n)`，所以 `c^d ≡ m^(e*d)`

同理 `(4^e)^d ≡ 4^(e*d)`。代入上式得到：

```
m1 ≡ m^(e*d) * 4^(e*d) (mod n)
```

又因为 `m^(e·d) = m^(1 + k·φ(n)) = m · (m^φ(n))^k ≡ m · 1^k = m (mod n)`

所以得到 `m1 ≡ m * 4 (mod n)`

**步骤 2：消除模运算**

根据模运算的定义，若 `A ≡ B (mod n)`，则必然存在一个整数 `k`，使得 `A = k*n + B`。这里将 `A = 4m，B = m1` 代入，得到：

```
4m = k1 * n + m1
```

**步骤 3：限定 `k1` 的取值范围**

已知明文边界为：`0 ≤ m < n`

不等式各项同时乘以 `4：0 ≤ 4m < 4n`

将步骤 2 的等式 `4m = k1*n + m1` 代入上式中间：

```
0 ≤ k1*n + m1 < 4n
```

因为 `m1` 是模 `n` 的余数，所以必然满足 `0 ≤ m1 < n`。为了让上述不等式成立，整数 `k1` 只能在以下集合中取值：

```
k1 ∈ {0, 1, 2, 3}
```

**步骤 4：两边同时取模 4**

将步骤 2 的等式 `4m = k1*n + m1` 两边同时进行 (`mod 4`) 运算：

```
4m (mod 4) = (k1*n + m1) (mod 4)
```

由于 `4m` 显然是 `4` 的倍数，所以 `4m (mod 4) = 0`。左边化简为 `0`：

```
0 ≡ k1*n + m1 (mod 4)
```

移项，将 `m1` 孤立在等式一边

```
m1 ≡ -k1 * n (mod 4)
```

**步骤 5：代入服务器特有条件 `n ≡ 3 (mod 4)`**

已知条件中 `n ≡ 3 (mod 4)`，在模 4 运算中，`3 ≡ -1 (mod 4)`。因此我们可以把 `n` 替换为 `-1`：

将 `n ≡ -1` 代入步骤 4 结尾的方程中：

```
m1 ≡ -k1 * (-1) (mod 4)

m1 ≡ k1 (mod 4)
```

**步骤 6：建立预言机输出与 `k1` 的等价关系**

预言机返回的结果定义为 `R1 = m1 (mod 4)`。根据步骤 5 的结论，`m1 (mod 4) = k1 (mod 4)`

由于我们在步骤 3 中已经严格证明了 `k1 ∈ {0,1,2,3}`，且预言机返回值 R1 也必然在 `{0,1,2,3}` 之间。两个位于 `[0,3]` 区间内的整数如果模 4 同余，则它们必然严格相等：

```
R1 = k1
```

**任意一轮（第 `i` 轮）的递推数学证明**

在第 `i` 轮，我们发送 `ci = c * (4^i)^e (mod n)`，解密得到：

```
Xi ≡ 4^i * m (mod n)
```

写成等式形式：

```
4^i * m = Ki * n + Xi   (0 ≤ Xi < n)
```

我们同样将上一轮（第 `i-1` 轮）的等式列出来：

```
4^(i-1) * m = K_(i-1) * n + X_(i-1)   (0 ≤ X_(i-1) < n)
```

**步骤 1：寻找上下两轮之间的递推关系**

我们将第 `i-1` 轮的等式两边同时乘以 4：

```
4 * (4^(i-1) * m) = 4 * (K_(i-1) * n + X_(i-1))
```

左边合并幂次，右边展开括号：

```
4^i * m = 4K_(i-1) * n + 4X_(i-1)
```

现在，我们单独对 `4X_(i-1)` 这一项进行除以 `n` 的操作，假设其余数为 `Xi`，不完全商为 `k'`

```
4X_(i-1) = k' * n + Xi
```

由于 `0 ≤ X_(i-1) < n`，同理可得这个局部商的范围也是：

```
k' ∈ {0, 1, 2, 3}
```

将 `4X_(i-1) = k'*n + Xi` 代入上面展开的递推式中

```
4^i * m = 4K_(i-1) * n + (k'*n + Xi)
```

提取公因数 `n`

```
4^i * m = (4K_(i-1) + k') * n + Xi
```

**步骤 2：对比系数**

对比本节开头的定义式 `4^i * m = Ki * n + Xi`，由于余数 `Xi` 具有唯一性，我们可以直接对应出 `n` 的系数相等：

```
Ki = 4K_(i-1) + k'
```

**步骤 3：对总商 `Ki` 取模 4**

将上式两边同时 `(mod 4)`：

```
Ki (mod 4) = (4K_(i-1) + k') (mod 4)
```

由于 `4K_(i-1)` 是 4 的倍数，模 4 为 0，因此式子化简为

```
Ki ≡ k' (mod 4)
```

因为 `k' ∈ {0,1,2,3}`，所以：

```
Ki (mod 4) = k'
```

**步骤 4：计算预言机回显与 `k'` 的关系**

第 `i` 轮预言机收到的解密结果是 `Xi`，它返回 `Ri = Xi (mod 4)`。根据本节开头定义式 `4^i * m = Ki * n + Xi`，变形可得

```
Xi = 4^i * m - Ki * n
```

两边同时取模 4

```
Xi (mod 4) = (4^i * m - Ki * n) (mod 4)
```

由于 `i ≥ 1，4^i * m (mod 4) = 0`。式子化简为

```
Xi ≡ -Ki * n (mod 4)
```

再次代入 `n ≡ -1 (mod 4)`

```
Xi ≡ -Ki * (-1) ≡ Ki (mod 4)
```

将步骤 3 的结论 `Ki ≡ k' (mod 4)` 代入上式

```
Xi ≡ k' (mod 4)
```

因为 `Ri = Xi (mod 4)`，且 `Ri` 和 `k'` 都在 [0,3] 范围内，所以它们严格相等

```
Ri = k'
```

最终递推结论：第 `i` 轮预言机返回的 `Ri`，正是 `4X_(i-1)` 除以 `n` 的商 `k'`。也就是说，它反应的是上一轮的余数 `X_(i-1)` 落在 `n` 的哪一个四分之一区间内

既然知道了 `Ri = k' = floor(4X_(i-1) / n)`，我们就可以据此来对明文真实空间 `[lower, upper]` 实施精准切分。根据 `k'` 的值，上一轮的余数空间被划分为四个区间：

- 当 `k' = 0` 时：`0 ≤ 4X_(i-1) < n  =>  X_(i-1) ∈ [0, n/4)`

- 当 `k' = 1` 时：`n ≤ 4X_(i-1) < 2n => X_(i-1) ∈ [n/4, 2n/4)`

- 当 `k' = 2` 时：`2n ≤ 4X_(i-1) < 3n => X_(i-1) ∈ [2n/4, 3n/4)`

- 当 `k' = 3` 时：`3n ≤ 4X_(i-1) < 4n => X_(i-1) ∈ [3n/4, n)`

映射回当前动态维护的明文全局边界 `[lower, upper]`，其物理跨度为 `W = upper - lower`。我们将这个跨度四等分，每一份的权重为 `(upper - lower) / 4`

- 当 `rev == 0 (k'=0)`，说明真实值位于当前区间的第一个四分之一。下界不变：`lower = lower`。上界缩减：`upper = lower + 1 * (upper - lower) // 4`

- 当 `rev == 1 (k'=1)`，说明真实值位于当前区间的第二个四分之一。下界右移：`lower = lower + 1 * (upper - lower) // 4`。上界左移：`upper = lower + 2 * (upper - lower) // 4`

- 当 `rev == 2 (k'=2)`，说明真实值位于当前区间的第三个四分之一。下界右移：`lower = lower + 2 * (upper - lower) // 4`。上界左移：`upper = lower + 3 * (upper - lower) // 4`

- 当 `rev == 3 (k'=3)`，说明真实值位于当前区间的第四个四分之一。上界不变：`upper = upper`。下界右移：`lower = lower + 3 * (upper - lower) // 4`

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
from pwn import *
import hashlib
import string
import re
from Crypto.Util.number import long_to_bytes

# 设置日志级别为 info，如果需要看底层原始交互数据可改为 'debug'
context.log_level = 'info'

# 目标服务器配置
HOST = "127.0.0.1"
PORT = 10001

def solve_pow(r):
    """
    自动化求解工作量证明 (Proof of Work)
    """
    try:
        r.recvuntil(b"sha256(XXXX+")
        suffix = r.recvuntil(b")", drop=True).decode()
        r.recvuntil(b"== ")
        target = r.recvline().strip().decode()
        
        log.info(f"PoW 挑战参数 -> 后缀: {suffix}, 目标哈希: {target}")
        
        alphabet = string.ascii_letters + string.digits
        prefix = util.iters.mbruteforce(
            lambda x: hashlib.sha256((x + suffix).encode()).hexdigest() == target,
            alphabet, 
            4, 
            'exact'
        )
        
        log.success(f"成功爆破出 PoW 前缀: {prefix}")
        r.sendlineafter(b"Give me XXXX: ", prefix.encode())
        return True
    except Exception as e:
        log.error(f"PoW 爆破失败: {e}")
        return False

def get_oracle_mod4(r, cipher_int):
    """
    与服务端交互，利用 Modulo-4 Oracle 获取解密明文的最后 2 个比特
    """
    # 选择菜单 1. decrypt
    r.sendlineafter(b"> ", b"1")
    
    # 发送十六进制格式的密文
    cipher_hex = hex(cipher_int)[2:].encode()
    r.sendlineafter(b"Your cipher (in hex): ", cipher_hex)
    
    # 接收预言机返回的单行结果 (0, 1, 2, 3)
    response = r.recvline().decode().strip()
    return int(response, 16)

def pwn_start():
    # 建立网络连接
    r = remote(HOST, PORT)
    
    # 1. 绕过验证码
    if not solve_pow(r):
        r.close()
        return

    # 2. 接收初始化 RSA 公钥与目标密文
    try:
        r.recvuntil(b"e = ")
        e = int(r.recvline().decode().strip())
        r.recvuntil(b"n = ")
        n = int(r.recvline().decode().strip())
        r.recvuntil(b"c = ")
        c = int(r.recvline().decode().strip())
        
        log.success(f"成功获取公钥与密文:")
        log.info(f"n 的模数位长: {n.bit_length()} bits")
        log.info(f"验证特殊条件 (n % 4): {n % 4} (预期应为 3)")
        assert n % 4 == 3, "模数 n 不满足定理条件 n ≡ 3 (mod 4)！"
    except Exception as e:
        log.error(f"公钥/密文解析异常: {e}")
        r.close()
        return

    # 3. 初始化四分查找的全局物理边界
    # 明文 m 必然处于 [0, n] 之间
    lower = 0
    upper = n
    i = 1  # 放大因子指数

    log.info("开始执行基于 Modulo-4 Oracle 的四分查找...")
    
    # 进度条初始化（2048位模数，每次恢复2位，约需1024次）
    progress = log.progress("明文空间逼近进度")

    # 当区间宽度小于 2 时停止收敛
    while (upper - lower) >= 2:
        progress.status(f"当前区间跨度: {upper - lower}")
        
        # 构造选择密文: c' = c * (4^i)^e 
        # 根据同态性，这会在解密时将明文放大 4^i 倍： m' = m * 4^i
        power = pow(4, i, n)
        chosen_c = (pow(power, e, n) * c) % n
        
        # 获得本轮放大后对应的商数（当前真实值落在上一轮空间的哪个四分之一区间）
        rev = get_oracle_mod4(r, chosen_c)
        
        # 暂存当前边界快照，以防计算公式时上下界互相污染
        old_upper = upper
        old_lower = lower
        
        # 严格执行前文推导的四分搜索状态机
        if rev == 0:
            # 落在第 1 个四分之一区间: [lower, lower + W/4]
            upper = (3 * old_lower + old_upper) // 4
        elif rev == 1:
            # 落在第 2 个四分之一区间: [lower + W/4, lower + 2W/4]
            upper = (old_lower + old_upper) // 2
            lower = (3 * old_lower + old_upper) // 4
        elif rev == 2:
            # 落在第 3 个四分之一区间: [lower + 2W/4, lower + 3W/4]
            upper = (old_lower + 3 * old_upper) // 4
            lower = (old_lower + old_upper) // 2
        elif rev == 3:
            # 落在第 4 个四分之一区间: [lower + 3W/4, upper]
            lower = (old_lower + 3 * old_upper) // 4
        else:
            log.error(f"预言机异常返回非法值: {rev}")
            break
            
        i += 1  # 步进到下一个 4 的更高幂次

    progress.success("区间收敛完毕！")
    log.info(f"收敛结束。最终下界 (lower): {lower}")

    # 4. 消除整数除法带来的截断误差（边界微调）
    log.info("正在对收敛边界邻域进行精确匹配校验...")
    found = False
    # 在最终下界附件上下 100 整数范围内做精确的 RSA 加密比对
    for candidate in range(lower - 100, lower + 100):
        if pow(candidate, e, n) == c:
            flag = long_to_bytes(candidate)
            print("\n" + "="*50)
            log.success("成功完美恢复 Flag 原始明文:")
            print(flag.decode('utf-8', errors='ignore'))
            print("="*50 + "\n")
            found = True
            break
            
    if not found:
        log.error("未能在边界邻域内匹配到有效明文，请检查推导公式截断行为。")

    # 安全关闭连接
    r.sendlineafter(b"> ", b"2")
    r.close()

if __name__ == "__main__":
    pwn_start()
```
