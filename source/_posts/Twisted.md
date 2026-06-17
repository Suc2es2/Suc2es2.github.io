---
title: MazeSec Twisted
date: 2025-04-01 17:37:00
categories: 
  - MazeSec
tags:
  - Stegseek 隐写
  - 弱口令
  - Capabilities 提权
  - SUID 提权
---

# Twisted

项目地址：https://downloads.hackmyvm.eu/twisted.zip

是一个打包好的镜像文件

**注意：导入 ova 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/6a252a76d71dee7c94195e55.png)

## CTF

### Stegssek 隐写

访问 80 端口有图片

![](https://pic1.imgdb.cn/item/6a2530fad71dee7c94196036.png)

使用 stegseek 提取隐藏信息

![](https://pic1.imgdb.cn/item/6a255b9bd71dee7c9419eeee.png)

![](https://pic1.imgdb.cn/item/6a255d4ed71dee7c941a1d33.png)

## 弱口令

### SSH

推测 meteo 是用户名，thisismypassword 是密码

![](https://pic1.imgdb.cn/item/6a255bfbd71dee7c9419eef5.png)

提示有个 `gogogo.wav` 文件，但是后续没有用

![](https://pic1.imgdb.cn/item/6a255c54d71dee7c9419ef04.png)

切换用户登录，用户名密码是上面的第二张图片

![](https://pic1.imgdb.cn/item/6a255d96d71dee7c941a1d36.png)

## 提权

### Capabilities 提权

查看上面提示给出的文件，只有 root 能读取

使用 `getcap` 查看文件能力。当一个文件被设置了能力，它的行为会受到特殊影响

- `cap_setuid`：允许程序修改自己的UID，即把自己变成 root

- `cap_net_bind_service`：允许程序绑定小于1024的特权端口

- `cap_sys_ptrace`：允许程序读写其他进程的内存，可用于注入代码

- `cap_dac_read_search`：允许程序绕过文件系统的所有读权限检查，直接读取任何文件

![](https://pic1.imgdb.cn/item/6a255dbad71dee7c941a1d39.png)

可以看到 `tail` 被设置了 `cap_dac_read_search`，所以我们使用这个命令直接读取上面给出的文件

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEA8NIseqX1B1YSHTz1A4rFWhjIJffs5vSbAG0Vg2iTa+xshyrmk6zd
FyguFUO7tN2TCJGTomDTXrG/KvWaucGvIAXpgV1lQsQkBV/VNrVC1Ioj/Fx3hUaSCC4PBS
olvmldJg2habNOUGA4EBKlTwfDi+vjDP8d77mF+rvA3EwR3vj37AiXFk5hBEsqr9cWeTr1
vD5282SncYtJb/Zx0eOa6VVFqDfOB7LKZA2QYIbfR7jezOdX+/nlDKX8Xp07wimFuMJpcF
gFnch7ptoxAqe0M0UIEzP+G2ull3m80G5L7Q/3acg14ULnNVs5dTJWPO2Fp7J2qKW+4A5C
tt0G5sIBpQAAA8hHx4cBR8eHAQAAAAdzc2gtcnNhAAABAQDw0ix6pfUHVhIdPPUDisVaGM
gl9+zm9JsAbRWDaJNr7GyHKuaTrN0XKC4VQ7u03ZMIkZOiYNNesb8q9Zq5wa8gBemBXWVC
xCQFX9U2tULUiiP8XHeFRpIILg8FKiW+aV0mDaFps05QYDgQEqVPB8OL6+MM/x3vuYX6u8
DcTBHe+PfsCJcWTmEESyqv1xZ5OvW8PnbzZKdxi0lv9nHR45rpVUWoN84HsspkDZBght9H
uN7M51f7+eUMpfxenTvCKYW4wmlwWAWdyHum2jECp7QzRQgTM/4ba6WXebzQbkvtD/dpyD
XhQuc1Wzl1MlY87YWnsnaopb7gDkK23QbmwgGlAAAAAwEAAQAAAQAuUW5GpLbNE2vmfbvu
U3mDy7JrQxUokrFhUpnJrYp1PoLdOI4ipyPa+VprspxevCM0ibNojtD4rJ1FKPn6cls5gI
mZ3RnFzq3S7sy2egSBlpQ3TJ2cX6dktV8kMigSSHenAwYhq2ALq4X86WksGyUsO1FvRX4/
hmJTiFsew+7IAKE+oQHMzpjMGyoiPXfdaI3sa10L2WfkKs4I4K/v/x2pW78HIktaQPutro
nxD8/fwGxQnseC69E6vdh/5tS8+lDEfYDz4oEy9AP26Hdtho0D6E9VT9T//2vynHLbmSXK
mPbr04h5i9C3h81rh4sAHs9nVAEe3dmZtmZxoZPOJKRhAAAAgFD+g8BhMCovIBrPZlHCu+
bUlbizp9qfXEc8BYZD3frLbVfwuL6dafDVnj7EqpabmrTLFunQG+9/PI6bN+iwloDlugtq
yzvf924Kkhdk+N366FLDt06p2tkcmRljm9kKMS3lBPMu9C4+fgo9LCyphiXrm7UbJHDVSP
UvPg4Fg/nqAAAAgQD9Q83ZcqDIx5c51fdYsMUCByLby7OiIfXukMoYPWCE2yRqa53PgXjh
V2URHPPhqFEa+iB138cSgCU3RxbRK7Qm1S7/P44fnWCaNu920iLed5z2fzvbTytE/h9QpJ
LlecEv2Hx03xyRZBsHFkMf+dMDC0ueU692Gl7YxRw+Lic0PQAAAIEA82v3Ytb97SghV7rz
a0S5t7v8pSSYZAW0OJ3DJqaLtEvxhhomduhF71T0iw0wy8rSH7j2M5PGCtCZUa2/OqQgKF
eERnqQPQSgM0PrATtihXYCTGbWo69NUMcALah0gT5i6nvR1Jr4220InGZEUWHLfvkGTitu
D0POe+rjV4B7EYkAAAAOYm9uaXRhQHR3aXN0ZWQBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

![](https://pic1.imgdb.cn/item/6a256102d71dee7c941a1dac.png)

前面查看家目录发现还有个用户

![](https://pic1.imgdb.cn/item/6a25617fd71dee7c941a1dbe.png)

直接用私钥登录

![](https://pic1.imgdb.cn/item/6a25619ed71dee7c941a1dbf.png)

### SUID 提权

查看文件有个程序具有 SUID 权限

![](https://pic1.imgdb.cn/item/6a256234d71dee7c941a1dd9.png)

反编译代码如下

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+1Ch] [rbp-4h] BYREF

  printf("Enter the code:\n ");
  scanf("%i", &v4);
  if ( v4 == 5880 )
  {
    setuid(0);
    setgid(0);
    system("/bin/bash");
  }
  else
  {
    puts("\nWRONG");
  }
  return 0;
}
```

运行直接输入 5880 提权成功

![](https://pic1.imgdb.cn/item/6a256287d71dee7c941a1ddd.png)

```
HMVblackcat
HMVwhereismycat
```