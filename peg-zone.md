
基于 IBC 协议可以实现不同链之间的跨链交易，但前提是交易的链都具有及时终止性并且实现了IBC协议。但像 Bitcoin 和 Ethereum 等基于 PoW 的区块链链只具有概率终止性，需要等待一定的区块确认高度才能确保交易的最终一致性，而且也没有实现IBC协议。对于这类区块链如何和 cosmos 区块链实现跨链交易是下面要探讨的。



![images](https://github.com/nil-zhang/cosmos-dev-notes/blob/master/cosmos-peg-zone.png)

