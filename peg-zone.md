基于 IBC 协议可以实现 cosmos 网络不同链之间的跨链交易，但前提是交易的链都具有及时终止性并且实现了IBC协议。但像 Bitcoin 和 Ethereum 等基于 PoW 的区块链链只具有概率终止性，需要等待一定的区块确认高度才能确保交易的最终一致性，而且也没有实现IBC协议。对于这类区块链如何和 cosmos 区块链实现跨链交易是下面要探讨的。

# peg zone 

既然最终一致性的链需要等待一定区块确认高度才能确保交易的不可回滚，那就可以引入一条适配链来监听该链的交易以及交易被打包之后的确认高度，等到达确认高度安全阈值再由适配链来发给和 cosmos 其他链的跨链交易。这就是 cosmos 链和 Ethereum 跨链的基本思路，适配链就是 peg zone，当然 peg zone 自身也是具有及时终止性和支持 IBC 跨链协议的 cosmos 链。

由于 peg zone 需要和跨链的原生链进行深度耦合，所以不同的链需要专用的 peg zone 来对接。下面就以对接 Ethereum 的 peg zone 为例来阐述如何实现将 Ethereum 的 ERC20 token 通过 peg zone 转移到 cosmos 链，以及 cosmos 链的资产转移到 Ethereum 并成为一种 ERC20 token。

# peggy

对接 Ethereum 的peg zone （简称 peggy）结构如下图所示，主要有5部分组成，分别是 Ethereum smart contract，Witness，PegZone，Signer 和 Relayer。其中 Witness，PegZone，Signer 和 Relayer 是同一个系统的不同模块（概念上区分是为了更清晰的职能和流程设计）。

Ethereum smart contract 是 Ethereum 原生资产的保管者，同时也是 cosmos 原生资产的发行者（转为对应的 ERC20 token）。

Witness 是一个 Ethereum 网络中的全节点，可以证实 Ethereum 的状态，并实现在交易确认等待一定区块高度阈值之后来证实状态的确定性。主要的职能是关注 Ethereum 网络中的 Event 并给 PegZone 发送相关消息。

PegZone 是一条适配链，用户可以把资产存储在这条链，也可以通过交易把资产转移给链上的其他用户。当然最重要的是可以和 Ethereum 发送和接收资产；同时也可以和其他链通过 IBC 跨链交易。

Signer 负责对消息进行签名转换，从而使 Ethereum smart contract 可以进行验证。这些消息包含了发行到 Ethereum 链上的 token 信息。

Relayer 负责把 signer 签名的消息发送到 Ethereum smart contract。

![images](https://github.com/nil-zhang/cosmos-dev-notes/blob/master/cosmos-peg-zone.png)

# 主流程分析

## Ethereum 原生 token 转移

1、Alice 在 Ethereum 上发起一个交易，将自己 Ethereum 账户的 10Ether 转移给自己在 cosmos peg zone 的账户；这个交易会首先发送给 Ethereum smart contract；

2、Witness （Ethereum 全节点）可以监听到 Ethereum smart contract 上的交易，并会在等待一个 Ethereum 区块高度阈值（确定终止性）之后，向 PegZone 提交一个 WitnessTx 交易；

3、PegZone 收到 WitnessTx 之后，会向 Alice 在 PegZone 的账户发放和 10Ether 等值的 token；这样 Alice 在 PegZone 账户就会收到 10CEther；

4、Alice 在 PegZone 可以将 4 CEther 发送给 Bob 在 PegZone 的账户；

5、Bob 可以通过向 Ethereum smart contract 提交 RedeemTx 来将收到的 4 CEther 转回自己在 Ethereum 的账户；

6、Signer 可以监听到 RedeemTx 并为之生成 Ethereum smart contract 可以验证的签名，然后通过 SignTx 将 签名发送给 PegZone；

7、Relayer 监听 SignTx，在大于 2/3 的 validators （基于 voting power）已经提交 SignTx 之后，relayer 会将 SignTx 和原生的消息一起发送给 Ethereum smart contract。

## Cosmos 原生 token 转移

1、Alice 在 cosmos 链上的账户拥有 10 Photons，可以通过 RedeemTx 将 10 Photons 全部发送给自己在 PegZone 的账户；

2、Signer 监听到 RedeemTx 交易并为之生成 Ethereum smart contract 可以验证的签名，然后通过 SignTx 将 签名发送给 PegZone；

3、Relayer 监听 SignTx，在大于 2/3 的 validators （基于 voting power）已经提交 SignTx 之后，relayer 会调用 Ethereum smart contract 的函数来触发发行 ERC20 token；

4、Ethereum smart contract 首先会验证数据是被大于 2/3 的validators 签名的，然后为 Photons 生成 ERC20 合约，再会向 Alice 控制的地址发送 10 EPhotons token；

5、Alice 可以在 Ethereum 上将 4 EPhotons 发送给 Bob；

6、Bob 可以发起交易，将 4 EPhotons 发送给自己在 PegZone 的地址，并将交易发送给 Ethereum smart contract；smart contract 会燃烧这些 token 并生成一个 event；

7、Witness 证实 4 EPhotons 确实被燃烧并向 PegZone 发送 WitnessTx；

8、PegZone 收到 WitnessTx 并给 Bob 的账户发放 4 Photons。



