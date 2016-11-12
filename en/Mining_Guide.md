# Zcash Mining Guide

Welcome! This guide is intended to get you mining Zcash, a.k.a. "ZEC", on the Zcash mainnet. The unit for mining is Sol/s (Solutions per second).

If you run into snags, please let us know. There's plenty of work needed to make this usable and your input will help us prioritize the worst sharpest edges earlier. For user help, we recommend using our forum:

https://forum.z.cash/

## Setup

First, you need to set up your local Zcash node. Follow the [1.0 User Guide](1.0 User Guide) up to the end of the section "Compiling", then come back here. (You can also do the "Testing" section if you want!)

## Configuration

Configure your node as per [[1.0-User-Guide#configuration]], including the section [Enabling CPU Mining](https://github.com/zcash/zcash/wiki/1.0-User-Guide#enabling-cpu-mining).

## Mining

Now, start Mining!
```bash
$ ./src/zcashd
```

To run it in the background (without the node metrics screen that is normally displayed):

```bash
$ ./src/zcashd -daemon
```

You should see the following output in the debug log (`~/.zcash/debug.log`):

```bash
Zcash Miner started
```

Congratulations! You are now mining on the mainnet.

### Spending Mining Rewards

Coins are mined into a t-addr (transparent address), but can only be spent to a z-addr (shielded address). Refer to our [1.0 User Guide](https://github.com/zcash/zcash/wiki/1.0-User-Guide) for instructions on how to use the `z_sendmany` command to send coins from a t-addr to a z-addr. You will need at least 4GB of RAM for this operation.

## Modifications

### Mine to a single address

The internal `zcashd` miner uses a new transparent address for each mined block. If you want to instead use the same address for every mined block, find the following line in both `src/miner.cpp` (in the function `ProcessBlockFound()`) and `src/wallet/wallet.cpp` (in the function `CommitTransaction()`):

```cpp
reservekey.KeepKey();
```

Remove or comment out that line in both places.

### Use P2PKH transactions

The internal `zcashd` miner inherited from Bitcoin uses P2PK for coinbase transactions. The trend in the Bitcoin blockchain has been to use P2PKH instead; we are considering [changing the internal miner to use P2PKH](https://github.com/zcash/zcash/issues/945), but not for the 1.0 release.

If you want to use P2PKH for your coinbase transactions, find the following line in `src/miner.cpp` (in the function `CreateNewBlockWithKey()`):

```cpp
CScript scriptPubKey = CScript() << ToByteVector(pubkey) << OP_CHECKSIG;
```

Change it to:

```cpp
CScript scriptPubKey = CScript() << OP_DUP << OP_HASH160 << ToByteVector(pubkey.GetID()) << OP_EQUALVERIFY << OP_CHECKSIG;
```
