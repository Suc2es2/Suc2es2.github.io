---
title: MFC
date: 2019-05-22 14:20:00
categories: 
  - Reverse
tags:
  - Reverse
---

# MFC

## 介绍

MFC（Microsoft Foundation Classes），即微软基础类库，是微软为 Windows 平台开发者提供的一个 C++ 类库

它的核心价值在于，用 C++ 的 "类" 将复杂的 Windows API (Win32 API) 给 "包裹" 起来，让开发者能通过面向对象的方式去调用

这样做的好处是大大简化了 Windows 图形界面程序的开发工作

## 核心思想与架构

MFC 的精髓在于其六个相互配合的内部机制，包括我们马上会介绍的消息映射和序列化。简单来说：

- 程序启动机制：负责程序启动的流程，从 `CWinApp` 派生对象到 `AfxWinMain` 主函数等

- 窗口创建机制：管理窗口从无到有的过程，如调用 `CWnd::CreateEx`、注册窗口类、最终创建窗口

- 动态创建/运行时类信息(RTTI)机制：允许程序在运行时创建对象或查询类的信息（如类名）

- 消息映射机制：避免虚函数开销的轻量级事件处理方式

- 序列化机制：实现对象持久化存储的 `Serialize` 方法

类继承体系：MFC 的类层次结构清晰，几乎所有类都直接或间接继承自根类 `CObject`

## 实战案例

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