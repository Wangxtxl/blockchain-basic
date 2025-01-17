# 比特币是如何交易的

## 如何持有和使用比特币


作为比较，我们先回顾下现实的银行系统：

1. 首先我们需要使用个人信息（如身份证）给银行，银行给我们开户，开户时确立了我们对该账户的所有权。
2. 进行支付的时候，银行对交易双方账户余额进行增减。

但比特币是一个去中心化的系统，没有这样的第三方，它是如何确定某个账户的比特币是属于谁的？这个问题等同于谁可以使用这个账户的比特币？

在比特币的公共共享总帐本中，记录了所有地址持有的余额，比特币的账户是用地址来表示。 




当需要转账时，发送一笔类似的交易（为了理解做了简化）：

```
    "from"："1ABzp1eP5QGefi2DMPTf..."
    "to"："3FRdnTq18LyNveWa1gQJcgp..."
    "amount"："1 btc"
```

当然比特币节点需要这笔交易是谁发起的，只有当由地址 `1ABzp1eP5QGefi2DMPTf...` 发起才是真正有效，因此要求发起人对交易信息进行签名，签名是用地址 `1ABzp1eP5QGefi2DMPTf...` 对应的私钥进行的。

当我们创建一个比特币地址（账户时），会首先生成一个随机数作为私钥，然后根据椭圆曲线算法(ECDSA)计算出私钥，然后在根据哈希运算及校验编码得到比特币地址。

<details>
  <summary>比特币地址格式</summary>
    <div>目前比特币有三种地址类型：
    <br/>
    1. P2PKH 地址，也叫 “传统地址（Legacy address）”，以数字 “1” 开头，长度为 26 个到 36 个字符， 如：`1ABzp1eP5QGefi2DMPTfTL5SLmv7DivfNa`
    <br/>
    2. P2SH 地址，以数字 “3” 开头, 如：`3FRdnTq18LyNveWa1gQJcgp8qEnzijv5vR` .
    <br/>
    3. P2WPKH 地址，也叫 “Bech32 地址”，是一种高级的地址，以 “bc1” 开头.
  </div>
</details>

公钥及地址是公开的，私钥这是保密的，私钥推导地址的过程也是单向的，无法通过地址反推到公钥及私钥。

