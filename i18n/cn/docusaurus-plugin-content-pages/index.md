# Chialisp

Chialisp 是一种功能强大且安全的 LISP 类语言，用于通过智能合约功能来限制和释放资金。该网站是了解 Chialisp、CLVM 和条件语言的综合场所。 

这是一个示例：

```chialisp
(mod (password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
    (list (list CREATE_COIN new_puzhash amount))
    (x)
  )
)
```

<details>
<summary>原文参考</summary>

Chialisp is a powerful and secure LISP-like language for encumbering and releasing funds with smart-contract capabilities.
This website is a consolidated place to learn about Chialisp, CLVM and the conditions language.

Here's a sample:
```chialisp
(mod (password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
    (list (list CREATE_COIN new_puzhash amount))
    (x)
  )
)
```

</details>

## 为什么是 Lisp？

很多人进入我们的 keybase 频道，问我们为什么选择一种 60 岁的语言作为我们的链上编程语言。我们之所以选择它，是因为它的一些独特功能使其非常适合 Chia 区块链：

* **完全沙盒化。** Chialisp 资源利用率完全受控。该语言需要在 50 万台计算机上运行，因此该程序不能以意外的方式影响每个人的机器，这一点很重要。一个 lisp 程序被*评估*，因此不能产生任何新进程或与它运行的系统交互。

* **可组合性。** 一个 lisp 程序本身就是一个列表。此功能允许使用强大的技术，允许您在程序评估期间修改源代码。这样做可以允许“智能硬币”对参与的硬币执行规则，同时仍然允许它利用 Chialisp 必须提供的完整可编程性。使用这样的 lisp 程序可以让您拥有*层智能硬币*，其中“内部”谜语的输出可用于评估“外部”谜语。

* **互操作性。** Chia 生态系统中的每一个智能硬币，无论多么复杂，从根本上来说都是一个被 Chialisp 谜语锁定的硬币。任何谜语的输入将始终是 lisp 数据结构，输出将始终是所有谜语共享的**条件**列表。这意味着 Chia 中的所有内容都可以与其他所有内容互操作。任何智能代币都应该能够与任何其他智能代币进行交互或通信，无论这两种代币是否是专门为此设计的。

<details>
<summary>原文参考</summary>

- ## Why Lisp?

Many people come into our keybase channel and ask us why we chose a 60 year old language as our on chain programming language.
We chose it due to a few unique features that make it remarkably well suited to the Chia blockchain:

* **Completely sandboxed.** Chialisp resource utilization is completely controlled.
The language needs to be run on half a million computers, so it is important that the program cannot reach out and affect everyone's machines in an unintended way.
A lisp program is *evaluated* and therefore cannot spawn any new processes or interact with the system it is running on.

* **Composability.** A lisp program is itself just a list.
This feature allows for powerful techniques that allow you to modify source code during program evaluation.
Doing so can allow a "smart coin" to enforce rules on a participating coin while still allowing it to utilize the full programmability that Chialisp has to offer.
Using lisp programs like this allows you to have *layers of smart coins* in which the output of an "inner" puzzle can be used in the evaluation of the "outer" puzzle.

* **Interoperability.** Every smart coin in the Chia ecosystem, no matter how complex, is fundamentally a coin that is locked up with a Chialisp puzzle. The input to any puzzle will always be a lisp data structure, and the output will always be a list of **conditions** that all puzzles share. This means that everything in Chia interoperates with everything else.
Any smart coin should be able to interact or communicate with any other smart coin, regardless of whether either coin was specifically designed to do so.

</details>

## Chia 资产代币 (CAT)

