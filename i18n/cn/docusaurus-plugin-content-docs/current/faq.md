---
id: faq
title: ChiaLisp 和 CLVM 常见问答
sidebar_label: ChiaLisp 和 CLVM 常见问答
---

> ChiaLisp and CLVM FAQ

**问：** 为什么我的号码被评估为 `()`，又名 `nil`？

**答:** 在 clvm（`brun` 命令）中，整数被评估为参数树中参数的引用。如果命令行上没有给出参数树，则默认为空参数树。如果没有找到参数，则返回 `nil`。在 ChiaLisp（`run` 命令）中，整数被编译为引用的原子，这会给你你期望的值。
____

**问：** 是否可以在智能币中存储数据或维护状态？

**答:** 是的，但可能不是您的想法。ChiaLisp 环境的设计非常有意，以便状态仅存储在硬币中。请记住，Chia 使用智能代币，而不是智能合约。这导致了与智能合约不同的设计。Chia 智能硬币的一个常见设计模式是，它们将使用相同的谜题重新创建自己，但更改了一些“状态”。
___

**问：** ChiaLisp、CLVM 字节码、CLVM 程序集和条件语言有什么区别？

**答:** ChiaLisp 是高级语言，可以编译成称为 CLVM 的低级语言。

CLVM 汇编是 ChiaLisp 编译成的低级语言。

CLVM 字节码是 CLVM 程序集的序列化形式。

当 CLVM 在网络上运行时，它可以使用一种称为条件语言的语言来声明满足某些要求。条件语言是一系列同时被评估的语句。
____

**问：** 什么是 CAT？

**答:** CAT 代表 Chia 资产代币（Chia Asset Token）。

CAT 是同质化的代币，由 XCH 铸造并存在于 Chia 的区块链上。CAT 具有被“标记”的特性，使它们无法作为常规 XCH 使用。然而，通常有可能“退役” CAT，然后它们“熔炼”回 XCH。
____

**问：** 什么是 TAIL？

**答:** TAIL 代表代币和资产发行限制器。

TAIL 是 Chialisp 程序，用于验证是否遵守了 CAT 的所有供应规则。共享相同 TAIL 的两个 CAT 属于相同类型。 TAIL 定义了 CAT。

有关 CAT 和 TAILS 的更多信息，请查看我们的 [CAT1 标准](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation")文档。
____

**问：** 我怎样才能收到一些代币？

**答:** 如果您将您的 XCH 钱包地址提供给某人，他们可以向您发送代币，就像他们向您发送 XCH 一样。如果您收到的代币在我们的验证列表中，它们将自动显示在您的轻钱包中（完整的钱包功能将在未来版本中推出）。

如果您的新代币不在我们的验证列表中，您需要手动添加钱包。首先从向您发送令牌的人那里获取 CAT 的 ID。在您的轻钱包左上角，点击“+ ADD TOKEN”，然后点击“+ Custom”。在名称字段中输入您的 CAT 的名称。对于令牌和资产发行限制字段，添加 CAT 的 ID。单击添加。

你应该被带到你的 CAT 的钱包里。如果您已经收到代币，它们将显示在此钱包中。
____

**问：** 我可以制作自己的 CAT 吗？

**答：** 当然！我们有教程来指导您在 [Windows](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Windows "Chia Asset Token tutorial for Windows users") 和 [Linux/MacOS](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Linux_MacOS "Chia Asset Token tutorial for Linux and MacOs users")上完成 CAT 创建过程。
____

**问：** 如何验证我的 CAT？

**答:** 最终我们会发布一个正式的流程供您验证您的 CAT，之后它将在我们的钱包 GUI 中列为默认 CAT 之一。请记住，这将是一个严格控制的过程，很少有 CAT 被验证。但是不要让这阻止您创建自己的 CAT——它们仍然可以工作，即使它们没有列在我们的 GUI 中。

**问：** 什么样的 CAT 正在开发中？

**答:** 现在给出任何具体细节还为时过早，但许多不同的 CAT 即将*很快*！

预计验证过程将在 2022 年 1 月左右发布。


<details>
<summary>原文参考</summary>

**Q:** Why is my number being evaluated to `()`, a.k.a. `nil`?

**A:** In clvm (the `brun` command), integers are evaluated as references to arguments in the argument tree.
If no argument tree is given on the command line, the default is an empty argument tree. When an argument is not found, `nil` is returned.
In ChiaLisp (the `run` command), integers are compiled to quoted atoms, which will give you the value you expected.
____

**Q:** Is it possible to store data or maintain state in smart coins?

**A:** Yes, but probably not how you are thinking.
Quite deliberately the ChiaLisp environment is designed so that state is stored exclusively in coins.
Remember Chia uses smart coins, not smart contracts. This leads to a different kind of design to smart contracts.
A common design pattern in Chia smart coins is that they will recreate themselves with the same puzzle but with some "state" changed.
___

**Q:** What is the difference between ChiaLisp, CLVM bytecode, CLVM assembly and the Conditions Language?

**A:** ChiaLisp is the higher level language which can be compiled into the lower level language called CLVM.

CLVM Assembly is the lower level language that ChiaLisp is compiled to.

CLVM Bytecode is the serialized form of CLVM Assembly.

When CLVM is run on the network, it can use a language called the Conditions Language to declare certain requirements be met.
The conditions language is a series of statements which are evaluated all at the same time.
____

**Q:** What is a CAT?

**A:** CAT stands for Chia Asset Token.

CATs are fungible tokens that are minted from XCH and live on Chia's blockchain. CATs have the property of being "marked" in a way that makes them unusable as regular XCH. However, it is often possible to "retire" CATs, which then "melt" back into XCH.
____

**Q:** What is a TAIL?

**A:** TAIL stands for Token and Asset Issuance Limiter.

A TAIL is a Chialisp program that verifies that all of a CAT's supply rules are being followed. Two CATs that share the same TAIL are of the same type. The TAIL defines the CAT.

For more information on CATs and TAILS, check out our [CAT1 standard](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation") documentation.
____

**Q:** How can I receive some tokens?

**A:** If you give someone your XCH wallet address, they can send you tokens, just like they would send you XCH. If the tokens you receive are on our verified list, they'll automatically show up in your light wallet (full wallet functionality is coming in a future release).

If your new tokens are not on our verified list, you'll need to add a wallet manually. First obtain the CAT's ID from whoever sent you the tokens. In the upper left corner of your light wallet, click "+ ADD TOKEN", then click "+ Custom". Enter the name of your CAT in the Name field. For the Token and Asset Issuance Limitations field, add the CAT's ID. Click ADD.

You should be taken to a wallet for your CAT. If you have already received tokens, they'll show up in this wallet.
____

**Q:** Can I make my own CAT?

**A:** Sure! We have tutorial to guide you through the CAT creation process on both [Windows](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Windows "Chia Asset Token tutorial for Windows users") and [Linux/MacOS](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Linux_MacOS "Chia Asset Token tutorial for Linux and MacOs users").
____

**Q:** How can I get my CAT verified?

**A:** Eventually we will release a formal process for you to verify your CAT, after which it will be listed as one of the default CATs in our wallet GUI. Keep in mind that this will be a tightly-controlled process, where few CATs be verified. But don't let that stop you from creating your own CATs -- they will still work, even if they are not listed in our GUI.

Expect the verification process to be published around January 2022.


**Q:** What sort of CATs are in development?

**A:** It's too early to give any specific details, but many different CATs are coming _soon_!

</details>

