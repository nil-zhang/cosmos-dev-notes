
基于 IBC 协议可以实现 cosmos 网络不同链之间的跨链交易，但前提是交易的链都具有及时终止性并且实现了IBC协议。但像 Bitcoin 和 Ethereum 等基于 PoW 的区块链链只具有概率终止性，需要等待一定的区块确认高度才能确保交易的最终一致性，而且也没有实现IBC协议。对于这类区块链如何和 cosmos 区块链实现跨链交易是下面要探讨的。

## peg zone 

既然最终一致性的链需要等待一定区块确认高度才能确保交易的不可回滚，那就可以引入一条适配链来监听该链的交易以及交易被打包之后的确认高度，等到达确认高度安全阈值再由适配链来发给和 cosmos 其他链的跨链交易。这就是 cosmos 链和 Ethereum 跨链的基本思路，适配链就是 peg zone，当然 peg zone 自身也是具有及时终止性和支持 IBC 跨链协议的 cosmos 链。

![images](https://github.com/nil-zhang/cosmos-dev-notes/blob/master/cosmos-peg-zone.png)

