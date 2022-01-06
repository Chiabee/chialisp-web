---
id: coins_spends_and_wallets
title: 2 - 硬币、花费和钱包
---

> Coins, Spends and Wallets

本指南假定您了解 [CLVM 的基础知识](/docs/)，因此如果您尚未阅读该页面，请在阅读本文之前先阅读。

本指南的这一部分将涵盖评估程序内的程序、ChiaLisp 如何与 Chia 网络上的交易和硬币相关联，并涵盖一些使用 ChiaLisp 创建智能交易的技术。如果有任何您不确定的术语，请务必查看[术语表](/docs/glossary)。

<details>
<summary>原文参考</summary>

This guide assumes knowledge of [the basics of CLVM](/docs/) so if you haven't read that page, please do so before reading this.

This section of the guide will cover evaluating a program inside a program, how ChiaLisp relates to transactions and coins on the Chia network, and cover some techniques to create smart transactions using ChiaLisp.
If there are any terms that you aren't sure of, be sure to check the [glossary](/docs/glossary).

</details>

## 谜语和谜底

当我们在区块链上的硬币上下文中提及 Chialisp 时，我们将程序称为**谜语**，我们将环境或参数称为**谜底**。

```
brun <puzzle> <solution>
```

每当您想在 Chia 上花费一枚硬币时，您必须揭示它的谜语以及您想用来运行该谜语的谜底。如果谜语运行没有任何错误并返回一个有效的**条件**列表（更多关于下面的条件），则支出成功并处理条件列表。

<details>
<summary>原文参考</summary>

- ## Puzzles and Solutions

When we refer Chialisp in the context of coins on the blockchain, we refer to the program as a **puzzle** and we refer to the environment or arguments as the **solution**.

```
brun <puzzle> <solution>
```

Whenever you want to spend a coin in Chia, you must reveal its puzzle and the solution you would like to use to run that puzzle.
If the puzzle runs without any errors and returns a valid list of **conditions** (more on conditions below) then the spend succeeds and the list of conditions is processed.

</details>

## 币

一个币的主体由 3 条信息组成。这是定义币的实际代码：

```python
class Coin:
    parent_coin_info: bytes32
    puzzle_hash: bytes32
    amount: uint64
```

1. 其父 ID
2. 谜语的树哈希（又名谜语哈希）
3. 它的价值

要构造一个币的 ID，只需将这 3 条信息按顺序连接起来的哈希值即可。

```
coinID == sha256(parent_ID + puzzlehash + amount)
```

这意味着币的谜语和金额是它的固有部分。您不能更改币的谜语或金额，您只能花费一个币。

<details>
<summary>原文参考</summary>

- ## Coins

The body of a coin is made up of 3 pieces of information.
Here is the actual code that defines a coin:

```python
class Coin:
    parent_coin_info: bytes32
    puzzle_hash: bytes32
    amount: uint64
```

1. The ID of its parent
2. The tree hash of its puzzle (AKA the puzzlehash)
3. The amount that it is worth

To construct a coin ID simply take the hash of these 3 pieces of information concatenated in order.

```
coinID == sha256(parent_ID + puzzlehash + amount)
```

This means that a coin's puzzle and amount are intrinsic parts of it.
You cannot change a coin's puzzle or amount, you can only spend a coin.

</details>

## 花费

当你花掉一个币时，你就摧毁了它。除非谜语的行为指定在花费时如何处理硬币的价值，否则币的价值也会在花费中被破坏。

要花费一个币，您需要 3 条信息（以及可选的第 4 条信息）。

1. 币的 ID
2. 币谜语的完整来源
3. 币谜语的解法
4. （可选）一组签名组合在一起，称为聚合签名

请记住，谜语和谜底与我们在基础知识中介绍的相同，只是谜语已经存储在币里面并且任何人都可以提交谜底。

网络没有硬币所有权的概念，任何人都可以尝试在网络上花费任何硬币。由谜语来防止硬币被盗或以意外方式花费。

如果有人可以提交硬币的谜底，您可能想知道某人如何“拥有”硬币。在指南的下一部分结束时，希望它应该是清楚的。

<details>
<summary>原文参考</summary>

- ## Spends

When you spend a coin you destroy it.
Unless the behaviour of a puzzle designates what to do with the coin's value when it is spent, the value of the coin is also destroyed in the spend.

To spend a coin you need 3 pieces of information (and an optional 4th).

1. The coin's ID
2. The full source of the coin's puzzle
3. A solution to the coin's puzzle
4. (OPTIONAL) A collection of signatures grouped together, called an aggregated signature

