# Multicoin

## APP基础开发框架

在 cosmos-sdk 定义好的框架下面开发 APP，需要做的基本是下面几步：

1）首先是定义 APP 特有的 message 以及对应的 handler；

2）然后 New 一个 ABCI APP，并为各个 module 创建对应的 KVStore；

3）还需要将 message 和 handler 通过 Router 关联起来，为后续 message 处理铺路；

4）运行APP，完成初始化，加载存储数据，监听各种 message，timer，event，signal；

5）handler 处理 message 完成状态转换，并将结果存入 KVStore；

6）message 需要包装在 transaction 里面，transaction 需要有对应的 Decoder。

    以上这些 basecoin 基本完成，multicoin 就是在 basecoin 的基础上继续开发的。

## 编解码

问题：transaction 里面可以包含多种 message 类型，需要先 decode 为 interface，然后再根据 message 类型来分开处理。而在 golang 中没有内置的将一个 bytes 数组 decode 为一个 interface 类型的方法。

解法：cosmos/Tendermint 库中引入了 amino 编解码，就是为了解决上面的问题。需要先将具体的 struct register 为一个全局唯一的名称，然后在编码的时候携带该名称，这样解码的时候就知道类型。

    在 APP 中引入新的模块时，需要同时调用对应的 RegisterWire 函数。
    
    multicoin 引入 stake 和 gov 是就调用了：stake.RegisterWire(cdc) 和 gov.RegisterWire(cdc)。

## 读写权限控制

为了确保数据存储的安全性，cosmos-sdk 使用最小权限原则对 KVStore 的访问进行了两层抽象。

首先用 Mapper 封装 KVStore，默认情况下模块只能访问自己的 Store，主要是通过 key 来控制的；

然后用 Keeper 又一次封装了 Mapper，主要是区分读写权限，只需要读权限的，就传递个Read Keeper。

    multicoin 中引入的 stake 和 gov 模块 都是基于 Keeper 的访问。

## 自定义 AppInit 阶段

Genesis transaction 和 state 是一条链的第一个共识。PS：Genesis 配置不同就不会加入同一个网络。：）

自定义 AppInit 阶段的好处是可以个性化定制创世参数。（GodMode）

主要的步骤如下，其中 AppGenTx() 和 AppGenState() 是可以自定义的：

1、传入 moniker，生成 nodeKey，创建 validator；

2、调用自定义的 AppInit.AppGenTx() 来完成：

1）生成 privKey 和 secret phrase；

2）初始化 Genesis transaction。

3、调用自定义的 AppInit.AppGenState() 来完成：

1）创建 Genesis Account 并配置一定的 Token 和 coin；
2）初始化 stake 模块的 GenesisState；
3）New validator 并添加 Token，同时为stakeData pool 添加 Token。

4、生成 genesis.json, config.toml, 写入 配置文件。

    multicoin 在 自定义 AppInit 中为每个 Account 初始化了 Token，并为每个 validator 预支了 steak 用来做 PoS。

## Notes
1、使用标准的 x/auth 实现 Account 以及 签名验证（首次签名的 Account 要带上pubKey），x/bank 实现 代币的转移；

2、InitChainer 在 APP 首次启动时会被 Tendermint 调用一次，用来初始化 coinbase Account；

3、anteHandler 是全局的函数，在handler 之前执行，主要是为了验证交易和Fee。


