# Substrate-taproot
## Introduction
Borrowing from BTC's taproot implementation, taproot consists of two parts (aggregate signature + MAST contract). 
Taproot can realize address privacy and contract privacy. We combine taproot's theory to achieve transaction privacy under the Substrate framework. 
We need to implement two pieces of content, 1. Customize the Substrate Mast contract. 2. Combined with merkle tree to realize threshold signature.
There is a relative difference between Substrate and BTC's implementation of taproot, which is the account model. BTC is a UTXO account system, and Substrate is an object-oriented account system. In the process of implementation, we need to asynchronousize the logical format of the UTXO system to the object account system. Similar to the translation of functional languages into object-oriented languages.

## Customized Substrate MAST contract
      cript = [Account List] + [Call] + [Values] + [Time Lock]
- Account List: It is composed of N accounts, N >= 1 and a positive integer.
- Call: Call interface that can be called on all pallets on the Substrate chain.
- Values: The corresponding setting value of the corresponding Call.
- Time Lock: (lower,uper) represents the tupper structure, lower and upper are in blockheight as the unit, lower represents the lower limit, uper represents the upper limit, and 0 represents no upper limit, or lower limit.

## Threshold signature pallet
Through the musig interface of sr25519, the aggregation signature is realized under the chain, and combined with the merkle tree to realize the threshold signature.

## The whole business logic
- Generate threshold signature address. generate_address() -> Address.
- Multi-signature organizations submit script hashes for review. pass_script(script hash) -> bool.
- Authorize the account to upload the script to perform its own privacy operations. execute_script(script) -> bool.

## Suitable for business scenarios:
Address privacy, contract privacy, such as decentralized contract market value management, or privacy oracle.

## Business scenario 1:
A project party A, A has M investors, the project party has N board members, and the N board members of the project party A need to sign multiple off-chain (in order to ensure the privacy of the board members of the project party) to make M investments People release tokens in accordance with the contract, But it is necessary to ensure the privacy of contracts for M investors, because all these data are disclosed on the chain, which will cause the market-making team to master the laws and go short and long, and control to disrupt the market.

In the above, both trust and privacy must be guaranteed. 

Let's give a well-understood number: For example, N=3, M=10.

Step 1: The three board members use the off-chain threshold signature to call generate_address() to generate the threshold address. The signature address of the board member under this address is private.

Step 2: Fix the Mast contract under the chain, which is the Script script. Script = Account: ** (one of M) + Call: transfer + Values: Amount (the number of tokens occupied by the investor) + Time Lock: (lower, 0) Start execution when the lower block is high.

Step 3: Board members off-chain call musig to aggregate signature voting and upload each privacy contract script hash, pass_script(script hash). And the board members passed the privacy scripts to M investors one by one through ComingChat's private communication channel.

Step 4: After the investment reaches a fixed block height, the operation publishes its own script and calls the execute_script(script) contract.

Thus, the privacy of the contract is realized within a certain period of time. It avoids malicious market makers in the market from maliciously controlling the price of the currency, and also avoids the trust problem of the centralized issuance of tokens between investors and project parties.
    
    
## 简介
   借鉴于BTC的taproot实现， taproot 由两部分组成（聚合签名 + MAST合约）。 
   taproot可以实现 地址隐私 和合约的隐私。 我们结合taproot的理论在Substrate框架下实现交易的隐私。
   我们需要实现 两块内容， 1， 定制Substrate Mast合约。 2， 结合merkle树实现门限签名。
   Substrate 和 BTC 实现 taproot 有个相对的差异，那就是账户模型。
   BTC是 UTXO账户体系， Substrate是 面向对象账户体系。 实现的过程中，我们需要把UTXO体系的逻辑格式异步化对象账户体系。 类似于函数式语言翻译成面向对象式的语言。
   
 ## 定制Substrate MAST合约
    Script = [Account List] + [Call] + [Values] + [Time Lock]
 - Account List: N 个账户组成， N >= 1 且为 正整数。
 - Call： Substrate链上所有pallet 上可以调用的Call接口。
 - Values：相应Call 对应的设置值。
 - Time Lock:  (lower，uper)表示的tupper结构， lower，uper 以blockheight 为单位， lower 代表下限， uper代表上限， 为0 代表无上限，或者下限。
 
## 门限签名pallet
    通过 sr25519 的 musig 接口 链下实现聚合签名， 再结合merkle树实现门限签名。

## 整个业务逻辑
- 1, 生成门限签名地址。generate_address() -> Address.
- 2, 多签组织提交审核Script 脚本hash。 pass_script(script hash) -> bool.
- 3, 授权账户上传脚本执行属于自己的隐私操作。 execute_script(script) -> bool.

## 适合业务场景：
   地址隐私，合约隐私， 比如 去中心合约市值管理，或者隐私预言机。

## 业务场景1
    某项目方A， A的投资人有M个, 项目方有N个董事会成员， 项目方A中的N个董事会成员要通过链下多签（为了保证项目方董事会成员的隐私）给M个投资人按照合约释放token，
    但又要保证给M个投资人的合约隐私， 因为这些数据全公开到链上， 会导致做市团队掌握规律做空做多，控制扰乱市场。
    在上面这个既要保证 信任，又要保证隐私。
    我们举个很好理解的数目：比如N=3， M=10.
    第1步： 3个董事会成员，通过链下门限签名 调用generate_address() 生成 门限地址， 该地址下的董事会成员签名地址隐私。
    第2步： 固定好链下Mast 合约，也就是Script脚本。 Script = Account: **（ M 中的一个） + Call:transfer + Values: Amount(投资人所占的token数量) + Time Lock: (lower, 0) 从为lower块高时开始执行。
    第3步： 董事会成员 链下 调用musig 聚合签名投票 上传 各个隐私合约脚本hash， pass_script(script hash)。并且董事会成员通过ComingChat隐私通讯渠道把隐私脚本逐个传递给M个投资人。
    第4步： 投资到了固定块高后， 操作公布自己的脚本，调用execute_script(script) 合约。
    从而在一定时间内实现了合约的隐私。 避免了 市场上的恶意做市商 恶意控制币价，也避免了投资人和项目方之间代币中心化发放的信任问题。
    
    
     