Remember the puzzle and solution is the same as we covered in the basics, except the puzzle has already been stored inside the coin and anybody can submit a solution.

The network has no concept of coin ownership, anybody can attempt to spend any coin on the network.
It's up to the puzzles to prevent coins from being stolen or spent in unintended ways.

If anybody can submit a solution for a coin, you may be wondering how somebody can "own" a coin.
By the end of the next section of the guide, hopefully it should be clear.

</details>

## 实践中的谜语和谜底

到目前为止，我们已经介绍了将评估某些结果的 ChiaLisp 程序。请记住，第一部分代表一个致力于锁定币的谜语，第二部分是任何人都可以提交的谜底：

```chialisp
$ brun '(+ 2 5)' '(40 50)'
90

$ brun '(c (q . 800) 1)' '("some data" 0xdeadbeef)'
(800 "some data" 0xdeadbeef)
```

这些是孤立的有趣练习，但这种格式可用于向区块链网络传达有关币在花费时应如何表现的指令。这可以通过将评估结果作为条件列表来完成。

<details>
<summary>原文参考</summary>

- ## Puzzles and Solutions in Practice

So far we have covered ChiaLisp programs that will evaluate to some result.
Remember the first part represents a puzzle which is committed to locking up a coin, and the second part is a solution anybody can submit:

```chialisp
$ brun '(+ 2 5)' '(40 50)'
90

$ brun '(c (q . 800) 1)' '("some data" 0xdeadbeef)'
(800 "some data" 0xdeadbeef)
```

These are fun exercises in isolation, but this format can be used to communicate instructions to the blockchain network of how a coin should behave when it is spent.
This can be done by having the result of an evaluation be a list of **conditions**.

</details>

### 条件

条件分为两类：*"此支出仅在 X"* 时有效，*"如果此支出有效则 X"*。

这是条件及其格式和行为的完整列表。

* **AGG_SIG_UNSAFE - [49] - (49 pubkey message)**：仅当附加的聚合签名包含来自给定消息的给定公钥的签名时，此支出才有效。这被标记为不安全，因为如果您对一条消息进行一次签名，则您拥有的任何其他需要该签名的硬币也可能会被解锁。由于硬币 ID 引入的自然熵，最好仅使用 AGG_SIG_ME。
* **AGG_SIG_ME - [50] - (50 pubkey message)**：仅当附加的聚合签名包含来自该消息的指定公钥的签名与硬币的 ID 和网络的创世挑战连接时，此支出才有效。
* **CREATE_COIN - [51] - (51谜语哈希金额)**：如果此支出有效，则使用给定的谜语哈希和金额创建一个新硬币。
* **RESERVE_FEE - [52] - (52 amount)**：仅当本次交易中存在大于或等于*金额*的未使用价值（明确用作费用）时，此支出才有效。
* **CREATE_COIN_ANNOUNCEMENT - [60] - (60 message)**：如果此支出有效，则会创建一个临时公告，其 ID 取决于创建它的代币。其他币然后可以断言存在用于块内币间通信的公告。
* **ASSERT_COIN_ANNOUNCEMENT - [61] - (61 noticeID)**：只有在此区块中有与announcementID 匹配的公告时，此支出才有效。
announcementID 是宣布消息的哈希值与宣布它的硬币的硬币 ID 连接起来`announcementID == sha256(coinID + message)`。
* **CREATE_PUZZLE_ANNOUNCEMENT - [62] - (62 message)**：如果此支出有效，则会创建一个临时公告，其 ID 取决于创建它的谜语。其他币然后可以断言存在用于块内币间通信的公告。
* **ASSERT_PUZZLE_ANNOUNCEMENT - [63] - (63 noticeID)**：只有在此区块中有与 announcementID 匹配的公告时，此支出才有效。
announcementID 是宣布的消息与宣布它的硬币的谜语哈希连接`announcementID == sha256(puzzle_hash + message)`。
* **ASSERT_MY_COIN_ID - [70] - (70 coinID)**：仅当提供的硬币 ID 与包含此谜语的硬币 ID 完全相同时，此支出才有效。
* **ASSERT_MY_PARENT_ID - [71] - (71 parentID)**：只有当提供的父代币信息与包含此谜语的代币的父代币信息完全相同时，此支出才有效。
* **ASSERT_MY_PUZZLEHASH - [72] - (72puzzlehash)**：仅当提供的谜语哈希与包含此谜语的硬币的谜语哈希完全相同时，此支出才有效。
* **ASSERT_MY_AMOUNT - [73] - (73 amount)**：仅当显示的金额与包含此谜语的硬币的数量完全相同时，此支出才有效。
* **ASSERT_SECONDS_RELATIVE - [80] -（80 秒）**：此支出仅在自该硬币创建后经过给定时间后才有效。硬币的创建时间或“生日”由前一个块的时间戳定义，*而不是*创建它的实际块。类似地，在评估这些时间锁时，前一个块的时间戳用作当前时间。
* **ASSERT_SECONDS_ABSOLUTE - [81] - (81 time)**：仅当此区块上的时间戳大于指定的时间戳时，此支出才有效。同样，硬币的生日和当前时间由前一个块的时间戳定义。
* **ASSERT_HEIGHT_RELATIVE - [82] - (82 block_age)**：此支出仅在自该硬币创建以来经过指定数量的区块时才有效。
* **ASSERT_HEIGHT_ABSOLUTE - [83] - (83 block_height)**：此支出仅在达到给定的 block_height 时才有效。

