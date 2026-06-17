---
title: 区块链基础
date: 2026-05-22 14:18:00
categories: 
  - 区块链
tags:
  - 区块链
---

# 区块链基础

## 智能合约

**生活类比：自动售货机**

- 传统的买卖：你给老板钱，老板给你水

- 智能合约：一台自动售货机

  - 你把钱塞进去，按下按钮，水自动掉下来

  - 它的规则是写死在机器里的（**代码**）

  - 只要条件满足，就一定会执行（**吐水**）

  - 没有人能中途反悔（**去中心化**）

总结：智能合约就是运行在区块链上的、自动执行的计算机程序

而 **Solidity**，就是用来写这种程序的编程语言

## Solidity 基础

### 变量类型

在 Solidity 中，有几种最常用的变量

#### uint（无符号整数）

```
uint256 public totalMoney;
uint8 public gameId;
```

#### address（地址）

以太坊里每个人的 "银行卡号"，是一串 42 位的字符（如 `0x123...abc`）

别人给你转账，必须知道你的 `address`

```
address public boss;
```

#### mapping（映射）

用来建立 "一对一" 的查询关系

```
// 语法：mapping(查什么 => 得到什么) 名字;
mapping(address => uint256) public balances;
```

### 函数

合约里有很多功能（函数），我们需要规定**谁有权限调用它们**

- `public` (公开)

- `external` (外部)

- `internal` (内部)

- `private` (私有)

### `payable`

收钱：`payable` 关键字

默认情况下，智能合约是**拒收现金**的

如果你想让一个功能可以接收别人打来的以太币（ETH），必须给它贴上 `payable`（可支付）的标签

核心变量：

- `msg.sender`：投币人的地址

- `msg.value`：他投了多少钱？

### 转账

有三种方式把钱打给别人：

- `transfer`：死板的快递员，只给你 2300 步的时间签收，超时直接退回（报错）

- `send`：随意的快递员，超时不报错，只告诉你 "发送失败"

- `call`：灵活的快递员，一直等你签收，并且告诉你详细结果

### 事件

合约执行完一个操作后，向外界发出的 "通知"

你在游戏里充值了 100 元，系统广播："玩家张三充值 100 元！"

```
event 充值成功(address 谁, uint 多少钱); // 定义广播内容
emit 充值成功(msg.sender, 100);         // 触发广播
```

### 修饰符

在执行核心功能前，先做一轮 "安检"

去银行取大额现金，**保安（Modifier）** 先检查你是不是本人，确认无误后，才让柜员（函数） 给你办业务

```
require(你的余额 >= 取款金额, "余额不足，滚蛋！");
// 如果条件不满足，直接中断，并喊出后面的错误提示。
```

### 合约调用

一个合约可以去使用另一个合约的功能

```
import "./微信支付合约.sol"; // 引入工具包
```

### 实战

简单的存取款本

```
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract SimpleLedger {
    // ① uint：记录总存款金额（不能为负）
    uint256 public totalBalance;

    // ② address：记录合约部署者（管理员）地址
    address public owner;

    // ③ mapping：地址 -> 该地址的存款余额
    mapping(address => uint256) public balances;

    // 事件，便于在浏览器/控制台观察存款行为
    event Deposited(address indexed who, uint256 amount);

    // 构造函数：初始化 owner 为合约创建者
    constructor() {
        owner = msg.sender;
    }

    // 存款函数：调用者向合约存入一定数量的 wei（以太币最小单位）
    // payable：允许函数在调用时接收以太币
    function deposit() external payable {
        require(msg.value > 0, "Deposit amount must be positive");
        // 从账本中取出当前余额，加上本次存款金额。将新余额写回账本
        balances[msg.sender] += msg.value;
        // 统计所有用户存入的总金额
        totalBalance += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    // 查询任意地址的余额
    function getBalance(address who) external view returns (uint256) {
        return balances[who];
    }

    // 仅管理员可以查看合约总余额（另一种只读方式）
    function getTotalBalance() external view returns (uint256) {
        return totalBalance;
    }
}
```

打开 https://remix.ethereum.org/ 在线部署

新建一个 `SimpleLedger.sol` 文件在 `contracts` 文件夹中，把代码粘贴进去

Ctrl + S 自动编译

在 Deploy & run transactions 中点击部署合约

