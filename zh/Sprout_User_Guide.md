# Zcash 1.0 "发芽" 版本指南

欢迎！这个指导的意图是让你学会运行 Zcash 的官方网络。Zcash 当前有一些局限性：它仅支持 Linux 系统，并且需要 64 位系统，在某些情况下它需要大量的存储空间和 CPU 计算来创造交易。

如果你遇到困难，请告诉我们。我们计划在未来改进我们的软件，让它不需要如此高的存储空间和 CPU 算力，并且运行在更多的系统上。

## 软件升级？

如果你曾经在测试网络上使用过 alpha/beta/rc 等版本，请确保你的 `~/.zcash/zcash.conf` 中并没有包涵 `testnet=1` 或是 `addnode=betatestnet.z.cash` 。如果你使用的是基于 Debian 的分发机制，你可以参考 [Debian 指导](https://github.com/zcash/zcash/wiki/Debian-binary-packages) 来在你的系统上安装 Zcash。另外，你可以升级您本地对于我们代码的快照：

```
git fetch origin
git checkout v1.0.10-1
./zcutil/fetch-params.sh
./zcutil/build.sh --disable-rust -j$(nproc)
```
注意：如果你没有安装``nproc``, 那么用你系统的CPU核心数代替nproc命令。如果build命令执行时内存溢出了，请去掉``-j``参数重试，例如：``./zcutil/build.sh --disable-rust`` 。

如果你是从测试网络升级的，请确保初始时您的 ``~/.zcash`` 目录仅含有 ``zcash.conf``，并且``~/.zcash/zcash.conf``不包含``testnet=1``和``addnode=testnet.z.cash``这两项配置。

如果build失败了，删除你的``zcash``文件夹，然后根据下面的[自己编译](https://github.com/zcash/zcash-docs/blob/master/zh/Sprout_User_Guide.md#使能-cpu-挖矿#自己编译)部分操作。

## 关于术语的简要说明

Zcash 支持两种不同的地址，一种是 _z-addr_ (以 **z** 作为开头) 地址，使用了零知识证明和其它密码学来保护用户隐私。另一种是 _t-addrs_ (以 **t** 作为开头)地址，与比特币地址类似。

## 配置需求

目前，你将会需要：

* Linux 系统 (基于 Debian 的发行版)
* 64位的 CPU 和 操作系统
* 4GB 空余存储空间
* 至少10GB的剩余磁盘空间（区块链所占空间会随着时间的推移越来越大）

目前的界面是命令行客户端 (`zcash-cli`) 和远程过程调用(RPC)接口，它们被记录在这里：

https://github.com/zcash/zcash/blob/v1.0.3/doc/payment-api.md

## 安全性

在安装，升级或运行 Zcash 之前，请确认你已经检查安全性问题。请查看安全页面：

https://z.cash/zh/support/security.html

## 开始

### 基于 Debian 的操作系统

在这里获得指导：
https://github.com/zcash/zcash/wiki/Debian-binary-packages

### 自己编译

#### 安装依赖

在基于 Ubuntu 或者 Debian 的系统中:

```bash
$ sudo apt-get install \
build-essential pkg-config libc6-dev m4 g++-multilib \
autoconf libtool ncurses-dev unzip git python python-zmq\
zlib1g-dev wget bsdmainutils automake
```

在基于 Fedora 的系统中：

```bash
$ sudo dnf install \
git pkgconfig automake autoconf ncurses-devel python \
python-zmq wget gtest-devel gcc gcc-c++ libtool patch
```

在基于RHEL的系统中（包括科学研究发行版）：
* 安装 devtoolset-3 和 autotools-latest(如果之前没有安装过)
* 运行 ``scl enable devtoolset-3 'scl enable autotools-latest bash``，然后完成剩余的构建。

#### 检查gcc版本

需要gcc/g++ 4.9或更高版本。但是，目前gcc/g++ 7.x中有一个bug，所以不能使用7.x版本。可以使用``g++ --version``来检查你的是哪个版本。

在Ubuntu Trusty中，你可以像这样安装gcc/g++ 4.9:
```bash
$ sudo add-apt-repository ppa:ubuntu-toolchain-r/test
$ sudo apt-get update
$ sudo apt-get install g++-4.9
```

#### 检查binutils版本
binutils需要2.22或者更高版本。用``as --version``来检查你的是哪个版本，如果没有达到版本要求的话请升级。

#### 获取软件和参数文件

通过使用 git 和运行 `fetch-params.sh`， 来获取我们的程序库：

```bash
$ git clone https://github.com/zcash/zcash.git
$ cd zcash/
$ git checkout v1.0.10-1
$ ./zcutil/fetch-params.sh
```

这将抓取我们发芽版本的验证密钥(最终的版本被建立在[参数生成仪式](https://github.com/zcash/mpc))，并已经把它们放入 `~/.zcash-params/`。这些密钥大约占用 911 MB 存储空间，因此下载它们需要一些时间。

这些由 ``git checkout`` 打印的关于"分离的文件头部（detached head）"的信息是正常的，并没有问题。

#### 构建（build）

确保您已经完整安装了以上提到的所有系统依赖包。之后运行构建程序，比如：

```bash
$ ./zcutil/build.sh --disable-rust -j$(nproc)
```

这样做会编译我们的从属文件，并建造 `zcashd`。(注意：如果你没有安装``nproc``, 那么用你系统的CPU核心数代替nproc命令。如果build命令执行时内存溢出了，请去掉``-j``参数重试，例如：``./zcutil/build.sh --disable-rust`` 。)

#### 测试

这项测试需要运行一段时间，也许会需要 8 GB 的 RAM。如果你想立即开始，你可以跳至下面的章节。如果你想要运行测试来验证 Zcash 的网络正在工作，请运行：

```bash
$ ./qa/zcash/full-test-suite.sh
```

你同样可以运行 RPC 测试，那样做会花的时间更长一点：

```bash
$ ./qa/pull-tester/rpc-tests.sh
```

这项测试需要许多存储器运行正确。内存不足的错误通常会导致失败或错误出现，同时在输出中出现 "std::bad_alloc" 。

## 配置

建立 `~/.zcash` 目录，并在 `~/.zcash/zcash.conf` 中使用以下命令行放置配置文件。

```bash
mkdir -p ~/.zcash
echo "addnode=mainnet.z.cash" >~/.zcash/zcash.conf
echo "rpcuser=username" >>~/.zcash/zcash.conf
echo "rpcpassword=`head -c 32 /dev/urandom | base64`" >>~/.zcash/zcash.conf
```

注意：这样做将覆盖所有你之前在测试网络中对 `zcash.conf` 内部的设置。你可以在测试网络中保留 `zcash.conf` ，但请注意 `testnet=1` 和 `addnode=betatestnet.z.cash` 两个设置要删除；请使用 `addnode=mainnet.z.cash` 。我们强烈建议您使用随机密码来避免 [在进入 RPC 接口时的潜在安全问题](https://github.com/zcash/zcash/blob/master/doc/security-warnings.md#rpc-interface)。

### 启用 CPU 挖矿:

如果你需要启用 CPU 挖矿，请运行以下命令行：

```bash
$ echo 'gen=1' >> ~/.zcash/zcash.conf
$ echo "genproclimit=-1" >> ~/.zcash/zcash.conf
```

设置``genproclimit=-1``可以使用CPU的最大线程来挖矿。如果你想设置一个低一点的线程数来挖矿，那么把它设置为你想要的数字就行。


默认的挖矿程序并不高效，但已经被很好的审阅过了。如果想要运行更加高效但并没有被审阅过的解决方案，你可以运行以下命令行：

```bash
$ echo 'equihashsolver=tromp' >> ~/.zcash/zcash.conf
```

注意，你也许想要阅读 [挖矿指导](https://github.com/zcash/zcash-docs/blob/master/zh/Mining_Guide.md) 来了解更多挖矿的细节。

## 运行 Zcash:

现在，运行 zcashd！

```bash
$ ./src/zcashd
```

想要在后台运行程序 (没有像往常一样可见的窗口展示)，可以使用 ``./src/zcashd --daemon``。

你可以在 RPC 加载完毕后使用它。以下是一个快速测试它的方法：

```bash
$ ./src/zcash-cli getinfo
```

**注意**: 如果你对于比特币的 RPC 接口熟悉，你可以使用很多其中的命令来在 `t-addr` 的地址之间发送 ZEC。我们并不支持 '账户' 特性(这项功能同样被 ``bitcoind`` 所弃用) — 仅用空串 ``""`` 可以命名一个账户。

**注意**: 在 mainnet.z.cash 的主网节点同样可以通过 Tor 的匿名服务 zcmaintvsivr7pcn.onion 所访问。

想要看到你链接到的节点：

```bash
$ ./src/zcash-cli getpeerinfo
```

## 使用 Zcash

第一，如果你想要得到 Zcash ，你可以从交易所、普通用户或服务供应商处购买。如果(安全地)得到 Zcash，并不在本文的讨论范畴，但你需要保持警惕。避免被骗！

### 生成一个 t-addr

让我们首先生成一个 t-addr。

```bash
$ ./src/zcash-cli getnewaddress
t14oHp2v54vfmdgQ3v3SNuQga8JKHTNi2a1
```

### 使用 z-addr 接收 Zcash

现在，让我们生成一个 z-addr.

```bash
$ ./src/zcash-cli z_getnewaddress
zcBqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG
```

这些命令生成了一个私有的地址，并将私钥存储于您的本地钱包文件中。可以把这个地址交付给要发币给您的人！

一个 z-addr 是非常长的，因此使用它很可能会犯错误。让我们把它变成一个环境变量来避免错误：

```bash
$ ZADDR='ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG'
```

为了得到你钱包中所有消费过的地址列表，可以运行以下命令：

```bash
$ ./src/zcash-cli z_listaddresses
```

你应该可以看到以下的内容：
```json
[
"zta6qngiR3U7HxYopyTWkaDLwYBd83D5MT7Jb9gpgTzPLMZytzRbtdPP1Syv4RvRgHeoZrJWSask3DyfwXG9DGPMWMvX7aC",
"ztbqWB8VDjVER7uLKb4oHp2v54v2a1jKd9o4FY7mdgQ3gDfG8MiZLvdQga8JK3t58yjXGjQHzMzkGUxSguSs6ZzqpgTNiZG"
]
```

很好！现在，发送你的 z-addr 给要给你发币的人。你可以检查下面的信息来最终看到他们发送的转账：

```bash
$ ./src/zcash-cli z_listreceivedbyaddress "$ZADDR"
```
```json
[
{
"txid" : "af1665b317abe538148114a45322f28151925501c081949cc7a5207ef21cb750",
"amount" : 1.23,
"memo" : "48656c6c6f20ceb2210000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
}
]
```

### 使用 z-addr 发送你的货币

如果某个人给你了他们的 z-addr...

```bash
$ FRIEND='ztcDe8krwEt1ozWmGZhBDWrcUfmK3Ue5D5z1f6u2EZLLCjQq7mBRkaAPb45FUH4Tca91rF4R1vf983ukR71kHyXeED4quGV'
```

你可以发送 0.8 个 ZEC 给他，步骤如下...

```bash
$ ./src/zcash-cli z_sendmany "$ZADDR" "[{\"amount\": 0.8, \"address\": \"$FRIEND\"}]"
```

在等待了大约 1 分钟之后，你可以查看进程是否结束，预期的结果是：
```bash
$ ./src/zcash-cli z_getoperationresult
```
```json
[
{
"id" : "opid-4eafcaf3-b028-40e0-9c29-137da5612f63",
"status" : "success",
"creation_time" : 1473439760,
"result" : {
"txid" : "3b85cab48629713cc0caae99a49557d7b906c52a4ade97b944f57b81d9b0852d"
},
"execution_secs" : 51.64785629
}
]
```
### zcash-cli的其它操作
介于Zcash是基于bitcoin的扩展，zcash-cli支持所有的Bitcoin Core API(从0.11.2版本开始)，参见：
https://en.bitcoin.it/wiki/Original_Bitcoin_client/API_calls_list

要查看完成的新命令列表，即非Bitcoin API的部分（大部分是z-addrs的操作），参见：
https://github.com/zcash/zcash/blob/master/doc/payment-api.md

列出所有的zcash命令：``./src/zcash-cli help``

获取具体某个命令的帮助文档：``./src/zcash-cli help <command>``

## 已知的安全问题

每一个版本更新都包涵一个名为 `./doc/security-warnings.md` 的文档，它描述了这个版本所包涵的已知安全问题。你可以在这里找到最新版本的文档：

https://github.com/zcash/zcash/blob/master/doc/security-warnings.md

请同时查看我们的安全页面来了解近期的通知和其他资源：

https://z.cash/zh/support/security.html