条件以以下形式的列表列表形式返回：

```chialisp
((51 0xabcd1234 200) (50 0x1234abcd "hello") (60 0xdeadbeef))
```

*记住：这是一个谜语在提出谜底时应该评估的内容，以便全节点可以理解它。*

让我们创建一些示例谜语和谜底来演示如何在实践中使用它。

<details>
<summary>原文参考</summary>

- ### Conditions

Conditions are split into two categories: *"this spend is only valid if X"* and *"if this spend is valid then X"*.

Here is the complete list of conditions along with their format and behaviour.

* **AGG_SIG_UNSAFE - [49] - (49 pubkey message)**: This spend is only valid if the attached aggregated signature contains a signature from the given public key of the given message. This is labeled unsafe because if you sign a message once, any other coins you have that require that signature may potentially also be unlocked. It's probably better just to use AGG_SIG_ME because of the natural entropy introduced by the coin ID.
* **AGG_SIG_ME - [50] - (50 pubkey message)**: This spend is only valid if the attached aggregated signature contains a signature from the specified public key of that message concatenated with the coin's ID and the network's genesis challenge.
* **CREATE_COIN - [51] - (51 puzzlehash amount)**: If this spend is valid, then create a new coin with the given puzzlehash and amount.
* **RESERVE_FEE - [52] - (52 amount)**: This spend is only valid if there is unused value in this transaction greater than or equal to *amount*, which is explicitly to be used as the fee.
* **CREATE_COIN_ANNOUNCEMENT - [60] - (60 message)**: If this spend is valid, this creates an ephemeral announcement with an ID dependent on the coin that creates it. Other coins can then assert an announcement exists for inter-coin communication inside a block.
* **ASSERT_COIN_ANNOUNCEMENT - [61] - (61 announcementID)**: This spend is only valid if there was an announcement in this block matching the announcementID.
The announcementID is the hash of the message that was announced concatenated with the coin ID of the coin that announced it `announcementID == sha256(coinID + message)`.
* **CREATE_PUZZLE_ANNOUNCEMENT - [62] - (62 message)**: If this spend is valid, this creates an ephemeral announcement with an ID dependent on the puzzle that creates it. Other coins can then assert an announcement exists for inter-coin communication inside a block.
* **ASSERT_PUZZLE_ANNOUNCEMENT - [63] - (63 announcementID)**: This spend is only valid if there was an announcement in this block matching the announcementID.
The announcementID is the message that was announced concatenated with the puzzle hash of the coin that announced it `announcementID == sha256(puzzle_hash + message)`.
* **ASSERT_MY_COIN_ID - [70] - (70 coinID)**: This spend is only valid if the presented coin ID is exactly the same as the ID of the coin that contains this puzzle.
* **ASSERT_MY_PARENT_ID - [71] - (71 parentID)**: This spend is only valid if the presented parent coin info is exactly the same as the parent coin info of the coin that contains this puzzle.
* **ASSERT_MY_PUZZLEHASH - [72] - (72 puzzlehash)**: This spend is only valid if the presented puzzle hash is exactly the same as the puzzle hash of the coin that contains this puzzle.
* **ASSERT_MY_AMOUNT - [73] - (73 amount)**: This spend is only valid if the presented amount is exactly the same as the amount of the coin that contains this puzzle.
* **ASSERT_SECONDS_RELATIVE - [80] - (80 seconds)**: This spend is only valid if the given time has passed since this coin was created. The coin's creation time or "birthday" is defined by the timestamp of the previous block *not* the actual block in which it was created. Similarly, the previous block's timestamp is used as the current time when evaluating these time locks.
* **ASSERT_SECONDS_ABSOLUTE - [81] - (81 time)**: This spend is only valid if the timestamp on this block is greater than the specified timestamp. Again, the coin's birthday and the current time are defined by the timestamp of the previous block.
* **ASSERT_HEIGHT_RELATIVE - [82] - (82 block_age)**: This spend is only valid if the specified number of blocks have passed since this coin was created.
* **ASSERT_HEIGHT_ABSOLUTE - [83] - (83 block_height)**: This spend is only valid if the given block_height has been reached.

