# Zcash 挖矿指南

欢迎大家！这篇指南是为大家介绍如何在 Zcash 的主网络进行 Zcash 挖矿，a.k.a. "ZEC"。挖矿的单位是 Sol/s (解决方案每秒)。

如果您遇到障碍，请联系我们。在真正让这项技术便于大家使用之前还有很多工作需要完成，您所提供的信息，将有助于我们发现我们产品的薄弱环节。用户支持，我们推荐使用我们的论坛：

https://forum.z.cash/

## 安装

首先，您需要搭建您本地的 Zcash 节点。请按照 [1.0 用户指导](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md) 中，从开头到 "编译"的章节，按步进行操作，之后再回到这里。(您同样可以进入"测试"环节，如果您需要的话！)

## 配置

按照 [[1.0-User-Guide#configuration]] 中的提示，配置您的节点，包括以下章节的内容， [使能 CPU 挖矿] (https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md).

## 挖矿

现在，开启挖矿！
```bash
$ ./src/zcashd
```

想要在后台运行程序 (并没有通常可见的节点指示屏幕)。

```bash
$ ./src/zcashd -daemon
```

您应该在调试日志中看到以下的输出(`~/.zcash/debug.log`):

```bash
Zcash Miner started
```

恭喜你！你现在已经在 Zcash 主网中运行了挖矿程序。

### 花费挖矿收益

币被挖到了一个 t-addr (透明地址), 但是只能发送到 z-addr(隐私地址)。参考我们 [1.0 用户指导](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md)，来了解如何使用 `z_sendmany` 命令和如何从 t-addr 发送货币到 z-addr. 你需要至少 4 GB 的 RAM 来运行这些功能。

## 修饰

### 挖掘并发币到一个单独地址

在 `zcashd`_ 挖矿程序内部，对于每一个被挖掘出的区块都使用一个新的透明地址。如果你向对于每个被挖掘出的区块，使用同样的地址，请找到以下函数进行设置，`src/miner.cpp` (在以下函数中 `ProcessBlockFound()`) 和 `src/wallet/wallet.cpp` (在以下函数中 `CommitTransaction()`):

```cpp
reservekey.KeepKey();
```

在以上两个函数中去掉或注释掉以上的命令。

### 使用 P2PKH 交易

在 `zcashd`_ 挖矿程序中，继承了比特币对于货币库交易使用的 P2PK。随着之后的发展，比特币区块链使用了 P2PKH 代替了 P2PK；我们已经考虑了[修改内部挖矿程序来使用 P2PKH] (https://github.com/zcash/zcash/issues/945)，但目前没有包涵在 1.0 版本中。

如果你想使用 P2PKH 来进行您的货币库交易，请查看以下命令 `src/miner.cpp` (在函数 `CreateNewBlockWithKey()` 中)：

```cpp
CScript scriptPubKey = CScript() << ToByteVector(pubkey) << OP_CHECKSIG;
```

将其修改为:

```cpp
CScript scriptPubKey = CScript() << OP_DUP << OP_HASH160 << ToByteVector(pubkey.GetID()) << OP_EQUALVERIFY << OP_CHECKSIG;
```