我们集成到 chia-blockchain 中的第一个 Chialisp 智能交易是 [CATs](https://chialisp.com/docs/puzzles/cats)（以前称为*彩色硬币*）。CAT 允许您在 Chia 区块链上创建完全由您控制的代币。这允许您在 Chia 区块链上发行未经您许可他人无法创建或销毁的资产。这可用于稳定币、股票发行、投票股份或您能想到的任何其他事物。阅读有关 [CAT](https://www.chia.net/2021/09/23/chia-token-standard-naming.en.html) 命名法的更多信息。

<details>
<summary>原文参考</summary>

- ## Chia Asset Tokens (CATs)

The first Chialisp smart transaction that we integrated into chia-blockchain were [CATs](https://chialisp.com/docs/puzzles/cats) (formerly known as *coloured coins*). CATs allow you to create tokens on the Chia blockchain that are entirely controlled by you. This allows you to issue assets on the Chia blockchain that cannot be created or destroyed by others without your permission. This can be used for stable coins, stock issuance, voting shares, or anything else you can think of. Read more about nomenclature of [CATs](https://www.chia.net/2021/09/23/chia-token-standard-naming.en.html).

</details>

## 单例硬币

Chialisp 的另一个引人入胜的应用是创建**单例硬币**。单例硬币是一种可以验证只有一个的硬币。当您可以验证只有一个硬币时，您就可以启用一些有趣的功能。Chia 网络池协议使用它来验证您已将您的地块提交给一个矿池，并且没有将它们承诺给任何其他矿池。您还可以制作 NFT、去中心化身份以及任何其他可以使用独特硬币的东西。

<details>
<summary>原文参考</summary>

- ## Singletons

Another fascinating application of Chialisp is the creation of **singletons**.
Singletons are a type of coin that there is verifiably only one of.
When you can verify that there is only one of a coin, you can enable some interesting functionality.
The Chia Network pooling protocol uses this to verify that you have committed your plots to a pool and have not promised them to any other pool.
You can also make NFTs, decentralized identities, and anything else that could make use of a unique coin.

</details>

## 去中心化金融（DeFi）

Chialisp 还能够使用您今天在其他区块链上找到的任何流行的去中心化金融工具。实现这一点的一个功能是硬币在花费时可以相互通信。您可以让做市商宣布价格，并让其他代币在花费时按照自己的逻辑利用这些价格。Chialisp 提供的自然互操作性也很重要，因为它允许参与者同时分层和利用许多不同的 DeFi 工具！

<details>
<summary>原文参考</summary>

- ## DeFi

Chialisp is also capable of any of the popularly available decentralized finance tools you find on other blockchains today.
One feature that enables this is the fact that coins can communicate with each other when they are spent.
You can have market makers announce prices and have other coins utilize those prices in their own logic when they are spent.
The natural interoperability that Chialisp provides is also relevant because it will allow participants to layer and leverage many different DeFi tools all at once!

</details>

## 介绍材料

- [Chialisp](https://www.chia.net/2019/11/27/chialisp.en.html) 的介绍性帖子
- 介绍我们的[彩色币 MVP](https://www.chia.net/2020/04/29/coloured-coins-launch.en.html)
- [Chia 在去中心化金融领域](https://www.chia.net/2021/07/13/a-vision-for-defi-in-chia.en.html)的愿景

<details>
<summary>原文参考</summary>

- ## Introductory Material

- The introductory post on [Chialisp](https://www.chia.net/2019/11/27/chialisp.en.html)
- Introduction to our [MVP of Coloured coins](https://www.chia.net/2020/04/29/coloured-coins-launch.en.html)
- A Vision for [DeFi in Chia](https://www.chia.net/2021/07/13/a-vision-for-defi-in-chia.en.html)

</details>

## 开发者文档

- [ChiaLisp 编译器存储库](https://github.com/Chia-Network/clvm)
- [Chialisp 开发视频介绍](https://chialisp.com/docs/tutorials/developing_applications)
- [用于在 Chialisp 中开发的 clvm 工具](https://github.com/Chia-Network/clvm_tools)
- [CLVM 基础](/docs/)
- [Chialisp 术语表](/docs/glossary/)
- [低级语言参考文档](/docs/ref/clvm/)

<details>
<summary>原文参考</summary>

- ## Developer Documentation

- [ChiaLisp Compiler Repository](https://github.com/Chia-Network/clvm)
- [A video introduction to developing in Chialisp](https://chialisp.com/docs/tutorials/developing_applications)
- [clvm_tools for developing in Chialisp](https://github.com/Chia-Network/clvm_tools)
- [CLVM Basics](/docs/)
- [Glossary of Chialisp terms](/docs/glossary/)
- [Lower Level Language Reference Document](/docs/ref/clvm/)

</details>
