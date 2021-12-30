---
id: cats
title: Chia 资产代币（Cats）
---

> Chia Asset Tokens (CATs)

目录：

* [CAT 简介](#cat-简介)
* [设计选择](#设计选择)
* [支出账户](#支出账户)
* [额外的 Delta](#额外的-delta
* [代币和资产发行限制器（TAIL）计划](#代币和资产发行限制器tail计划)
* [TAIL 力量的极限](#tail-力量的极限)
* [TAIL 示例](#tail-示例)
* [CAT 面额、价值和退休规则](#cat-面额价值和退休规则)
* [结论](#结论)

-----

## CAT 简介

**Chia 资产代币 (CAT)** 是由 XCH 发行的可替代代币。
CAT1 标准是第一个（也是迄今为止唯一的）CAT 标准。 CAT1 是截至 2021 年 11 月 16 日的标准草案。经过 Chia 社区的评论期后，CAT1 将最终确定。这可能包括附加功能，并可能导致对现有 CAT 的一些重大更改。有关本文档中使用的命名约定的更多信息，请参见 [此处](https://www.chia.net/2021/09/23/chia-token-standard-naming.en.html "Blog entry explaining CAT1 naming conventions")。

>**提醒：**
>
>>**同质代币**可以拆分或合并在一起。
>>它们也可以代替等值的令牌。
>>一些常见的例子包括黄金、石油和美元。
>
>>**不可替代的代币**，另一方面，是不可分割的，不能合并。
>>它们是独一无二的，因此无法替代。
>>一些常见的例子包括汽车、棒球卡和衣帽间门票。

CAT 具有被“标记”的特性，使它们无法作为常规 XCH 使用。但是，通常可以稍后将 CAT“融化”回 XCH。 CAT 通常用作积分或代币——有点像赌场筹码。

**所有 CATs** 共享的 chialisp 代码是 [here](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/cat.clvm "cat.clvm - the source code that all CATs share")。如果不遵循这种谜语格式，钱包将无法将代币识别为 CAT。

上面链接的代码的全部目的是确保特定 CAT 的供应永远不会改变，除非遵循一组特定的“发行规则”。每个 CAT 都有自己独特的发行规则，**这是不同类型 CAT 之间的唯一区别**。这些发行规则采用遵循特定结构的任意 Chialisp 程序的形式。我们称该程序为**代币和资产发行限制（TAIL）**。

CAT 层是一个[外部谜语](https://chialisp.com/docs/common_functions#outer-and-inner-puzzles "Chialisp documentation for how to create outer and inner puzzles")，其中包含两个柯里化参数：
1. 内部谜语，控制 CAT 的所有权。
2. TAIL 的谜语哈希，它定义了 CAT 的三个方面：
   * CAT 的类型。（具有相同 TAIL 的两只 CAT 属于相同类型，即使它们包含不同的内部谜语。）
   * CAT 的发布规则。
   * CAT 的熔断规则。

不同 TAIL 可以满足的发行要求的一些示例包括：
  * 稳定币——创建者将希望铸造新的代币，因为他们获得资金支持。
  * 供应有限的代币——创建者希望只发行一次，并保证不会再铸造出更多的相同类型的代币。
  * 资产赎回代币——创建者将希望允许 CAT 的所有者将代币融化成标准的 XCH，只要他们遵循某些规则。

在所有这些情况下，当花费硬币时运行 TAIL 程序以检查发行是否有效。

稍后我们将更详细地介绍 TAIL 程序，但首先让我们介绍 CAT 层的作用。

<details>
<summary>原文参考</summary>

- ## Introduction to CATs

**Chia Asset Tokens (CATs)** are fungible tokens that are issued from XCH.
The CAT1 Standard is the first (and so far only) CAT Standard. CAT1 is a draft standard as of Nov 16, 2021. After a comment period from the Chia community, CAT1 will be finalized. This may include additional capabilities, and could result in some breaking changes to existing CATs. More information on the naming conventions used in this document can be found [here](https://www.chia.net/2021/09/23/chia-token-standard-naming.en.html "Blog entry explaining CAT1 naming conventions").

>**Reminder:**
>
>>**Fungible tokens** can be split apart, or merged together.
>>They can also be substituted for a token of equal value.
>>Some common examples include gold, oil, and dollars.
>    
>>**Non-fungible tokens**, on the other hand, are indivisible and cannot be merged.
>>They are unique, so they cannot be substituted.
>>Some common examples include cars, baseball cards, and cloakroom tickets.

CATs have the property of being "marked" in a way that makes them unusable as regular XCH. However, it is usually possible to "melt" CATs back into XCH later. CATs are often used as credits, or tokens - kind of like casino chips.

The chialisp code that **all CATs** share is [here](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/cat.clvm "cat.clvm - the source code that all CATs share"). Without following this puzzle format, wallets will not be able to recognize a token as a CAT.

The entire purpose of the code linked above is to ensure that the supply of a specific CAT never changes unless a specific set of “rules of issuance” is followed. Each CAT has its own unique rules of issuance, **which is the only distinction between different types of CATs**. These issuance rules take the form of an arbitrary Chialisp program that follows a specific structure.  We call that program the **Token and Asset Issuance Limitations (TAIL)**.

The CAT layer is an [outer puzzle](https://chialisp.com/docs/common_functions#outer-and-inner-puzzles "Chialisp documentation for how to create outer and inner puzzles"), which contains two curried parameters:
1. An inner puzzle, which controls the CAT's ownership.
2. The puzzlehash of a TAIL, which defines three aspects of a CAT:
   * The CAT's type. (Two CATs with the same TAIL are of the same type, even if they contain different inner puzzles.)
   * The CAT's issuance rules.
   * The CAT's melting rules.

Some examples of issuance requirements that different TAILs could accommodate include:
  * Stablecoins - The creator will want to mint new tokens as they gain funds to back them.
  * Limited supply tokens - The creator will want to run a single issuance, with the guarantee that no more tokens of the same type can ever be minted.
  * Asset redemption tokens - The creator will want to allow the CAT's owners to melt the tokens into standard XCH, as long as they follow certain rules.

In all of these cases, the TAIL program is run when a coin is spent to check if the issuance is valid.

We will cover the TAIL program in more detail later, but first let's cover what the CAT layer does.

</details>

## 设计选择

* **当花费一个 CAT 时，任何创建的硬币都会自动变成具有相同 TAIL 的 CAT**
  
  当内部谜语返回 CREATE_COIN 条件时，CAT 层将识别出这一点并将条件更改为与自身类型相同的 CAT。
  
  例如，假设内部谜语返回以下 CREATE_COIN 条件：`（51 0xcafef00d 数量）`

  在这种情况下，CAT 层将为与自身具有相同 TAIL 的 CAT 以及内部谜语 `0xcafef00d` 计算谜语哈希。

* **如果 CAT 不使用 TAIL，则 CAT 的花费组合不得获得或失去任何价值**

  为了确保 CAT 在没有官方认证的情况下不能被铸造或退役，所有不使用 TAIL 程序的 CAT **必须**是输出与其输入相同数量价值的花费组合的一部分。
我们使用组会计技巧来保证这一点，我们将在下面更详细地介绍。

* **如果 CAT 未被 TAIL 程序批准，则其父级必须是同一类型的 CAT**

  我们防止以未经批准的方法铸造 CAT 的另一种方法是确保令牌具有有效的谱系。通常，这是通过断言 CAT 的父级也是同一类型的 CAT 来完成的。

  这是通过传入硬币的信息，返回 `ASSERT_MY_ID` 条件，然后传入父信息来完成的。

* **CAT 强制执行硬币公告的前缀**

  为了确保 CAT 可以在不受内部谜题干扰的情况下相互通信，他们必须预先发布适当的硬币公告，并遵循以下规则：
  * 如果公告来自 CAT 层，则在其前面加上 `0xcb`。
  * 如果公告来自内部谜语，则以`0xca` 开头。

* **CAT 将预先计算好的真相列表传递给内部拼图**

  许多内部谜语需要诸如硬币 ID 和谜语哈希等信息。幸运的是，我们已经在 CAT 层中计算了大部分信息，因此我们将其组合在一起作为 **Truths** 的预先验证集合。然后我们将这些真理作为谜底的第一个参数传递到内部谜语中。

  真相是：
  * 我的 ID —— 硬币的 ID
  * 我父代的 ID —— 硬币父代的 ID
  * 我的完整谜语哈希 —— 硬币内包含的谜语哈希
  * 我的金额 —— 硬币的价值
  * 我的内部谜语哈希 —— 此硬币的 CAT 层内的内部谜语的谜语哈希
  * 我的血统证明 —— （可选）证明 CAT 的父代与 CAT 属于同一类型
  * 我的 TAIL 谜底 ——（可选）传入 TAIL 程序的参数列表
  * 我的硬币信息
  * CAT Mod 哈希 —— 在任何东西被咖喱之前的 CAT 哈希
  * CAT Mod 哈希的哈希 —— CAT Mod 哈希的哈希
  * CAT TAIL 程序哈希 —— CAT 中的 TAIL 程序的哈希值

<details>
<summary>原文参考</summary>

- ## Design choices

* **When a CAT is spent, any coins created automatically become CATs with the same TAIL**
  
  When an inner puzzle returns a CREATE_COIN condition, the CAT layer will recognize this and change the condition to a CAT of the same type as itself.
  
  For example, let's say the inner puzzle returns the following CREATE_COIN condition:
`(51 0xcafef00d amount)`

  In this case, the CAT layer will calculate a puzzlehash for a CAT with the same TAIL as itself, and an inner puzzle of `0xcafef00d`.

* **If a CAT does not use a TAIL, then a SpendBundle of CATs must not gain or lose any value**

  In order to ensure that a CAT cannot be minted or retired without official authentication, all CATs that do not use a TAIL program **MUST** be a part of a spendbundle that outputs the same amount of value as its input.
We use a group accounting trick to guarantee this, which we will cover in more detail below.

* **If a CAT is not approved by a TAIL program, then its parent must be a CAT of the same type**

  Another way we prevent CATs from being minted in unapproved methods is by ensuring that the tokens have a valid lineage. Commonly this is done by asserting that the CAT's parent was also a CAT of the same type.

  This is accomplished by passing in the coin's information, returning an `ASSERT_MY_ID` condition, and then passing in the parent information.

* **CATs enforce prefixes for Coin Announcements**

  In order to ensure that the CATs can communicate with each other without interference from an inner puzzle, they must prepend an appropriate coin announcement with the following rules:
  * If the announcement comes from the CAT layer, it is prepended with `0xcb`.
  * If the announcement comes from the inner puzzle, it is prepended with `0xca`.

* **CATs pass a list of pre-calculated Truths to the inner puzzle**

  Many inner puzzles require information such as their coin ID and puzzlehash. Luckily, we already calculate much of this information in the CAT layer, so we bundle it together as a pre-validated collection of **Truths**. We then pass these Truths into the inner puzzle as the first parameter in the solution.

  The Truths are:
  * My ID - The ID of the coin
  * My Parent's ID - The ID of the coin's parent
  * My Full Puzzle Hash - The puzzlehash contained inside the coin
  * My Amount - The value of the coin
  * My Inner Puzzle Hash - The puzzlehash of the inner puzzle inside this coin's CAT layer
  * My Lineage Proof - (optional) Proof that the CAT's parent is of the same type as the CAT
  * My TAIL Solution - (optional) A list of parameters passed into the TAIL program
  * My Coin Info
  * CAT Mod Hash - The hash of the CAT before anything is curried
  * CAT Mod Hash Hash - The hash of the CAT Mod Hash
  * CAT TAIL Program Hash - The hash of the TAIL program that was curried into the CAT

</details>

## 支出账户

每个 CAT 硬币都对其数量与其输出数量之间的差值进行真实计算，称为 **Delta**。

CAT 硬币也被赋予了所有其他硬币的 Delta 的总和，该总和必须为零。 为了执行这个零增量规则，每个 CAT 硬币都被分配了一个下一个和上一个邻居，最终形成一个环。

硬币使用硬币公告与它们的邻居进行通信。 对于我们的用例，公告必须包含两条信息：
   * 创建此币的币的 ID。 这已经隐含在硬币公告中。
   * 预期收件人的硬币 ID。 公告的消息必须包含此信息，以防止代币收到不适合他们的消息的攻击。

为了形成环，每个硬币输出以下条件：

```
(
  (CREATE_COIN_ANNOUNCEMENT previous_coin_ID + sum_of_deltas_before_me)
  (ASSERT_COIN_ANNOUNCEMENT sha256(next_coin_id + my_coin_id + sum_of_deltas_including_me))
)
```

其中`+`表示字节的连接，并且公告断言采用公告创建者的 ID 和消息的哈希。

为了创建 `next_coin_id`，我们用当前硬币的 CAT 信息包装下一个硬币的内部谜语。 这保证 next_coin_id 是与当前硬币类型相同的 CAT。

因为两种硬币都遵循相同的 CAT 模块代码，所以它们必须遵守相同的一组真理。 这反过来又保证了整个环都是在说实话。 只要环是连通的，总的 Delta 就必须为零。

<details>
<summary>原文参考</summary>

- ## Spend Accounting

Each CAT coin has a truthful calculation of the difference between its amount and its output amount, called its **Delta**.

The CAT coins also are given the sum of every other coin's Deltas, which must be zero. In order to enforce this zero-Delta rule, each CAT coin is assigned a Next and Previous neighbor, which ultimately forms a ring.

The coins use Coin Announcements to communicate with their neighbors. For our use case, the announcements must contain two pieces of information:
  * The ID of the coin that created this coin. This is already implicitly contained in the Coin Announcement.
  * The intended recipient's coin ID. The announcement's message must contain this information in order to prevent attacks where coins can receive messages that weren't intended for them.

To form the ring, every coin outputs the following conditions:
```
(
  (CREATE_COIN_ANNOUNCEMENT previous_coin_ID + sum_of_deltas_before_me)
  (ASSERT_COIN_ANNOUNCEMENT sha256(next_coin_id + my_coin_id + sum_of_deltas_including_me))
)
```
Where `+` represents concatenation of bytes, and announcement assertions take the hash of the announcement creator's ID and message.

In order to create the `next_coin_id`, we wrap the next coin's inner puzzle with the current coin's CAT information. This guarantees that the next_coin_id is a CAT of the same type as the current coin.

Because both coins follow the same CAT module code, they must comply with the same set of truths. This, in turn, guarantees that the whole ring is telling the truth. As long as the ring is connected, the total Delta must be zero.

</details>

## 额外的 Delta

上述零差值规则有两个例外：
   * 铸造硬币（从 XCH 创建 CAT）
   * 退役硬币（将 CAT 熔化为其原始 XCH 形式）

为了解决这些情况，TAIL 程序可能会批准误报 CAT 硬币的 Delta 特定数量，称为**额外的 Delta**。 这是传递给 TAIL 程序的参数之一。

有一些规则可以确保不会铸造或淘汰额外的硬币：
   * 如果额外的 Delta 不是 `0`，则 TAIL 程序将被强制运行。 它必须评估是否允许额外的增量，或失败与`(x)` 调用。
   * 如果方案中的额外的 Delta 值没有导致 TAIL 程序失败，那么它会自动添加到报告的 Delta 中，用于通告环中。
   * 如果 TAIL 程序未显示且额外的 Delta 不为 `0`，则谜语将失败。

<details>
<summary>原文参考</summary>

- ## Extra Delta

There are two exceptions to the aforementioned zero-Delta rule:
  * Minting coins (creating CATs from XCH)
  * Retiring coins (melting CATs to their original XCH form)

To account for these cases, the TAIL program may approve a misreporting of a CAT coin's Delta by a specific amount, called the **Extra Delta**. This is one of the parameters passed to the TAIL program.

There are a few rules to ensure that extra coins are not minted or retired:
  * If the Extra Delta is anything other than `0`, the TAIL program is forced to run. It must evaluate whether to permit the Extra Delta, or fail with an `(x)` call.
  * If the Extra Delta value in the solution does not cause the TAIL program to fail, then it is automatically added to the reported Delta, which is used in the announcement ring.
  * If the TAIL program is not revealed and the Extra Delta is not `0`, then the puzzle will fail.

</details>

## 代币和资产发行限制器（TAIL）计划

TAIL 程序功能强大且灵活。它可以控制和访问许多事物。这给了程序员很大的控制权，但也有很大的责任。

> **警告**：如果 TAIL 没有正确编程，
> 然后攻击者可能会铸造代币，
> 使资产变得毫无价值。

TAIL 应遵循所有负责锁定资金的 Chialisp 程序应遵循的所有常规安全规则。

必须将几个参数传递给 TAIL 的谜底：
  * 真相 —— 这些被组合在一起，如上所述
  * parent_is_cat —— 指示父项是否已被验证为与此 CAT 类型相同的 CAT 的标志
  * lineage_proof —— （可选）证明父代是与此 CAT 相同类型的 CAT
  * delta —— 额外的 Delta 值，如上所述
  * inner_conditions —— 内部谜语返回的条件
  * tail_solution —— （可选）不透明参数列表

TAIL虽然强大，但是每次花完币都**不一定**运行。如果在内部谜语中创建了“魔法”条件，则运行 TAIL。需要这个“魔法”条件来防止可以花费 TAIL 的人拦截花费并违背花费者的意愿改变它。

TAIL 应认真检查以下事项：
  * 额外的 Delta 是否铸造或淘汰任何硬币，如果是，我是否同意？
  * 如果这枚硬币的父母不是与我相同类型的 CAT，我是否同意？

<details>
<summary>原文参考</summary>

- ## The Token and Asset Issuance Limiter (TAIL) Program

The TAIL program is powerful and flexible. It has control over, and access to, many things. This gives a programmer a lot of control, but also a great deal of responsibility.

>**Warning**: If the TAIL is not programmed correctly,
>then tokens may be minted by attackers,
>rendering the asset worthless.

A TAIL should follow all of the conventional rules of security that any Chialisp program responsible for locking up money should follow.

Several parameters must be passed to a TAIL's solution:
  * Truths - These are bundled together, as explained above
  * parent_is_cat - A flag indicating whether the parent has been validated as a CAT of the same type as this CAT
  * lineage_proof - (optional) Proof that the parent is a CAT of the same type as this CAT
  * delta - The Extra Delta value, as explained above
  * inner_conditions - The conditions returned by the inner puzzle
  * tail_solution - (optional) A list of opaque parameters

Although the TAIL is powerful, it is **not necessarily** run every time the coin is spent. The TAIL is run if a "magic" condition is created in the inner puzzle. This "magic" condition is required to prevent people who can spend the TAIL from intercepting the spend and changing it against the spender's will.

The TAIL should check diligently for the following things:
  * Is the Extra Delta minting or retiring any coins, and if so, do I approve?
  * If this coin's parent is not a CAT of the same type as me, do I approve?

</details>

## TAIL 力量的极限

在以太坊中，代币的发行者可能有能力在未经所有者许可的情况下冻结或没收资金。这在Chia是不可能的。让我们来探究一下原因。

在 Chia 中，发行人创建了一个 TAIL，它存在于所有相同类型的 CAT 中，包括那些已经分发的 CAT。但是，发行人无权使用他们不拥有的硬币。 TAIL 只能作为 CAT 支出的最后一步运行，并且 CAT 的所有者（而不是发行人）负责提供其解决方案。这意味着只有所有者才能运行 TAIL。因此，CAT 的所有者是唯一有能力完成支出的人。

这一决定为用户增加了一些权力下放。它还增加了一些复杂性。创建结构良好的 TAIL 的重要性怎么强调都不为过。一旦您分发了 CAT，就无法再更改整个代币供应中的 TAIL。 TAIL 永远被锁定在同一组规则中。改变 TAIL 无异于更换流通中的实物现金。您将不得不提供新代币的交换，并最终弃用旧代币，即使有些人仍然持有它。

这也意味着，如果这组规则遭到破坏，人们可能会恶意地铸造或淘汰 CAT。除了通过上述过程之外，没有简单的方法可以“修补”TAIL，这显然最好避免。

<details>
<summary>原文参考</summary>

- ## The limits of a TAIL's power

In Ethereum, a token's issuer might have the ability to freeze or confiscate funds without the owner's permission. This is not possible in Chia. Let's explore why.

In Chia, an issuer creates a TAIL, which lives inside all CATs of the same type, including those that have already been distributed. However, the issuer does not have the power to spend coins they do not own. A TAIL can only run as the last step in a CAT's spend, and the owner of the CAT (and not the issuer) is responsible for providing its solution. This means that only the owner can run the TAIL. Therefore, the CAT's owner is the only one with the ability to complete the spend.

This decision adds some decentralization for users. It also adds some complexity. The importance of creating a well-constructed TAIL cannot be emphasized enough. Once you have distributed a CAT, it is no longer possible to change the TAIL across the entire token supply. The TAIL is locked into the same set of rules forever. To change the TAIL would be tantamount to replacing physical cash in circulation. You would have to offer an exchange for new tokens and eventually deprecate the old token, even if some people still carry it.

It also means that if the set of rules is compromised, people may be able to mint or retire CATs maliciously. There’s no easy way to “patch” the TAIL, other than through the process above, which is obviously best avoided.

</details>

## TAIL 示例

CAT1 标准目前包括三个示例 TAIL，但可能还有更多示例。

* [一次性铸造](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/genesis-by-coin-id-with-0.clvm "Chialisp code for the One-Time Minting TAIL")

  我们目前发行 CAT 的默认方式是使用只允许从特定硬币 ID 创建硬币的 TAIL。在 Chia 中，硬币只能使用一次，因此这会导致一次性铸造 CAT。发行后，CAT 将永远不会增加或减少，也没有人能够再次运行相同的 TAIL。

* [带签名的一切](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/everything_with_signature.clvm "Chialisp code for the Everything With Signature TAIL")

  与上面的 TAIL 截然相反的是，创建者可以为所欲为，只要他们提供公钥的签名即可。这个密钥被压缩到 TAIL 中，它返回一个单一的 AGG_SIG_ME 条件，要求一个匹配的签名。如果创建者可以提供该签名，则支出通过并且任何违反的供应规则都将被忽略。

  请记住，AGG_SIG_ME 只允许签名对单个硬币起作用。因此，创作者不能发布签名供所有人使用；相反，创建者必须亲自签署每个 TAIL 执行。

* [委托 TAIL](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/delegated_genesis_checker.clvm "Chialisp code for the Delegated TAIL")

  这是我们目前拥有的安全性和灵活性的最佳平衡。 委托 TAIL 类似于 `Everything With Signature` 示例，不同之处在于它不需要来自特定硬币的签名，而是需要来自特定谜语哈希的签名。当谜语哈希被签名后，创建者可以运行该谜语来代替 TAIL。
  
  这个 TAIL 允许创建者创建他们可以与 CAT 一起使用的新 TAIL，即使这些 TAIL 在初始发布期间不存在！例如，创建者可以创建：
  
  * 一次铸造
  * 铸造新硬币的 DID
  * 他们想要的任何其他东西！
  
  请注意，我们使用 AGG_SIG_UNSAFE 以使此签名适用于所有硬币。创建者可以发布有效签名，允许 CAT 的任何所有者自行运行 TAIL。这很有用的一种情况是在赎回计划中——你希望允许人们将他们的 CAT 融入 XCH，只要他们在这样做时遵循某些规则。
  
  当您签署新的委托 TAIL 时，还有另一个考虑因素。一旦你签署并发布签名，它就会永远存在。请注意您授予的权限，因为您永远无法收回它们。

<details>
<summary>原文参考</summary>

- ## TAIL Examples

The CAT1 standard currently includes three example TAILs, though many more are possible.

* [One-Time Minting](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/genesis-by-coin-id-with-0.clvm "Chialisp code for the One-Time Minting TAIL")

  The default way in which we currently issue CATs is with a TAIL that only allows coin creation from a specific coin ID. In Chia, coins can only be spent once, so this results in a one-time minting of a CAT. After the issuance, there will never be any more or less of the CAT, and no one will be able to run the same TAIL ever again.

* [Everything With Signature](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/everything_with_signature.clvm "Chialisp code for the Everything With Signature TAIL")
  
  The polar opposite of the TAIL above is the ability of the creator to do whatever they want, as long as they provide a signature from their public key. This key is curried into the TAIL, which returns a single AGG_SIG_ME condition asking for a matching signature. If the creator can provide that signature, then the spend passes and any supply rules that were violated are ignored.
  
  Keep in mind that AGG_SIG_ME only allows the signature to work on a single coin. Therefore, the creator cannot release a signature for everyone to use; instead the creator must personally sign every TAIL execution.

* [Delegated TAIL](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/delegated_genesis_checker.clvm "Chialisp code for the Delegated TAIL")

  This is the best balance of security and flexibility that we currently have. The Delegated TAIL is similar to the "Everything With Signature" example, except instead of requiring a signature from a specific coin, it requires a signature from a specific puzzlehash. When the puzzlehash has been signed, the creator may run that puzzle in place of the TAIL.
  
  This TAIL allows the creator to create new TAILs that they can use with the CAT, even if those TAILs didn't exist during the initial issuance! For example, the creator could create:
  
  * A single minting
  * A DID to mint new coins
  * Anything else they want!
  
  Note that we use AGG_SIG_UNSAFE in order to make this signature work for all coins. The creator can publish a valid signature, allowing any owner of the CAT to run the TAIL on their own. One scenario where this is useful is in redemption schemes -- you want to allow people to melt their CATs into XCH as long as they follow certain rules when they do so.
  
  There is another consideration to make when you are signing new Delegated TAILs. Once you sign it and publish the signature, it is out there forever. Be careful what permissions you grant because you can never take them back.

</details>

## CAT 面额、价值和退休规则

关于 CAT 与 XCH 的粒度和面额的一些设计决策：

* 大多数 Chia 钱包选择在 XCH 中显示其价值。然而，这纯粹是一种装饰性的选择，因为 Chia 的区块链只知道 mojos。一个 XCH 等于一万亿（1,000,000,000,000）个魔力。
* 同样，默认决定将 1 CAT 映射到 1000 XCH mojo。默认情况下，该比率对于所有 CAT 都是相同的。
* 可以将特定 CAT 的 CAT:mojo 比率设置为 1:1000 以外的值，但这样做可能会对令牌之间的互操作性产生负面影响。我们建议您使用默认设置，除非您有充分的理由不这样做。
* 因此，单个代币的默认熔炼值为 1000 mojo。无论代币的面值或流通量如何，这都是正确的。
* 代币的面值与其熔值不一定相关，更不用说匹配了。

以此类推，美国财政部曾多次提出铸造价值 1 万亿美元的铂金硬币的想法。撇开这样做的实际意义不谈，这种假设硬币的面值和熔化值之间的差异幅度将类似于 CAT 和 XCH。这枚硬币价值 1 万亿美元，但如果有人将其熔化并出售铂金，他们只会得到该金额的一小部分。

另一方面，考虑一下美国便士。其基本金属（97.5% 的锌和 2.5% 的铜）价值约 2 美分。所以从理论上讲，如果你能在最小化成本的同时将一分钱熔化成锌和铜，你就可以出售这些金属以获得可观的利润。

**XCH 和 CAT 的价值**

XCH 和 CAT 的面值都是由市场驱动的——无论有人愿意为它们支付什么，硬币都值得。

如果 CAT 在财务上取得一定程度的成功，其面值将大于其熔化价值，就像 1 万亿美元的硬币将比铸造它的金属更有价值。例如：

* 一个模因令牌可以交易 XCH 的百万分之一，或 1,000,000 mojo。代币价值很少，但它的价值仍然是其 1000 mojo 的熔化价值的 1000 倍。
* 美元支持的稳定币面值为 1 美元。它的熔化值为 1000 mojo。
* 一个非常成功的代币甚至可以卖出一个以上的 XCH——没有规则阻止这种情况发生。它的熔化值仍然是 1000 mojo。

这三种情况的一个现实类比是，您可以从一块金属开始铸造一枚价值不到一美分、1 美元、10,000 美元或任何其他价值的硬币。但无论面值如何，熔体值始终保持不变。

**CAT 退休用例和规则**

重要的是要记住，CAT 的 TAIL（以及其他任何东西）决定了退休的规则，_if_它完全允许退休。例如，我们的单铸 TAIL 只适用于特定的硬币，因此它不允许退休。使用这个 TAIL 的 CAT 永远不会被融化，无论它们的面值有多小。

我们委托的 TAIL 完全取决于 CAT 的创建者是否 - 以及如何 - 退休。

除了我们预先打包的示例之外，具有广泛功能的 TAIL 也是可能的。为了仅说明此功能的一部分，让我们考虑四个潜在的退休原因，以及谁可以退休代币：

**1. 从流通中删除（必须是创建者和所有者）**

  对于某些类别的 CAT（例如，稳定币和赎回代币），将允许退休，但仅限于创建者，他也必须拥有代币。在稳定币的情况下，如果支持资金减少，创建者可能需要从流通中移除一些代币。对于赎回代币，所有者可以与创建者交换代币以换取有价值的东西。代币不再具有任何面值，因此其创建者会将其从流通中移除。

**2. 价值交换（必须是所有者）**

  一些 CAT 可能允许其所有者融化代币以获得其他有价值的东西，例如 NFT 或其他 CAT。事实上，整个市场都可以从这个概念中产生。一些可能的例子：

  * 一组 NFT 的创建者还会发行少量“金票”，可用于在该组公开之前挑选出任何单个 NFT。
  * 名人铸造了一些代币，可以兑换一些非货币价值的东西，例如与名人会面。
  * CAT 持有者必须提交“融化证明”才能参加比赛。

**3. 临时代币（必须是创建者或预设算法）**

  CAT 可以创建为限时优惠或音乐椅游戏。在这些情况下，代币将被_违背所有者的意愿_融化。这可以随机完成，也可以作为一种刻意的削减。

**4. 熔体值检索（必须是所有者）**

  如果 CAT 在财务上不成功，其熔化价值可能会超过其面值，就像构成一美分的金属价值超过 1 美分一样。在这种情况下，CAT 的所有者通过将其分解为 1000 个 XCH mojo 来退役代币可能具有经济意义。由于代币的默认熔化值较低，这种熔化的动机可能很少见。

在这些示例中的每一个中，特定 CAT 的退役规则都在 TAIL 中明确说明。如果 TAIL 允许违背所有者的意愿退休，所有者将能够在获取代币之前确定此信息。

<details>
<summary>原文参考</summary>

- ## CAT denominations, value, and retirement rules

Some design decisions regarding the granularity and denomination of CATs versus XCH:

* Most Chia wallets choose to display their value in XCH. However, this is a purely cosmetic choice because Chia's blockchain only knows about mojos. One XCH is equal to one trillion (1,000,000,000,000) mojos.
* In a similar vein, a default decision was made to map 1 CAT to 1000 XCH mojos. By default, this ratio will be the same for all CATs.
* It is possible to set the CAT:mojo ratio to something other than 1:1000 for a specific CAT, but doing so could negatively affect interoperability between tokens. We recommend that you use the default setting unless you have a good reason to do otherwise.
* Therefore, the default melt value of a single token is 1000 mojos. This remains true regardless of the token's face value or its circulating supply.
* A token's face value and its melt value are not necessarily correlated, let alone matched.

By analogy, on multiple occasions the US Treasury has floated the idea of minting a $1 trillion coin made from platinum. Leaving aside the practical implications of doing this, the magnitude of the difference between this hypothetical coin's face value and melt value would be similar to that of CATs and XCH. The coin would be worth $1 trillion dollars, but if someone melted it and sold the platinum, they'd only receive a minuscule fraction of that amount.

On the other end of the spectrum, consider the US penny. Its base metals (97.5% zinc and 2.5% copper) are worth around two cents. So in theory, if you could melt a penny into zinc and copper while minimizing your costs, you could sell these metals for a sizable profit.

**The value of XCH and CATs**

The face value of both XCH and CATs is market-driven -- the coins are worth whatever someone is willing to pay for them.

If a CAT achieves a certain level of financial success, its face value will be greater than its melt value, just like the $1 trillion coin would be worth more than the metal it was minted from. For example:

* A meme token could trade for one-millionth of an XCH, or 1,000,000 mojos. The token is worth very little money, but it's still 1000 times more valuable than its melt value of 1000 mojos.
* A dollar-backed stablecoin will have a face value of $1. Its melt value will be 1000 mojos.
* A highly successful token could even sell for more than one XCH -- there are no rules preventing this from happening. Its melt value would still be 1000 mojos.

One real-world analogy for these three cases is that you could start with a piece of metal and mint a coin worth a fraction of a cent, or $1, or $10,000, or really any other value. But no matter the face value, the melt value would always remain the same.

**CAT retirement use cases and rules**

It's important to keep in mind that a CAT's TAIL (and nothing else) decides the rules for retirement, _if_ it allows retirement at all. For example, our single-mint TAIL only works with a specific coin, so it does not allow retirement. CATs that use this TAIL can never be melted, no matter how small their face value.

Our delegated TAIL leaves it entirely up to the CAT's creator whether -- and how -- retirement can happen.

Beyond our pre-packaged examples, TAILs with a wide range of functionality are also possible. To illustrate just some of this functionality, let's consider four potential reasons for retirement, along with who gets to retire the tokens:

**1. Removal from circulation (must be the creator AND owner)**

  For certain categories of CATs (for example, stablecoins and redemption tokens), retirement will be allowed, but only by the creator, who also must own the tokens. In the case of a stablecoin, the creator may need to remove some tokens from circulation if backing funds are reduced. For redemption tokens, the owner may exchange a token with the creator for something of value. The token no longer has any face value, so its creator will remove it from circulation.

**2. Value exchange (must be the owner)**

  Some CATs might allow their owners to melt tokens in order to gain something else of value, for example NFTs or other CATs. In fact, an entire marketplace could emerge from this concept. Some possible examples:

  * The creator of a set of NFTs also creates a small issuance of "golden tickets" that can be used to pick out any individual NFT before the set is made publicly available.
  * A celebrity mints some tokens that can be exchanged for something of non-monetary value, such as a meeting with said celebrity.
  * The holder of a CAT must submit a "proof of melt" in order to enter a contest.

**3. Ephemeral tokens (must be the creator OR a preset algorithm)**

  A CAT could be created as a limited-time offer or as a game of Musical Chairs. In these cases, tokens would be melted _against the owner's will_. This could be done either at random or as a deliberate type of slashing. 

**4. Melt-value retrieval (must be the owner)**

  If a CAT is not financially successful, its melt value could exceed its face value, in the same way that the metals that compose a US penny are worth more than one cent. In this case, it might make financial sense for the CAT's owner to retire a token by melting it into 1000 XCH mojos. Because of the low default melt value of tokens, this motivation for melting will likely be rare.

In each of these examples, the rules of retirement for a specific CAT are clearly spelled out in the TAIL. If a TAIL allows for retirement against the owner's will, the owner will be able to ascertain this information before acquiring the token.

</details>

## 结论

CAT1 标准是 Chia 生态系统的一个令人兴奋的补充。它允许发行可替代代币的近乎无限的功能。我们很高兴看到 Chia 社区提出了什么样的创意！

<details>
<summary>原文参考</summary>

- ## Conclusion

The CAT1 standard is an exciting addition to Chia's ecosystem. It allows near-limitless functionality for issuing fungible tokens. We're excited to see what kind of creative ideas the Chia community comes up with!

</details>
