基于 IBC 协议可以实现 cosmos 网络不同链之间的跨链交易，但前提是交易的链都具有及时终止性并且实现了IBC协议。但像 Bitcoin 和 Ethereum 等基于 PoW 的区块链链只具有概率终止性，需要等待一定的区块确认高度才能确保交易的最终一致性，而且也没有实现IBC协议。对于这类区块链如何和 cosmos 区块链实现跨链交易是下面要探讨的。

## peg zone 

既然最终一致性的链需要等待一定区块确认高度才能确保交易的不可回滚，那就可以引入一条适配链来监听该链的交易以及交易被打包之后的确认高度，等到达确认高度安全阈值再由适配链来发给和 cosmos 其他链的跨链交易。这就是 cosmos 链和 Ethereum 跨链的基本思路，适配链就是 peg zone，当然 peg zone 自身也是具有及时终止性和支持 IBC 跨链协议的 cosmos 链。

由于 peg zone 需要和跨链的原生链进行深度耦合，所以不同的链需要专用的 peg zone 来对接。下面就以对接 Ethereum 的 peg zone 为例来阐述如何实现将 Ethereum 的 ERC20 token 通过 peg zone 转移到 cosmos 链，以及 cosmos 链的资产转移到 Ethereum 并成为一种 ERC20 token。

## peggy

对接 Ethereum 的peg zone （简称 peggy）结构如下图所示，主要有5部分组成，分别是 Ethereum smart contract，witness，pegzone，signer 和 relayer。其中 witness，pegzone，signer 和 relayer 是同一个系统的不同模块（概念上区分是为了更清晰的职能和流程设计）。

Ethereum smart contract 是 Ethereum 原生资产的保管者，同时也是 cosmos 原生资产的发行者（转为对应的 ERC20 token）。

witness 是一个 Ethereum 网络中的全节点，可以证实 Ethereum 的状态，并实现在交易确认等待一定区块高度阈值之后来证实状态的确定性。主要的职能是关注 Ethereum 网络中的 Event 并给 pegzone 发送相关消息。

pegzone 是一条适配链，用户可以把资产存储在这条链，也可以通过交易把资产转移给链上的其他用户。当然最重要的是可以和 Ethereum 发送和接收资产；同时也可以和其他链通过 IBC 跨链交易。

signer 负责对消息进行签名转换，从而使 Ethereum smart contract 可以进行验证。这些消息包含了发行到 Ethereum 链上的 token 信息。

relayer 负责把 signer 签名的消息发送到 Ethereum smart contract。

![images](https://github.com/nil-zhang/cosmos-dev-notes/blob/master/cosmos-peg-zone.png)