Conditions are returned as a list of lists in the form:

```chialisp
((51 0xabcd1234 200) (50 0x1234abcd "hello") (60 0xdeadbeef))
```

*Remember: this is what a puzzle should evaluate to when presented with a solution so that a full-node can understand it.*

Let's create a few examples puzzles and solutions to demonstrate how this is used in practice.

</details>

### 示例 1：密码锁定硬币

让我们创建一个任何人只要知道密码就可以使用的硬币。

为了实现这一点，我们将密码的哈希提交到谜语中，如果提供正确的密码，谜语将返回指令以使用谜底中给出的谜语哈希创建新硬币。对于以下示例，密码为 “hello”，其哈希值为 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824。上述硬币的实现将是这样的：

```chialisp
(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))
```

该程序使用 `(sha256)`，使用 `2` 谜底中的第一个元素的哈希，并将该值与已提交的值进行比较。 如果密码正确，它将返回 `(c (q . 51) (c 5 (c (q . 100) ())))`，其计算结果为 `(51 0xmynewpuzzlehash 100)`。 请记住，`51` 是创建具有指定谜语哈希和金额的新硬币的条件的操作码。 `5` 等价于 `(f (r 1))`，我们使用它来访问解中的谜语哈希。

如果密码不正确，它将返回字符串“错误的密码”。

解决此问题的格式预计为 `(password newpuzzlehash)`。 请记住，只要知道硬币的 ID 和完整的谜语代码，任何人都可以尝试花费这枚硬币。

让我们使用 clvm\_tools 测试一下。


```chialisp
$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (c (q . 51) (c 5 (c (q . 100) ()))) (q ())) (q . "wrong password"))' '("let_me_in" 0xdeadbeef)'
"wrong password"

$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))' '("incorrect" 0xdeadbeef)'
"wrong password"

$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))' '("hello" 0xdeadbeef)'
((51 0xdeadbeef 100))
```

在这是一个完整的智能交易之前，我们需要做一个最后的改变。

如果您想使支出无效，则需要使用 `x` 引发异常。 否则，您只有一个不返回任何条件的有效支出，这将破坏我们的硬币而不是创建新的硬币！ 所以我们需要将失败条件更改为 `(x "wrong password")`，这意味着交易失败并且硬币没有被花费。