![](https://pic1.imgdb.cn/item/6a19f288dd7746a9b033ab98.png)

可以看到当前合约部署者的存款金额为 0

![](https://pic1.imgdb.cn/item/6a19f34add7746a9b033abc2.png)

选择我们手写的存款函数

![](https://pic1.imgdb.cn/item/6a19f3bfdd7746a9b033abdc.png)

开始转账（单位是 wei，比如 1000000000000000000 代表 1 ETH）

在 Debug 中查看转账记录

- `0x4e3795...b16b42`：这笔交易的唯一标识符，用于全网查询

- Tx Fee：交易消耗的总手续费 = Gas Used × Gas Price

- Block：这笔交易被打包到的区块高度，在 Remix VM 中区块号从 0 开始递增

- Tx Type：0 代表 Legacy 交易

- Timestamp：区块头中记录的时间戳

- Gas Price：每单位 gas 的价格，交易发送者愿意支付的单价

- From：交易发起者的地址

- Gas Used：这笔交易实际消耗的 gas 数量

- To：交易接收方地址

- Tx Index：这笔交易在该区块中的索引位置

- Function：Remix 根据 ABI 解析出调用的函数名

- Tx Nonce：该账户发起的交易序号，从 0 开始递增，每发送一笔加 1。防止重放攻击，并确保交易顺序

- Value：交易附带转移的以太币数量

- SENDER：当前调用上下文的 msg.sender

- CALL：EVM 指令操作码，表示发起一个子调用

![](https://pic1.imgdb.cn/item/6a19f40bdd7746a9b033abec.png)

通过 `getBalance` 函数可以查询交易发起者的存款

![](https://pic1.imgdb.cn/item/6a1a438ddd7746a9b0345060.png)

## 交易

交易就是 "你让区块链做一件事的请求"

在我们上面的实战中：

- 你点了 `deposit` 按钮，填了金额 1 ETH，然后点击发送

- 那一瞬间，你就发起了一笔交易

- 这笔交易的内容是：调用 SimpleLedger 合约的 deposit 函数，并附带 1 ETH

交易包含的最核心信息：

- `from` 地址

- `to` 地址，可以是普通账户或合约

- `value`，单位 ETH / wei

- 调用哪个函数、带什么参数

## 区块

区块就是 "装了很多交易的篮子"

每个区块里主要装着：

- 一大堆交易（比如几十上百笔）

- 这一块的 "编号"（区块高度）

- 这一块产生的时间戳

- 上一块的 "指纹"（哈希值）—— 这就是 "链" 的来源

在上面的实战中，我们的交易被打包进了 "Block 3"（如 debug 信息里的 Block: 3）

区块链就是所有区块前后链接形成的历史总账

## 全局变量

每个区块被打包出来的时候，都会自带一组固定的 "出厂参数" 叫——全局变量

- block.number	区块高度

- block.timestamp	时间戳

- block.prevrandao（合并后）	伪随机数 (合并后)

- block.coinbase	出块者地址

## Gas

Gas 是你在以太坊上做任何操作需要燃烧的燃料

以太坊上执行 `deposit` 代码 → 每步计算要烧 Gas，Gas 费付给矿工/验证者

- Gas Limit：你愿意为这笔交易支付的最大手续费

- Gas Used：这笔交易实际手续费多少

- Gas Price	单位价格（单位：Gwei）

如果没有 Gas：

- 某人写一个无限死循环的合约，所有节点就会永远执行下去，卡死

- 或者有人发海量交易，网络堵塞

有了 Gas：

- 每个操作都有成本，你的代码执行超过 Gas Limit → 自动停止，已消耗的 Gas 不退

## 矿工/验证者

矿工（PoW 时代）或验证者（PoS 时代）是负责打包区块、执行交易、维护网络安全的人

合并前（PoW）：

- 矿工用电脑算数学题，谁先算出谁有权打包下一个区块

- 赚取：区块奖励 + 交易费（Gas）

合并后（PoS）：

- 验证者质押 32 ETH 换取出块权

- 赚取：提议者奖励 + 交易费

## EOA 和合约账户

| 对比维度 | EOA（外部拥有账户） | 合约账户 |
| :--- | :--- | :--- |
| 一句话定义 | 你自己手里的银行卡/钥匙 | 自动售货机 / 程序机器人 |
| 有没有私钥 | ✅ 有（你保管） | ❌ 没有（代码控制） |
| 能不能主动发起交易 | ✅ 能（你签名发送） | ❌ 不能（只能被动响应） |
| 有没有代码 | ❌ 没有 | ✅ 有（Solidity 编译后的字节码） |
| 有没有存储（storage） | ❌ 没有（只有余额和 nonce） | ✅ 有（可以存数据，比如 balances mapping） |
| 典型例子 | 你的 MetaMask 地址、交易所提现地址 | 我们写的 SimpleLedger 合约地址 |

## 接口

接口 = 只声明函数的名字、参数、返回值，不写函数体

```
// IERC20.sol
// 只声明，不实现
interface IERC20 {
    // 查询某个地址的余额
    function balanceOf(address account) external view returns (uint256);
    
    // 转账
    function transfer(address to, uint256 amount) external returns (bool);
    
    // 允许别人代你转账
    function approve(address spender, uint256 amount) external returns (bool);
    
    // 查询授权额度
    function allowance(address owner, address spender) external view returns (uint256);
    
    // 事件（接口里也可以有事件声明）
    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(address indexed owner, address indexed spender, uint256 value);
}
```