![](https://img.learnblockchain.cn/pics/20230202124657.webp)

因此当我们持有某地址的私钥，就是持有该地址下的比特币，因此私钥必须妥善保管。


## UTXO 模型

实际上，在比特币账本中，并不是的记录某个账户的余额是多少（和以太坊的账户余额模型不一样），比特币引入了一个“未花费的交易输出”（UTXO： Unspent Transaction Output）概念，一个 UTXO 代表 “一整块” 的可以使用比特币。UTXO 作为交易的输入。

一个 UTXO 在交易时可以产生多个 UTXO ，比特币的交易时不断消费老 UTXO 产生新的 UTXO 的过程，当一个 UTXO 被作为交易的输入后，就不再是未花费的了（STXO），在某个时间点，所有 UTXO 的集合都被称为 UTXO 集。比特币节点会追踪 UTXO 集，从而确定哪些代币未被花费，以及哪些人可以花费它们。从而避免双花（Double Spend）问题。

<details>
  <summary> 思考：最初的 UTXO 从哪里来的呢？</summary>
    <div>最初的 UTXO都来自于区块挖掘奖励，这个称为 coinbase 交易，coinbase 交易可以没有 UTXO 输入，但是像所有正常输出一样，coinbase 交易的输出是新的 UTXO。
  </div>
</details>

UTXO 其实是包含一定数量的比特币（以 “聪（satoshi）” 为单位）以及花费这些比特币时所需满足的条件（叫做 “锁定脚本（locking script）”），当我们要使用一个 UTXO 时，就是对用私钥 UTXO 进行解锁（签名），以便使用其中的比特币。

UTXO 是不可分割的最小的交易单元，如果我们想要花费的比特币数额低于 UTXO 的面值，那该怎么办呢？
我想给小李发送 0.5 BTC，我的 UTXO 面值为 1 BTC，由于必须通过交易花掉一整个UTXO的 BTC，因此我们需要创建另一个UTXO输出作为找零。
好比我们用一张 100 元纸币去买 10 块钱的东西，需要找零 90 块。这就是比特币交易的关键特性。

出于安全性和匿名性的考虑，应该总是使用新比特币地址，来进行找零。

## 交易

![](https://img.learnblockchain.cn/pics/20230202115542.png)


这个是我在区块链浏览器中截下的一个交易（[链接](https://www.blockchain.com/explorer/transactions/btc/0cbc47147ba6e13743a14cfe744e86bbed6f9376e82b3d505e5ced352065327c)），我们可以看到这个交易有一个 UXTO 输入，两个 UXTO 输出。

交易的 “Fee” 显示的数值，其实是 UTXO 总输入和 UTXO总输出之间的差值。

交易结构中没有指明交易费，交易费总是动态计算得出的，在创建交易时，我们要确保输出总是略低于输入，以便让矿工计算交易费是多少。
交易费由矿工收取，用来其补偿保护网络安全，也是其重要的收入来源之一，如果没有交易费，由些矿工会阻止其在网络中广播。


## 钱包

在日常使用过程中，通常是借助钱包软件来完成交易的，因此钱包本身不保存资产，资产是在记录比特币网络账本中的（通常称为保存在链上）。
钱包为了显示你的比特币 “余额”，钱包软件必须在比特币区块链上查询所有由你的私钥控制的 UTXO，然后将这些 UTXO 的值相加，并显示最终余额。

如果想花费 1 BTC，钱包会检查你所有的 UTXO 加起来是否有 1 BTC。如果有的话，钱包就会使用这些 UTXO 作为输入来创建另一笔交易。

钱包实际是一个管理私钥（生成、存储、签名）的工具。

支持比特币的钱包很多，例如：imToken、Trust Wallet、Math Wallet、Ledger(硬件钱包)、Trezor（硬件钱包）等。
我们应该尽量选择**开源**、**知名度大**、大额资产还可以使用硬件钱包。



## 交易过程

 那么一笔交易最终是如何进入区块链的 ？

主要有这么几个阶段：

1. 发起交易及交易签名

2. 节点验证交易有效性

3. 使用工作量证明挖掘新区块



![](https://img.learnblockchain.cn/pics/20230202144210.png)

### 发起交易及交易签名

首先是创建一笔交易， 一笔比特币交易主要由以下元素组成：

1. 交易输入： 他们指向之前的交易创建的 UTXO，通常钱包会收集当前可用的 UTXO 集合作为交易的输入；
2. 交易输出：表明多少比特币会锁定到哪些地址，即生成新的 UTXO 。
3. 签名： 用私钥来解锁交易输入的UTXO。



创建比特币交易是在钱包内完成的，而不是在节点上，因此可以在离线的情况下创建交易，交易创建后，通过比特币节点发送到比特币网络中。 



### 节点验证交易有效性

无论交易来自哪里，交易就得先被节点的交易池（mempool）接受。交易池就是未确认交易的缓存，以便矿工从中挑选出手续费率最高的交易、打包到区块中。

当然矿工有首先检查交易是否有效，例如检查：

- 交易输出是否 小于 0 或者大于 2100 万 BTC 
- 交易不是一笔 coinbase 交易 ，因为区块之外无法存在任何 coinbase 交易。
- 交易的 “重量” 不超过 400000 单位 。这么大体积的交易可能在共识上是有效的，但会占据太多的交易池空间。防止攻击者尝试使用体积非常大但永远不会被挖出的交易来塞爆我们的交易池。
- 签名验证确定 UTXO 的有效性（UTXO 脚本检查）。



所有有效的交易会传播给网络中的所有节点。



### 使用工作量证明挖掘新区块

矿工得到前一个区块的哈希值之后，就可以开始挖下一个区块。创建一个新区块的过程差不多是这样的：

矿工从从交易池中找出最优的一组交易（在一个区块限制下，手续费收益最大），给这组交易创建merkle树， 然后不断的执行暴力哈希运算，以求解出满足一下 Hash 目标值的nonce值：

```
SHA256(SHA256(version + prev_hash + merkle_root + ntime + nbits + nonce )) < HASH 目标值
```

version:  block的版本
prev_hash: 上一个block的hash值
merkle_root:  需要写入的交易记录的hash树的值
ntime:  更新时间
nbits: 当前难度



这里有一个示例图：

![](https://img.learnblockchain.cn/2017/block_structure.jpeg)

挖出的区块由于包含了上一个区块的 Hash，通过这个方式把所有的区块串联起来了，区块挖出之后，会迅速的广播的网络的其他节点，只有当一笔交易被包含进一个带有有效工作量证明的区块，并且该区块被整个网络接受之后，我们就说这笔交易 “被确认了”，此时才可以认为资金的转移已经完成了。



[blockchaindemo](https://blockchaindemo.io/)是模拟区块链的运行的网站，你可以自己动手试一试添加一些区块、添加一些节点，感受区块链的运行。

> 另一个 Blockchain Demo:  https://andersbrownworth.com/blockchain/blockchain 也可以参考。