如果我们这样做，那么我们还应该将 `(i A B C)` 模式更改为 `(a (i A (q . B) (q . C)) 1)`。 原因在[后面的部分](https://github.com/Chiabee/chialisp-web/blob/main/docs/deeper_into_clvm)中有解释。 现在不要担心为什么。

这是我们完成的受密码保护的硬币：

```chialisp
'(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)'
```

让我们使用 clvm_tools 测试一下：

```chialisp
$ brun '(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)' '("let_me_in" 0xdeadbeef)'
FAIL: clvm raise ("wrong password")

$ brun '(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)' '("hello" 0xdeadbeef)'
((51 0xdeadbeef 100))
```

<details>
<summary>原文参考</summary>

- ### Example 1: Password Locked Coin

Let's create a coin that can be spent by anybody as long as they know the password.

To implement this we would have the hash of the password committed into the puzzle and, if presented with the correct password, the puzzle will return instructions to create a new coin with a puzzle hash given in the solution.
For the following example the password is "hello" which has the hash value 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824.
The implementation for the above coin would be thus:

```chialisp
(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))
```

This program takes the hash, with `(sha256 )`, of the first element in the solution, with `2`, and compares that value with the already committed.
If the password is correct it will return `(c (q . 51) (c 5 (c (q . 100) ())))` which evaluates to `(51 0xmynewpuzzlehash 100)`.
Remember, `51` is the opcode for the condition to create a new coin with the specified puzzle hash and amount. `5` is equivalent to `(f (r 1))` and we use it to access the puzzle hash from the solution.

If the password is incorrect it will return the string "wrong password".

The format for a solution to this is expected to be formatted as `(password newpuzzlehash)`.
Remember, anybody can attempt to spend this coin as long as they know the coin's ID and the full puzzle code.

Let's test it out using clvm_tools.

```chialisp
$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (c (q . 51) (c 5 (c (q . 100) ()))) (q ())) (q . "wrong password"))' '("let_me_in" 0xdeadbeef)'
"wrong password"

$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))' '("incorrect" 0xdeadbeef)'
"wrong password"

$ brun '(i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (c (q . 51) (c 5 (c (q . 100) ()))) (q . "wrong password"))' '("hello" 0xdeadbeef)'
((51 0xdeadbeef 100))
```

There is one final change we need to make before this is a complete smart transaction.

If you want to invalidate a spend then you need to raise an exception using `x`.
Otherwise you just have a valid spend that isn't returning any conditions, and that would destroy our coin and not create a new one!
So we need to change the fail condition to be `(x "wrong password")` which means the transaction fails and the coin is not spent.

If we're doing this then we should also change the `(i A B C)` pattern to `(a (i A (q . B) (q . C)) 1)`.
The reason for this is explained in [a later section](/docs/deeper_into_clvm/). For now don't worry about why.

Here is our completed password protected coin:

```chialisp
'(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)'
```

Let's test it out using clvm_tools:

```chialisp
$ brun '(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)' '("let_me_in" 0xdeadbeef)'
FAIL: clvm raise ("wrong password")

$ brun '(a (i (= (sha256 2) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q . (c (c (q . 51) (c 5 (c (q . 100) ()))) ())) (q . (x (q . "wrong password")))) 1)' '("hello" 0xdeadbeef)'
((51 0xdeadbeef 100))
```

</details>

### 从谜语和从谜底中生成条件

让我们花点时间考虑一下发送方和支出方之间的权力平衡。另一种表述方式是“谜底应该对输出有多少控制？”

假设我们使用以下谜语锁定硬币：

```chialisp
(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))
```

无论通过什么谜底，这个谜语都将*总是*返回指令以创建一个带有谜语哈希 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 和数量 100 的新硬币。

```chialisp
$ brun '(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))' '(80 90 "hello")'
((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100))

$ brun '(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))' '("it doesn't matter what we put here")'
((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100))
```

在这个例子中，花费硬币的结果完全由谜语决定。即使任何人都可以开始花费硬币，但锁定硬币的人拥有花费硬币的所有权力，因为谜底根本不重要。

相反，让我们考虑一个用以下谜语锁定的硬币：

```chialisp
1
```

这个例子可能看起来有点奇怪，因为大多数 ChiaLisp 程序都是列表，这只是一个原子，但它仍然是一个有效的程序。这个谜语只是返回整个谜底。您可以从权力和控制的角度考虑这一点。锁定硬币的人已将所有权力交给提供谜底的人。

```chialisp
$ brun '1' '((51 0xf00dbabe 50) (51 0xfadeddab 50))'
((51 0xf00dbabe 50) (51 0xfadeddab 50))

$ brun '1' '((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))'
((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))
```

在这种情况下，不仅任何人都可以花硬币，他们可以随心所欲地花！这种权力平衡决定了 ChiaLisp 中很多谜语的设计方式。

例如，让我们创建一个谜语，让消费者选择输出，但有一个规定。

```chialisp
(c (q . (51 0xcafef00d 200)) 1)
```

这将让花费者通过谜底返回他们想要的任何条件，但将始终添加条件以创建具有谜语哈希 0xcafef00d 和值 200 的硬币。

```chialisp
$ brun '(c (q . (51 0xcafef00d 200)) 1)' '((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))'
((51 0xcafef00d 200) (51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))
```

本节旨在说明条件可以来自接收方的谜底和发送方的谜语，以及这如何代表信任和权力平衡。

在下一个练习中，我们将把我们所知道的一切放在一起，并在 Chia 中创建一个基本的、安全的交易，以支持钱包如何相互汇款。在我们去那里之前，让我们解释一下签名：

<details>
<summary>原文参考</summary>

- ### Generating Conditions from the Puzzle vs. from the Solution

Let's take a moment to consider the balance of power between the send and the spender.
Another way of phrasing this is "how much control over the output should the solution have?"

Suppose we lock a coin up using the following puzzle:

```chialisp
(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))
```

Regardless of what solution is passed this puzzle will *always* return instructions to create a new coin with the puzzlehash 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a and the amount 100.

```chialisp
$ brun '(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))' '(80 90 "hello")'
((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100))

$ brun '(q . ((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100)))' '("it doesn't matter what we put here")'
((51 0x365bdd80582fcc2e4868076ab9f24b482a1f83f6d88fd795c362c43544380e7a 100))
```

In this example the result of spending the coin is entirely determined from the puzzle.
Even though anybody could initiate the spend of the coin, the person that locked the coin up has all the power in the way that the coin is spent as the solution doesn't matter at all.

Conversely lets consider a coin locked up with the following puzzle:

```chialisp
1
```

This example may look a little weird, because most ChiaLisp programs are lists, and this is just an atom, but it is still a valid program.
This puzzle simply returns the entire solution.
You can think about this in terms of power and control.
The person that locked the coin up has given all the power to the person who provides the solution.

```chialisp
$ brun '1' '((51 0xf00dbabe 50) (51 0xfadeddab 50))'
((51 0xf00dbabe 50) (51 0xfadeddab 50))

$ brun '1' '((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))'
((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))
```

In this situation, not only can anybody spend the coin, they can spend it however they like!
This balance of power determines a lot of how puzzles are designed in ChiaLisp.

For example, let's create a puzzle that lets the spender choose the output, but with one stipulation.

```chialisp
(c (q . (51 0xcafef00d 200)) 1)
```
This will let the spender return any conditions they want via the solution but will always add the condition to create a coin with the puzzle hash 0xcafef00d and value 200.

```chialisp
$ brun '(c (q . (51 0xcafef00d 200)) 1)' '((51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))'
((51 0xcafef00d 200) (51 0xf00dbabe 75) (51 0xfadeddab 15) (51 0x1234abcd 10))
```

This section is intended to demonstrate the point that conditions can come from both the recipient's solution and from the sender's puzzle, and how that represents trust and the balance of power.

In the next exercise we will put everything we know together and create a basic, secure transaction in Chia that underpins how wallets are able to send money to each other.
Before we go there, let's explain signatures:

</details>

## BLS 聚合签名

如果您对加密签名没有基本的了解，在继续之前最好先熟悉[基本概念](https://en.wikipedia.org/wiki/Digital_signature)。

您可能已经看到，上述条件之一允许您要求硬币花费者的**签名**。在 Chia，我们使用 [BLS 签名](https://crypto.stanford.edu/~dabo/pubs/papers/BLSmultisig.html) 来签署任何相关数据。

BLS 签名的一项有用功能是它们可以*非交互式聚合*。 您可以从不信任的一方获取签名，并将其与另一个签名组合以生成单个签名，以验证他们签名的所有消息的组合。

例如，如果谜语返回一组具有多个 AGG_SIG 条件的条件：

```
((AGG_SIG_UNSAFE <pubkey A> <msg A>) (AGG_SIG_UNSAFE <pubkey B> <msg B>))
```

处理此支出的节点将查找附加签名，该签名是来自公钥 A 对消息 A 的签名以及来自公钥 B 对消息 B 的签名的*聚合*。除非恰好有这样的组合，否则支出不会通过签名。不多也不少。

### 示例：签名锁定硬币

要“向某人发送硬币”，您只需创建一个需要收件人签名的谜语，然后允许他们返回他们喜欢的任何其他条件。这意味着代币不能被其他任何人使用，但输出完全由接收者决定。

我们可以构建以下智能交易，其中 AGG\_SIG\_ME 为 50，接收方的公钥为 `0xfadedcab` 。

```chialisp
(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) (q . ())))) 3)
```

这个谜语迫使结果评估包含 `(50 pubkey *hash_of_first_solution_arg*)`，然后添加谜底中提供的所有条件。

让我们在 clvm\_tools 中测试一下——在这个例子中，收件人的公钥将表示为 0xdeadbeef。接收者想要花费硬币来创建一个新硬币，该硬币与谜语 0xcafef00d 一起锁定。

```chialisp
$ brun '(c (c (q . 50) (c (q . 0xdeadbeef) (c (sha256 2) ()))) 3)' '("hello" (51 0xcafef00d 200))'
((50 0xdeadbeef 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824) (51 0xcafef00d 200))
```

精彩。

让我们撤回并在此处添加一些上下文。

<details>
<summary>原文参考</summary>

- ## BLS Aggregated Signatures

If you don't have a fundamental understanding of cryptographic signatures, it will be good to familiarize yourself with [the basic concepts](https://en.wikipedia.org/wiki/Digital_signature) before you continue.

You may have seen that one of the conditions above allows you to require a **signature** from the spender of the coin.
In Chia, we use [BLS Signatures](https://crypto.stanford.edu/~dabo/pubs/papers/BLSmultisig.html) to sign any relevant data.

One helpful feature of BLS signatures is that they can be *non-interactively aggregated*.  You can take a signature from a party you don't trust, and combine it with another signature to produce a single signature that verifies the combination of all of the messages they were signing.

For example, if a puzzle returns a set of conditions with multiple AGG_SIG conditions:
```
((AGG_SIG_UNSAFE <pubkey A> <msg A>) (AGG_SIG_UNSAFE <pubkey B> <msg B>))
```
the node processing this spend is going to look for an attached signature that is the **aggregation** of a signature from pubkey A on message A as well as a signature from pubkey B on message B.
The spend will not pass unless there is exactly that combination of signatures.  No more, no less.

- ### Example: Signature Locked Coin

To 'send a coin to somebody' you simply create a puzzle that requires the recipient's signature, but then allows them to return any other conditions that they like.
This means that the coin cannot be spent by anybody else, but the outputs are entirely decided by the recipient.

We can construct the following smart transaction where AGG_SIG_ME is 50 and the recipient's pubkey is `0xfadedcab`.

```chialisp
(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) (q . ())))) 3)
```

This puzzle forces the resultant evaluation to contain `(50 pubkey *hash_of_first_solution_arg*)` but then adds on all of the conditions presented in the solution.

Let's test it out in clvm_tools - for this example the recipient's pubkey will be represented as 0xdeadbeef.
The recipient wants to spend the coin to create a new coin which is locked up with the puzzle 0xcafef00d.

```chialisp
$ brun '(c (c (q . 50) (c (q . 0xdeadbeef) (c (sha256 2) ()))) 3)' '("hello" (51 0xcafef00d 200))'
((50 0xdeadbeef 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824) (51 0xcafef00d 200))
```

Brilliant.

Let's pull back and add some context here.

</details>

## 钱包

钱包是一些具有多种功能的软件，可以让用户轻松地与硬币进行交互。

* 钱包跟踪公钥和私钥
* 一个钱包可以产生谜语和谜底
* 钱包可以用它的密钥签名
* 钱包可以识别并记住用户“拥有”哪些硬币
* 一个钱包可以花硬币

您可能想知道如果有人可以尝试花费硬币，钱包如何能够识别用户“拥有”哪些硬币。这是因为所有钱包都已经知道并同意向某人发送硬币的标准格式。他们知道自己的公钥是什么，因此当创建一个新硬币时，钱包可以检查该硬币中的谜题是否是他们其中一个公钥的“标准发送谜语”。如果是，那么可以认为该硬币归该“钱包”所有，因为其他人无法使用它。

如果“拥有”硬币的钱包然后想再次将该硬币发送给其他人，他们会要求一个地址（这是一个 bech32m 编码的谜语哈希），然后他们可以花费他们拥有的硬币，销毁它，并创建一个新硬币，用新收件人的谜语哈希值锁定。新的接收者然后可以识别它“拥有”硬币并且可以在以后按照他们的意愿发送它。

### 改变

改变很简单。如果一个钱包的花费少于一枚硬币的总价值，他们可以用剩余的价值创造另一个硬币，并再次为自己用标准拼图将其锁定。 您可以将一枚硬币分成任意数量的新硬币，只要您愿意，即可使用原始价值的一小部分。

您不能从同一个父代创建两个具有相同拼图哈希值的相同价值的硬币，因为这将导致 ID 冲突并且支出将被拒绝。

### 硬币聚合和花费组合

您可以将一堆较小的硬币聚合成一个大硬币。为此，您可以创建一个花费组合，将一个或多个支出与单个聚合签名组合在一起。

花费组合在使用公告时尤为重要。 由于创建的公告仅适用于创建它们的区块，因此您需要确保声明这些公告的币与宣布币一起使用。

我们将在后面的部分中更多地讨论花费组合和硬币之间的凝聚力。

### 示例：支付给“委托谜语”

我们可以构建一个更强大的签名锁定硬币版本：

```chialisp
(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) ()))) (a 5 11))
```

第一部分基本相同，谜语总是返回一个 AGGSIG 检查公钥 `0xfadedcab` 。 然而，它只检查谜底的第一个元素。 这是因为，该谜语的谜底不是要打印出的条件列表，而是一个程序/谜底对。 这意味着接收者可以运行他们自己的程序作为谜底生成的一部分，或者签署一个谜语并让其他人提供谜底。 当我们使用程序参数生成谜底时，将其称为“委托谜语”。

计算谜底中的新程序和谜底，并将其结果添加到条件输出中。 我们将在本指南的[下一部分](https://github.com/Chiabee/chialisp-web/blob/main/docs/deeper_into_clvm)中更详细地介绍其工作原理。

此标准事务的基本谜底可能如下所示：

```chialisp
("hello" (q . ((51 0xmynewpuzzlehash 50) (51 0xanothernewpuzzlehash 50))) ())
```

在 clvm_tools 中运行它看起来像这样：

```chialisp
$ brun '(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) ()))) (a 5 11))' '("hello" (q . ((51 0xdeadbeef 50) (51 0xf00dbabe 50))) ())'

((50 0xfadedcab 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824) (51 0xdeadbeef 50) (51 0xf00dbabe 50))
```

<details>
<summary>原文参考</summary>

- ## Wallets

A wallet is some software that has several features that make it easy for a user to interact with coins.

* A wallet keeps track of public and private keys
* A wallet can generate puzzles and solutions
* A wallet can sign things with its keys
* A wallet can identify and remember what coins that the user 'owns'
* A wallet can spend coins

You may be wondering how a wallet is able to identify what coins that the user 'owns' if any person can attempt to spend a coin.
This is because all wallets already know and agree on what the standard format for sending a coin to somebody is.
They know what their own pubkeys are, so when a new coin is created a wallet can check if the puzzle inside that coin is a 'standard send puzzle' to one of their pubkeys.
If it is, then that coin can be considered to be owned by that 'wallet' as nobody else can spend it.

If the wallet that 'owns' the coin then wanted to send that coin on again to somebody else, they ask for an address (which is a bech32m encoded puzzle hash) and then they could then spend the coin that they own, destroying it, and creating a new coin that is locked up with the new recipient's puzzle hash.
The new recipient can then identify that it 'owns' the coin and can send it on as they wish later.

- ### Change Making

Change making is simple.
If a wallet spends less than the total value of a coin, they can create another coin with the remaining portion of value, and lock it up with the standard puzzle for themselves again.
You can split a coin up into as many new coins with fractions of the original value as you'd like.

You cannot create two coins of the same value, with the same puzzlehash, from the same parent as this will lead to an ID collision and the spend will be rejected.

- ### Coin Aggregation and Spend Bundles

You can aggregate a bunch of smaller coins together into one large coin.
To do this, you can create a SpendBundle which groups together one or more spends with a single aggregated signature.

SpendBundles are particularly important when the using announcements.
Since created announcements are only good for the block they are created in, you want to make sure that the coins that are asserting those announcements get spent alongside the announcing coins.

We'll go more into SpendBundles and cohesion between coins in a later section.

- ### Example: Pay to "Delegated Puzzle"

We can construct an even more powerful version of the signature locked coin:

```chialisp
(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) ()))) (a 5 11))
```

The first part is mostly the same, the puzzle always returns an AGGSIG check for the pubkey `0xfadedcab`.
However it only checks for the first element of the solution.
This is because, instead of the solution for this puzzle being a list of Conditions to be printed out, the solution is a program/solution pair.
This means that the recipient can run their own program as part of the solution generation, or sign a puzzle and let somebody else provide the solution.
When we use program parameters to generate solutions, refer to that as a "delegated puzzle".

The new program and solution inside the solution are evaluated and the result of that is added to the condition output.
We will cover in more detail how this works in the [next part](/docs/deeper_into_clvm/) of this guide.

A basic solution for this standard transaction might look like:

```chialisp
("hello" (q . ((51 0xmynewpuzzlehash 50) (51 0xanothernewpuzzlehash 50))) ())
```

Running that in the clvm_tools looks like this:

```chialisp
$ brun '(c (c (q . 50) (c (q . 0xfadedcab) (c (sha256 2) ()))) (a 5 11))' '("hello" (q . ((51 0xdeadbeef 50) (51 0xf00dbabe 50))) ())'

((50 0xfadedcab 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824) (51 0xdeadbeef 50) (51 0xf00dbabe 50))
```

</details>

## 结论

硬币所有权是指创造一个带有谜语的硬币的概念，这意味着它只能在由硬币“所有者”的私钥签名时才能使用。 钱包软件的目标是生成、解释和管理这些类型的硬币和谜语。

本指南的下一部分将更深入地介绍 ChiaLisp，并介绍如何编写更复杂的谜语。 如果指南的这一部分中的任何材料让您感到困惑，请尝试在下一部分之后返回。

<details>
<summary>原文参考</summary>

- ## Conclusions

Coin ownership refers to the concept of creating a coin with a puzzle that means it can only be spent when signed by the private key of the coin's "owner".
The goal of wallet software is to generate, interpret and manage these kinds of coins and puzzles.

The next part of this guide will go further in depth in ChiaLisp, and cover how to write more complex puzzles.
If any of the material in this part of the guide has got you confused, try returning to it after the next part.

</details>
