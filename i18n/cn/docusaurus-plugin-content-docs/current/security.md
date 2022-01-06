---
id: security
title: 8 - 安全
---

> Security

在编写 Chialisp 时，安全问题应该放在您的脑海中。该语言专门设计用于保护网络上的资金，而该网络没有*没有中央授权*来执行规则。唯一阻碍攻击者和潜在大笔资金的人就是你。

<details>
<summary>原文参考</summary>

When writing Chialisp, security concerns should be at the front of your mind.
The language is specifically designed to secure money on a network with *no centralized authority* to enforce rules.
The only person standing in the way of attackers and potentially large sums of money is going to be you.

</details>

## 签署和宣称谜底的真相

请记住[我们对硬币生命周期的讨论](/docs/coin_lifecycle)，当您推送一笔交易时，它会被传到其他节点，直到找到一个将其放入区块的节点。每个节点都选择将传递给区块的内容下一个节点。如果愿意，它可以在转发之前更改一些数据。

这就是为什么聚合签名是花费组合的一部分。它允许您将数据标记为有效，仅当还有一个签名保证其正确性时。签名是您防止节点以恶意方式更改您的交易的方式；如果他们这样做，支出将不再有效。

在查看谜底的值时签名尤其重要。谜语揭示由硬币上的谜语哈希保护。然而，谜底可以是任何东西。大多数情况下，当您花费硬币时，输出条件被传入以某种方式通过谜底。如果您不签署这些条件（或生成它们的委托谜语），您必须假设攻击者会注意到并尝试替换他们自己的值。

有时，需要具有逻辑上无法签名但也不应该更改的谜底的值。在此类情况下，您应该尝试使用签名币使用公告来宣称正在使用正确信息的币。

<details>
<summary>原文参考</summary>

- ## Signing and Asserting Solution Truth

Remember from [our discussion of coin lifecycles](/docs/coin_lifecycle) that when you push a transaction, it gets gossiped to other nodes until it finds one who will put it into a block.
Every node chooses what will be passed on to the next node. If it likes, it can change some data before it forwards it.

This is why the aggregated signature is part of the spend bundle.
It allows you to mark data as valid only if there is also a signature that vouches for its correctness.
Signatures are how you prevent nodes from changing your transaction in malicious ways; if they do, the spend will no longer be valid.

Signing is especially important when looking at solution values.
The puzzle reveal is secured by the puzzle hash on the coin.
The solution, however, can be anything.
Most of the time when you are spending a coin, a the output conditions are passed in somehow through the solution.
If you don't sign those conditions (or the delegated puzzle that generates them) you must assume that an attacker is going to notice and attempt to substitute their own values.

Sometimes, it is necessary to have solution values that logistically cannot be signed, but also should not be changed.
In scenarios like these, you should try to have a signed coin use announcements to assert that the coin is being spent with the correct information.

</details>

## 宣布币的信息

