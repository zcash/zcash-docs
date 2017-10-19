# Zcash 挖矿指南

欢迎大家！这篇指南是为大家介绍如何在 Zcash 的主网络进行 Zcash 挖矿，a.k.a. "ZEC"。挖矿的单位是 Sol/s (解决方案每秒)。

如果您遇到障碍，请联系我们。在真正让这项技术便于大家使用之前还有很多工作需要完成，您所提供的信息，将有助于我们发现我们产品的薄弱环节。用户支持，我们推荐使用我们的论坛：

https://forum.z.cash/

## 安装

首先，您需要搭建您本地的 Zcash 节点。请按照 [1.0 用户指导](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md) 中，从开头到 "编译"的章节，按步进行操作，之后再回到这里。(您同样可以进入"测试"环节，如果您需要的话！)

## 配置

按照 [1.0-User-Guide#configuration](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md#configuration) 中的提示，配置您的节点，包括以下章节的内容， [使能 CPU 挖矿](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md).

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


停止 Zcash 进程守护:

```bash
$ ./src/zcash-cli stop 
```

### 花费挖矿收益

币被挖到了一个 t-addr (透明地址), 但是只能发送到 z-addr(隐私地址)，并且会在交易过程中被完全清除，不会改变其他部分。参考我们的 [1.0 用户指导](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md)来了解如何使用 `z_sendmany` 命令从 t-addr 发送货币到 z-addr， 你需要至少 4 GB 的 RAM 来运行这些功能。

### 矿池

如果你在家里或者单独一人在挖矿，当你加入一个已有的矿池你大概也会用所成就。查看社区维护的[矿池列表](https://www.zcashcommunity.com/mining/mining-pools/)以获得下一步的指导。


### 使用 P2PKH 交易
在 `zcashd` 挖矿程序中，继承了比特币对于货币库交易使用的 P2PK，但是Zcash 1.0.6 以及更新的版本中，跟随比特币的趋势，[默认使用 P2PKH](https://github.com/zcash/zcash/issues/945) 。

## 配置选项

### 挖掘并发币到一个单独地址

在 `zcashd` 挖矿程序内部，对于每一个被挖掘出的区块都使用一个新的透明地址。如果你想对于每个被挖掘出的区块使用同样的地址，你可以在 Zcash 1.0.6 以及更新的版本中使用 -mineraddress= 选项。