签名是您防止节点干扰您自己支出的方式，但有时您想创建将按照特定规则进行交易的代币。因此，您不知道谁将花费这些代币，您也不知道如果他们愿意的话。我们在[我们对外部谜语的讨论](/docs/common_functions#outer-and-inner-puzzles)中看到，您可以使用柯里化和包装树哈希对您的子代币实施规则，但有时当您还想强制执行有关你自己或你父代的真相时。

这就是 `ASSERT_MY_*` 系列操作码的用武之地。当你需要关于你的硬币的信息（`parent_coin_info`、`puzzle_hash`、`amount`）以在谜语中使用时，它不能总是由诚实的一方提供。有时，它需要通过谜底传入。谜底应始终被视为由恶意或粗心方解决。如果传入任何硬币信息，应使用操作码进行宣称以确保可以看到该信息的网络可以对其进行确认。

请记住，`ASSERT_MY_COIN_ID` 实际上会隐式宣布硬币中的所有三个信息。对于父代币，`ASSERT_MY_PARENT_ID` 也是如此，这特别有用，因为例如没有`ASSERT_MY_PARENT_PUZZLE_HASH` 这样的东西。

<details>
<summary>原文参考</summary>

- ## Asserting Coin Information

Signing is how you prevent nodes from messing with your own spends, but sometimes you want to create coins that will be traded around with specific rules.
As a result you don't know who will be spending the coin, and you don't know if they will be honest.
We saw in [our discussion of outer puzzles](/docs/common_functions#outer-and-inner-puzzles) that you can enforce rules on your child coins using currying and wrapping tree hashes, but there are times when you also want to enforce truths about yourself or your parent.

This is where the `ASSERT_MY_*` family of opcodes comes in.
When you need information (`parent_coin_info`, `puzzle_hash`, `amount`) about your coin to use in the puzzle, it cannot always be curried in by an honest party.
Sometimes, it will need to be passed in through the solution.
The solution should always be treated as if it is being solved by malicious or careless parties.
If any coin information is being passed in, it should be asserted with opcodes to ensure that the network, who can see that information, can confirm it.

Keep in mind that `ASSERT_MY_COIN_ID` will actually implicitly assert all three of the pieces of information in a coin. The same is true of `ASSERT_MY_PARENT_ID` for parent coins, which is particularly useful since there is no such thing as `ASSERT_MY_PARENT_PUZZLE_HASH`, for example.

</details>

## 重放攻击

创建支出时的另一个重大问题是，如果它们的一部分被排除或重复使用，它们是否有效。这种攻击是 `AGG_SIG_UNSAFE` 被标记为这种方式的原因。

如果你用 `AGG_SIG_UNSAFE` 签名，唯一被签名的数据就是你试图签名的消息。一旦你签名并推送它，这个签名就会永远存在于区块链上。如果你以后创建了一个被锁定的谜语由于需要相同的签名，攻击者可以找到你上次使用的签名并重用它。这就是为什么你应该尽可能尝试总是使用 `AGG_SIG_ME`。它不仅让你在签名（每次支出都是独一无二的），但它也致力于应对您所在网络的创世挑战。否则，可以在主网上重放测试网上硬币的公开签名。

排除也应该是您最关心的一个问题。通常，您会在同一个花费组合中花费多个硬币，并且它们应该全部捆绑在一起形成一个聚合签名。如果您有充分的理由不签署其中一个，确保您知道如果它被从花费组合中排除会发生什么。此外，聚合签名不能分解为更小的签名*除非*您之前已经签署了组合中公钥-消息对的较小组合之一。攻击者可以排除包含 `AGG_SIG` 条件的其余交易，并在剩余的交易上再次使用较小的签名。他们还可以计算剩余的聚合签名，并可能对除了排除之外的每笔支出进行签名。这被称为**签名减法**，并且是尽可能多地使用 `AGG_SIG_ME` 的另一个重要原因。

<details>
<summary>原文参考</summary>

- ## Replay Attacks

Another huge concern when creating your spends is whether they will be valid if parts of them are excluded or reused.
This kind of attack is the reason why `AGG_SIG_UNSAFE` is labeled the way it is.

If you sign something with `AGG_SIG_UNSAFE`, the only data that is being signed is the message you are trying to sign.
Once you sign and push it, that signature lives on the blockchain forever.
If you later create a puzzle that is locked up with the need for the same signature, an attacker can find the signature you used last time and reuse it.
This is why you should try to always use `AGG_SIG_ME` if possible.
Not only does it make you commit to the coin ID in the signature (something that is unique to every spend), but it also commits to the genesis challenge of the network you are on. A revealed signature for a coin on testnet could be replayed in mainnet otherwise.

Exclusion should also be a concern at the forefront of your mind.
Oftentimes, you will be spending multiple coins in the same bundle, and they should all be tied together into one aggregated signature.
If you have good reason not to sign one of them, make sure you know what happens if it gets excluded from the bundle.
Furthermore, aggregated signatures can't be disaggregated into smaller signatures *unless* you have previously signed one of the smaller combinations of public key-message pairs in the bundle. The attacker can exclude the rest of the transactions that contain `AGG_SIG` conditions and reuse the smaller signature again on the remaining transactions.
They can also calculate the remaining aggregated signature and perhaps sign every spend except the one the exclude. This is known as **signature subtraction** and is another great reason to use `AGG_SIG_ME` as much as possible.

</details>

## “来自上帝的闪电贷”攻击

在构建您的硬币期间还必须考虑的一个有趣的角度是，如果花费它们的一方拥有无限的钱，它们的安全性如何。这可能看起来很荒谬，除了加密货币使**闪电贷款**能够存在，这是一种无条件的即时贷款，除了它们在同一块内退还给所有者。

举个例子，一个存钱罐硬币，只有当存钱罐的数量增长到一个确定的储蓄目标时，它才允许你提取资金。如果一个人想提早取回他们的资金，他们可以借与他们的储蓄目标相等的钱，兑现存钱罐，然后归还借来的钱。

如果以编程方式计算价格，也有可能使用大量借来的资金来影响某物的价格。如果你有足够的钱，你可以单独模拟一堆交易来影响价格计算到你想要的价格，在那个价格进行交易，然后在保持利润的同时返还你借来的所有模拟交易的钱。

幸运的是，这种攻击有一个相对容易的修复方法，那就是添加一个`(ASSERT_HEIGHT_RELATIVE 1)` 条件来防止钱在同一个区块中被退回。

<details>
<summary>原文参考</summary>

- ## The "Flash Loan from God" attack

An interesting angle that also has to be considered during the building of your coins is how their security holds up if a party that is spending them has infinite money.
This may seem ridiculous except that cryptocurrency enables **flash loans** to exist which are instant loans of money with no conditions except that they are returned to the owner within the same block.

Take for example, a piggybank coin that only allows you to withdraw funds once the amount of the piggybank has grown to a determined savings goal.
If a person wants to retrieve their funds early, they can borrow money equal to their savings goal, cash out the piggybank, and then return the money that they borrowed.

There's also potential to use vast sums of borrowed money to influence the price of something, if that price is calculated programmatically.
If you have enough money, you can singularly simulate a bunch of trades to influence the price calculation to the price you desire, make a transaction at that price, and then return all of the money you borrowed to simulate trading while keeping the profits.

Fortunately, this attack has a relatively easy fix, and that is to add an `(ASSERT_HEIGHT_RELATIVE 1)` condition to prevent the money from being returned in the same block.

</details>

## 谜语和谜底揭示

记住要考虑何时揭开谜语和谜底。 它们仅在承诺给它们的代币的花费时间显示。 在此之前，网络唯一看到的是父代币和谜语哈希。 这可能是一个优势，因为您可以在揭示谜语之前隐藏敏感信息，以便在谜语哈希中花费硬币。 然而，谜语一旦被揭开，它就会永远被揭开，所以敏感信息不能再被认为是敏感的。

还要记住，如果父代币在创建它之前将信息柯里化给它的子代币，那么在子代币被花费之前，该信息将是公开的。对于某些钱包来说，这是一个优势，因为您可能需要有关硬币谜语的某些数据来计算它是否属于您。但是，如果您尝试使用纯文本密码，那将不是很安全。相反，请确保预先提交带有哈希值的内容，然后断言它们稍后会正确显示。

<details>
<summary>原文参考</summary>

- ## Puzzle and Solution Reveals

Remember to think about when puzzles and solutions are revealed.
They are revealed only at spend time of the coin that is committed to them.
The only thing that the network sees prior to that is the parent coin and the puzzle hash.
This can be an advantage, since you can hide sensitive information for spending the coin inside the puzzle hash before it is ever revealed.
However, once the puzzle is revealed, it's revealed forever, so that sensitive information cannot be considered sensitive again.

Also keep in mind that if a parent coin is currying information to its child coin before it creates it, that will be public before the child coin is spent.
For some wallets, this is an advantage since you may want certain data about a coin's puzzle to calculate whether or not it's yours.
However, if you were trying to use a plain-text password, that won't be very secure.
Instead, make sure to pre-commit to things with hashes and then assert that they are revealed correctly later.

</details>

## 密码锁定硬币安全

值得注意的是，[我们一直在构建的密码锁定币](/docs/common_functions#outer-and-inner-puzzles)其实并不是很安全。当你解开谜语时，你必须要暴露密码。因为任何您向其支付费用的完整节点现在可以看到您的密码，他们可以更改谜底并支付所有费用！

为了修复它，最好输入一个也必须为谜底签名的公钥。新的谜语将只能使用密码*并且*只能由您决定拥有此密码的人使用硬币。 当然，这在大多数情况下并不是特别有用，通常与带有额外步骤的签名锁定硬币一样好。签名是迄今为止锁定硬币的最安全方式。

<details>
<summary>原文参考</summary>

- ## Password Locked Coin Security

It's worth noting that the [password locked coin we've been building](/docs/common_functions#outer-and-inner-puzzles) is actually not very secure.
When you solve the puzzle, you have to reveal the password.
Since any full nodes whom you give your spend to will now be able to see your password, they can change the solution and pay themselves all the money instead!

In order to fix it, it's probably best to curry in a public key that also has to sign for the solution.
The new puzzle will be able to be spent only with a password *and* only by the person who you have decided owns this coin. Of course, this is not particularly useful most of the time and is usually about as good as a signature locked coin with extra steps.
Signatures are by far the most secure way to lock up your coins.

</details>

## 结论

希望您对创建 Chialisp 谜语时涉及的风险有更好的了解。 通过传递危险的解决方案或忽略交易/签名来尝试和利用您的难题是非常值得的。你不仅要防止坏人，还要防止人们不小心把他们的硬币变砖。谜语通常是永久性的，因此值得花额外的时间。

<details>
<summary>原文参考</summary>

- ## Conclusion

Hopefully you have a better idea of what risks are involved when creating a Chialisp puzzle.
It's very worth your time to try and exploit your puzzles by passing in dangerous solutions or leaving out transactions/signatures.
You're not just trying to protect against bad actors, but also against people accidentally bricking their coins.
Puzzles are usually pretty permanent, so it's worth the extra time.

</details>
