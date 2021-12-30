---
id: pooling
title: 矿池
---

> Pooling

Chia Network 进行池化的方式与之前的许多区块链不同。矿池运营商实际上依靠链上智能代币来验证他们是否能够直接索取农民创造的任何潜在矿池奖励。这使得矿池能够足够信任农民来支付他们的费用，同时仍将制作区块的权力掌握在农民手中。这意味着网络的去中心化保持不变，即使奖励集中到池中！

在本节中，我们将分解所有这些在 Chialisp 中的工作原理。本节假设您已经阅读了有关[单例硬币](https://chialisp.com/docs/puzzles/singletons) 的部分（或至少了解它们的工作原理），因为这是包含池化谜语的外部谜语。

<details>
<summary>原文参考</summary>

The way that Chia Network does pooling is unlike many blockchains that have come before it. Pool operators actually rely on an on-chain smart coin to verify that they will be able to directly claim any potential pool rewards that farmers create.
This allows pools to trust farmers enough to pay them out, while still keeping the power of making the blocks in the hands of the farmer.
This means that the decentralization of the network remains the same, even as the rewards get concentrated to the pool!

In this section, we're going to break down how all of this works in Chialisp.
This section assumes you have already read the section about [singletons](https://chialisp.com/docs/puzzles/singletons) (or at least understand how they work) as that is the outer puzzle that wraps the pooling puzzles.

</details>

## 设计要求

矿池协议在 Chia 网络上的工作方式有一些要求。现在让我们回顾一下它们：

* **农民耕种的是区块，而不是矿池。** 这对于网络去中心化非常重要。如果这不是真的，那么矿池越大，它就越接近获得 51% 的网络资源。我们仍然希望农民创建区块，但我们需要一些方法来确保农民会给矿池当他们这样做时，他们就会获得奖励。我们通过创建直接耕种特定谜语哈希的图来做到这一点，并确保谜语哈希是矿池可以声明的东西。
* **农夫必须能够更改矿池。** 最初，这似乎与第一个要求相冲突。可以制作图以将奖励直接发送到谜语哈希，但是一旦绘制，您就无法更改该谜语哈希。如果要切换矿池，则必须重新制作所有图块！我们通过让我们的图块创建特定单例硬币（也称为“图块 nft”）可以要求的付款来解决这个问题。然后，我们可以将该单例硬币的部分控制权借给一个池，但仍然保留收回我们的单例硬币并借出的能力而是转移到不同的池中。只要我们保留对单例硬币的控制，我们的图块就会保持有效。
* **农民不能立即离开矿池。** 此要求可防止其他区块链常见的攻击，在这种攻击中，矿工将向矿池发送部分区块，直到他们赢得一个区块，然后立即离开矿池并为自己声明该区块。以防止为此，我们实现了一个叫做**等候室**的谜语，它与耕种到矿池的谜语几乎相同，除了农民可以在指定的时间（以块为单位）后回收他们的单例硬币。

<details>
<summary>原文参考</summary>

- ## Design Requirements

There are a few requirements that were set for how the pooling protocol would work on the Chia Network. Let's go over them now:

* **The farmer farms the blocks, not the pool.** This is incredibly important for network decentralization. If this is not true, then the bigger a pool gets, the closer it is to gaining 51% of the network resources.
We still want the farmers to create the blocks, but we need some way to assure the pool that the farmer will give them the reward when they do.
We do this by creating plots that farm directly to a specific puzzle hash, and ensuring that puzzle hash is something that the pool can claim.
* **The farmer must be able to change pools.** Initially, this seems to conflict with the first requirement.
Plots can be made to send rewards directly to a puzzle hash, but you cannot change that puzzle hash once they are plotted.
If you want to switch pools, you'll have to remake all of your plots! We solve this by having our plots create payments that a specific singleton (also called a "plot nft") can claim.
Then, we can lend partial control of that singleton to a pool, but still retain the ability to reclaim our singleton and lend it instead to a different pool.
Our plots will remain effective as long as we retain control of the singleton.
* **The farmer cannot leave the pool immediately.** This requirement prevents attacks common to other blockchains where a miner will send partials to a pool until they win a block, then leave the pool immediately and claim that block for themselves.
To prevent this, we have implemented something called a **waiting room** puzzle which is nearly the same as the puzzle for farming to a pool, except that the farmer can reclaim their singleton after a specified amount of time (in blocks).

</details>

## 矿池成员

我们将用于借出对我们的单例硬币的部分控制权的标准谜语称为**矿池成员**内部谜语。我们的目标有三个：

* 允许矿池领取[pay-to-singleton 硬币](https://chialisp.com/docs/puzzles/singletons#pay-to-singleton)
* 禁止农民索取任何硬币
* 允许农民开始收回对单例硬币的完全控制权的过程

现在让我们来看看完整的源代码：

```chialisp
(mod (
       POOL_PUZZLE_HASH
       P2_SINGLETON_PUZZLE_HASH
       OWNER_PUBKEY
       POOL_REWARD_PREFIX
       WAITINGROOM_PUZHASH
       Truths
       p1
       pool_reward_height
     )


  ; POOL_PUZZLE_HASH is commitment to the pool's puzzle hash
  ; P2_SINGLETON_PUZZLE_HASH is the puzzle hash for your pay to singleton puzzle
  ; OWNER_PUBKEY is the farmer pubkey which authorises a travel
  ; POOL_REWARD_PREFIX is network-specific data (mainnet vs testnet) that helps determine if a coin is a pool reward
  ; WAITINGROOM_PUZHASH is the puzzle_hash you'll go to when you iniate the leaving process

  ; Absorbing money if pool_reward_height is an atom
  ; Escaping if pool_reward_height is ()

  ; p1 is pool_reward_amount if absorbing money
  ; p1 is key_value_list if escaping

  ; pool_reward_amount is the value of the coin reward - this is passed in so that this puzzle will still work after halvenings
  ; pool_reward_height is the block height that the reward was generated at. This is used to calculate the coin ID.
  ; key_value_list is signed extra data that the wallet may want to publicly announce for syncing purposes

  (include condition_codes.clib)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  (defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
    (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
  )

  (defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
    (list
        (list CREATE_COIN my_inner_puzzle_hash my_amount)
        (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
        (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
        (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
    )
  )

  (defun-inline travel_to_waitingroom (OWNER_PUBKEY WAITINGROOM_PUZHASH my_amount extra_data)
    (list (list AGG_SIG_ME OWNER_PUBKEY (sha256tree extra_data))
          (list CREATE_COIN WAITINGROOM_PUZHASH my_amount)
    )
  )

  ; main

  (if pool_reward_height
    (absorb_pool_reward POOL_PUZZLE_HASH
                        (my_inner_puzzle_hash_truth Truths)
                        (my_amount_truth Truths)
                        p1
                        (calculate_pool_reward pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
    )
    (travel_to_waitingroom OWNER_PUBKEY WAITINGROOM_PUZHASH (my_amount_truth Truths) p1)
  )
)
```

和往常一样，让我们从论点开始：

`POOL_PUZZLE_HASH` 是我们在赢得区块奖励时想要向矿池支付的地址。池使用它来验证单例硬币是否已借给他们。

`P2_SINGLETON_PUZZLE_HASH` 是支付给这个单例硬币的谜语哈希。这是农民制作他们的图块的哈希值。

`OWNER_PUBKEY` 是将签署离开矿池决定的公钥。这很可能归农民所有。

`POOL_REWARD_PREFIX` 是一个独特的参数。在 chia 主网上，这将永远是 `ccd5bb71183532bff220ba46c268991a00000000000000000000000000000000`。它是矿池奖励币 `parent_coin_id` 的开头。因为这些硬币没有父代币，所以它们的 ID 是从 ID 左半部分的网络 ID 和右侧的递增标识符的组合中得出的。前半部分后跟 32 个零，使其成为标准散列的长度。当我们回顾使用它的代码段时，我们将更多地讨论为什么需要它。

`WAITINGROOM_PUZHASH` 是等候室谜语的谜语哈希，当我们试图离开矿池时，我们将前往该谜语。我们稍后会讨论那个谜语，但重要的是要知道我们在加入矿池之前致力于我们留下的谜语。这样矿池就可以对您离开的时间有一定的保证。

`Truths` 是作为单例硬币顶层的一部分强行添加到这个难题中的数据结构。它包含我们将在谜语中使用的一些数据，我们将使用 `singleton_truths.clib` 库中的一些函数访问这些数据。

这个谜语有两个花费案例。**吸收案例** 很可能由矿池触发，并且是他们要求农民获得奖励的方式。**逃逸案例**将由养殖者发起，开始前往等候室离开矿池的过程。以下两个参数根据我们正在执行的情况而变化：

`p1` 之所以如此命名是因为它会根据我们使用的支出类型而有所不同。它是：

* 正在领取的矿池奖励金额（前三年为1750000000000）
* 用于向区块链显示重要信息以供钱包使用的键值对列表（类似于 [singleton launcher](https://chialisp.com/docs/puzzles/singletons#the-launcher)）

`pool_reward_height` 也因支出情况而异，但在转义情况下它只是 `()`，所以我们保留它的名称。在吸收的情况下，它是被吸收的矿池奖励的高度。这与`POOL_REWARD_PREFIX` 一起使用来计算奖励币的 `parent_coin_id`，以便我们可以计算其ID。

让我们谈谈我们的主要入口点：

```chialisp
(if pool_reward_height
  (absorb_pool_reward POOL_PUZZLE_HASH
                      (my_inner_puzzle_hash_truth Truths)
                      (my_amount_truth Truths)
                      p1
                      (calculate_pool_reward pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
  )
  (travel_to_waitingroom OWNER_PUBKEY WAITINGROOM_PUZHASH (my_amount_truth Truths) p1)
)
```

这里的主要控制流程是高度是否作为 `()` 传入。这就是我们确定要执行的支出类型的方式。之后，我们只需转到适当的函数，该函数会生成我们正在执行的支出类型所需的条件。您会注意到，在将 `Truths` 对象传递给内部函数之前，我们正在从它们中提取相关数据。

我们还在计算一项额外的信息。我们现在来看看：

```chialisp
(defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
  (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
)
```

这一行可能看起来很复杂，但它做的事情相当简单。它通过手动计算 `POOL_REWARD_PREFIX` 和 `pool_reward_height` 的父 ID 来计算它所声称的奖励硬币的硬币 ID。

为什么我们必须这样做？手动计算是因为这个单例硬币可能有其他付款。现在，由于我们将我们的单例硬币借给了矿池，我们不希望单例硬币能够要求这些奖励。 我们特别希望矿池能够领取由农业产生的矿池奖励。因为我们知道这些奖励有一个 `parent_coin_id`，它是一种[特殊格式](https://chialisp.com/docs/coin_lifecycle#farming-rewards)，我们可以手动计算，保证矿池不会骗我们，传入非奖励币的 ID。

让我们来看看这里的这个部分：

```chialisp
(logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height))
```

`(- (lsh (q . 1) (q . 128)) (q . 1))` 只是一种生成 32 个 `f`s 序列的方法。然后，我们将 `logand` 与池奖励高度一起使用，在诚实的情况下应该保持不变。最后，我们使用 `logior` 将两个值组合成一个字符串。假设我们的高度为 `abcdef`。我们的最终产品将是 `ccd5bb71183532bff220ba46c268991a00000000000000000000000000abcdef`。

为什么我们需要这个额外的带有 `f`s 字符串的 `logand`？这是为了防止一个相对隐蔽但可能发生的攻击。请记住，攻击者可以通过谜底传递他们想要的任何内容。例如，一个 32 字节的十六进制字符串。如果他们通过正确的十六进制字符串，他们可以完全控制我们计算的输出*除了*在 `POOL_REWARD_PREFIX`（其中62/128）的创世挑战中碰巧设置为 1 的位。这个想法是，一个矿池可以磨出一个其父 ID 设置了所有这些位的硬币，创建硬币，然后使用单例硬币来声明它。为什么要索取您已经控制的硬币？这在等候室谜语中会变得更加明显，但如果他们能够重置谜语，他们可以假设“冻结”矿池中的单身人士。它在矿池成员谜语中没有那么重要，但最好还是防止池以意外方式花费单例硬币。 `logand` 确保只有右侧的位可以更改，左侧的每个位在使用 `logior` 评估之前都设置为 0。

好的，让我们进入吸收支出案例的条件：

```chialisp
(defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
  (list
      (list CREATE_COIN my_inner_puzzle_hash my_amount)
      (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
      (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
      (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
  )
)
```

第一个 `CREATE_COIN` 条件确保我们的单例完全按原样重新创建。 请记住，我们从单例硬币外部谜语中获得了 `my_inner_puzzle_hash` 和 `my_amount`，因此我们不需要宣传它们。

下一个 `CREATE_COIN` 条件创建一个硬币，该硬币使用从花费支付到单个硬币来支付矿池的超额价值。 请记住，此谜语哈希已嵌入谜语中且无法更改。 这是因为不需要来自矿池的签名来花费硬币。 如果我们不预先提交拼图哈希，任何人都可以使用自己的谜语哈希解决并将奖励花在自己身上。

公告条件为[支付到单例硬币公告](https://chialisp.com/docs/puzzles/singletons#pay-to-singleton)的另一面。公告创建允许支付到单例硬币断言它已收到支付指令。公告宣传确保实际花费了支付给单例硬币（否则我们会再次遇到上面的单例硬币“冻结”问题）。

最后，我们来看一下逃逸条件：

```chialisp
(defun-inline travel_to_waitingroom (OWNER_PUBKEY WAITINGROOM_PUZHASH my_amount extra_data)
  (list (list AGG_SIG_ME OWNER_PUBKEY (sha256tree extra_data))
        (list CREATE_COIN WAITINGROOM_PUZHASH my_amount)
  )
)
```

足够简单。我们首先需要对通过谜底传入的键/值列表进行签名以保护它并表明此支出确实由单身人士的所有者触发。我们唯一要做的另一件事就是把自己送到候诊室。

现在让我们谈谈这个谜语。

<details>
<summary>原文参考</summary>

- ## The Pool Member

We call the standard puzzle that we use to lend away partial control of our singleton the **pool member** inner puzzle.
Our goal here is threefold:
* Allow the pool to claim [pay-to-singleton coins](https://chialisp.com/docs/puzzles/singletons#pay-to-singleton)
* Disallow the farmer from claiming any coins
* Allow the farmer to begin the process of reclaiming full control of the singleton

Let's take a look at the full source now:

```chialisp
(mod (
       POOL_PUZZLE_HASH
       P2_SINGLETON_PUZZLE_HASH
       OWNER_PUBKEY
       POOL_REWARD_PREFIX
       WAITINGROOM_PUZHASH
       Truths
       p1
       pool_reward_height
     )


  ; POOL_PUZZLE_HASH is commitment to the pool's puzzle hash
  ; P2_SINGLETON_PUZZLE_HASH is the puzzle hash for your pay to singleton puzzle
  ; OWNER_PUBKEY is the farmer pubkey which authorises a travel
  ; POOL_REWARD_PREFIX is network-specific data (mainnet vs testnet) that helps determine if a coin is a pool reward
  ; WAITINGROOM_PUZHASH is the puzzle_hash you'll go to when you iniate the leaving process

  ; Absorbing money if pool_reward_height is an atom
  ; Escaping if pool_reward_height is ()

  ; p1 is pool_reward_amount if absorbing money
  ; p1 is key_value_list if escaping

  ; pool_reward_amount is the value of the coin reward - this is passed in so that this puzzle will still work after halvenings
  ; pool_reward_height is the block height that the reward was generated at. This is used to calculate the coin ID.
  ; key_value_list is signed extra data that the wallet may want to publicly announce for syncing purposes

  (include condition_codes.clib)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  (defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
    (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
  )

  (defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
    (list
        (list CREATE_COIN my_inner_puzzle_hash my_amount)
        (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
        (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
        (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
    )
  )

  (defun-inline travel_to_waitingroom (OWNER_PUBKEY WAITINGROOM_PUZHASH my_amount extra_data)
    (list (list AGG_SIG_ME OWNER_PUBKEY (sha256tree extra_data))
          (list CREATE_COIN WAITINGROOM_PUZHASH my_amount)
    )
  )

  ; main

  (if pool_reward_height
    (absorb_pool_reward POOL_PUZZLE_HASH
                        (my_inner_puzzle_hash_truth Truths)
                        (my_amount_truth Truths)
                        p1
                        (calculate_pool_reward pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
    )
    (travel_to_waitingroom OWNER_PUBKEY WAITINGROOM_PUZHASH (my_amount_truth Truths) p1)
  )
)
```

As always, let's begin with the arguments:

`POOL_PUZZLE_HASH` is the address to which we want to pay the pool when we win a block reward.
The pool uses this to verify that the singleton has been lent to them.

`P2_SINGLETON_PUZZLE_HASH` is the puzzle hash that payments to this singleton will have.
This is the hash that farmers make their plots to.

`OWNER_PUBKEY` is the public key that will sign the decision to leave the pool.
This is likely owned by the farmer.

`POOL_REWARD_PREFIX` is a bit of a unique argument.
On the chia mainnet, this will always be `ccd5bb71183532bff220ba46c268991a00000000000000000000000000000000`.
It is the beginning of the `parent_coin_id` of pool reward coins. Because those coins do not have a parent, their ID is derived from a combination of the network ID in the left half of the ID, and an incrementing identifier in the right.
That first half is followed by 32 zeroes to make it the length of a standard hash.
We'll talk more about why this is needed when we go over the segment of code that uses it.

`WAITINGROOM_PUZHASH` is the puzzle hash of the waiting room puzzle that we will go to when we attempt to leave the pool.
We will go over that puzzle later, but what's important to know is that we commit to the puzzle we leave to before we join the pool.
This is so the pool can have certain assurances about how long it will take you to leave.

`Truths` is the data structure that is forcefully added to this puzzle as part of the singleton top layer.
It contains some data we will use in the puzzle which we will access by using some functions from the `singleton_truths.clib` library.

There are two spend cases for this puzzle.
The **absorb case** will most likely be triggered by the pool and is how they claim the rewards that the farmers receive.
The **escape case** will be initiated by the farmer to begin the process of leaving the pool by heading to the waiting room.
The following two arguments change depending on the case we are executing:

`p1` is named the way it is because it is going to be different depending on the spend type we are using.
It is either:
 * The amount of the pool reward that is being claimed (1750000000000 during the first three years)
 * A list of key value pairs that is used to reveal important information to the blockchain for wallets to use (similar to the [singleton launcher](https://chialisp.com/docs/puzzles/singletons#the-launcher))

`pool_reward_height` is also different depending on the spend case, but in the escape case it is just `()` so we leave it named the way it is.
In the absorb case, it is the height of the pool reward that is being absorbed.
This is used along with `POOL_REWARD_PREFIX` to calculate the `parent_coin_id` of the reward coin so that we can calculate its ID.

Let's talk about our main entry point:

```chialisp
(if pool_reward_height
  (absorb_pool_reward POOL_PUZZLE_HASH
                      (my_inner_puzzle_hash_truth Truths)
                      (my_amount_truth Truths)
                      p1
                      (calculate_pool_reward pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
  )
  (travel_to_waitingroom OWNER_PUBKEY WAITINGROOM_PUZHASH (my_amount_truth Truths) p1)
)
```

The main control flow here is whether or not the height is passed in as `()`. This is how we determine the spend type to execute. After that we just head to the appropriate function that generates the conditions we will need for the type of spend we are executing. You'll notice that we are extracting the relevant data from the `Truths` object before we pass them to the inner functions.

We are also calculating one additional piece of information.
Let's look at it now:

```chialisp
(defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
  (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
)
```

This line may look complex, but it's doing something fairly simplistic.
It is calculating the coin ID of the reward coin it is claiming by manually calculating the parent id from the `POOL_REWARD_PREFIX` and `pool_reward_height`.

Why do we have to do this? The manual calculation is due to the fact that this singleton may have other payments made to it.
Right now, since we are lending our singleton to the pool, we don't want the singleton to be able to claim those rewards. We specifically only want the pool to be able to claim pool rewards that are generated from farming.
Since we know these rewards have a `parent_coin_id` that is a [special format](https://chialisp.com/docs/coin_lifecycle#farming-rewards), we can manually calculate it to ensure that the pool can't lie to us and pass in the ID of a non-reward coin.

Let's take a look at this section here:

```chialisp
(logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height))
```

`(- (lsh (q . 1) (q . 128)) (q . 1))` is simply a way of generating a sequence of 32 `f`s. We then use `logand` on that with the pool reward height, which in an honest scenario should leave it unchanged.
Finally, we use `logior` to combine the two values into one string.
Let's say we have a height of `abcdef`.
Our final product will be `ccd5bb71183532bff220ba46c268991a00000000000000000000000000abcdef`.

Why do we need this extra `logand` with the string of `f`s?  It's to prevent a relatively obscure, but possible attack.
Remember that an attacker can pass whatever they want in through the solution.
For example, a 32 byte hex string.
If they passed through the right hex string, they could completely control the output of our calculation *except* for the bits that happen to be set to 1 in the genesis challenge half of the `POOL_REWARD_PREFIX` (62/128 of them).
The idea is that a pool could grind out a coin whose parent ID has all of those bits set, create the coin, and then use the singleton to claim it.
Why claim a coin that you already had control of? This will become more apparent in the waiting room puzzle, but they could hypothetically "freeze" the singleton in their pool if they were able to reset the puzzle.
It's not as important in the pool member puzzle, but it is still probably best to prevent the pool from spending the singleton in an unintended way.
The `logand` ensures that only bits on the right can change, every bit on left gets set to 0 before it is evaluated with the `logior`.

Okay let's move onto the conditions for the absorb spend case:

```chialisp
(defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
  (list
      (list CREATE_COIN my_inner_puzzle_hash my_amount)
      (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
      (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
      (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
  )
)
```

The first `CREATE_COIN` condition ensures that our singleton is recreated exactly as it is. Remember that we get `my_inner_puzzle_hash` and `my_amount` from our singleton outer puzzle, so we don't need to assert them.

The next `CREATE_COIN` condition creates the coin that uses the excess value from spending the pay-to-singleton coin to pay the pool.
Keep in mind that this puzzle hash is curried into the puzzle and cannot change.
This is because there is no signature required from the pool to spend the coin.
If we didn't pre-commit to the puzzle hash, anyone could solve with their own puzzle hash and spend the rewards to themselves.

The announcement conditions are the other side of the [pay-to-singleton announcements](https://chialisp.com/docs/puzzles/singletons#pay-to-singleton). The announcement creation allows the pay-to-singleton to assert that it has received the instruction to pay out.
The announcement assertion ensures that the pay-to-singleton is actually spent (otherwise we run into the singleton "freezing" problem from above again).

Finally, let's look at the escape conditions:

```chialisp
(defun-inline travel_to_waitingroom (OWNER_PUBKEY WAITINGROOM_PUZHASH my_amount extra_data)
  (list (list AGG_SIG_ME OWNER_PUBKEY (sha256tree extra_data))
        (list CREATE_COIN WAITINGROOM_PUZHASH my_amount)
  )
)
```

Simple enough. We first require a signature on the key/value list that is being passed in through the solution to secure it and to signal that this spend is indeed triggered by the owner of the singleton. The only other thing we do is send ourselves to the waiting room.

Let's talk about that puzzle now.

</details>

## 候车室

在我们开始讨论这个谜语之前，让我们像往常一样检查完整的来源：

```chialisp
(mod (
        POOL_PUZZLE_HASH
        P2_SINGLETON_PUZZLE_HASH
        OWNER_PUBKEY
        POOL_REWARD_PREFIX
        RELATIVE_LOCK_HEIGHT
        Truths
        spend_type
        p1
        p2
      )

  ; POOL_PUZZLE_HASH is commitment to the pool's puzzle hash
  ; P2_SINGLETON_PUZZLE_HASH is the puzzlehash for your pay_to_singleton puzzle
  ; OWNER_PUBKEY is the farmer pubkey which signs the exit puzzle_hash
  ; POOL_REWARD_PREFIX is network-specific data (mainnet vs testnet) that helps determine if a coin is a pool reward
  ; RELATIVE_LOCK_HEIGHT is how long it takes to leave

  ; spend_type is: 0 for absorbing money, 1 to escape
  ; if spend_type is 0
    ; p1 is pool_reward_amount - the value of the coin reward - this is passed in so that this puzzle will still work after halvenings
    ; p2 is pool_reward_height - the block height that the reward was generated at. This is used to calculate the coin ID.
  ; if spend_type is 1
    ; p1 is key_value_list - signed extra data that the wallet may want to publicly announce for syncing purposes
    ; p2 is destination_puzhash - the location that the escape spend wants to create itself to

  (include condition_codes.clvm)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  (defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
    (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
  )

  (defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
    (list
        (list CREATE_COIN my_inner_puzzle_hash my_amount)
        (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
        (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
        (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
    )
  )

  (defun-inline travel_spend (RELATIVE_LOCK_HEIGHT new_puzzle_hash my_amount extra_data)
    (list (list ASSERT_HEIGHT_RELATIVE RELATIVE_LOCK_HEIGHT)
          (list CREATE_COIN new_puzzle_hash my_amount)
          (list AGG_SIG_ME OWNER_PUBKEY (sha256tree (list new_puzzle_hash extra_data)))
    )
  )

  ; main

  (if spend_type
    (travel_spend RELATIVE_LOCK_HEIGHT p2 (my_amount_truth Truths) p1)
    (absorb_pool_reward POOL_PUZZLE_HASH
                        (my_inner_puzzle_hash_truth Truths)
                        (my_amount_truth Truths)
                        p1
                        (calculate_pool_reward p2 P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
    )
  )
)
```

您可能会注意到它看起来与上面的几乎相同。我们不会将其逐一分解，而是将重点放在差异上。首先，参数：

```chialisp
(
  POOL_PUZZLE_HASH
  P2_SINGLETON_PUZZLE_HASH
  OWNER_PUBKEY
  POOL_REWARD_PREFIX
  RELATIVE_LOCK_HEIGHT
  Truths
  spend_type
  p1
  p2
)
```

我们还有`POOL_PUZZLE_HASH`、`P2_SINGLETON_PUZZLE_HASH`、`OWNER_PUBKEY`和`POOL_REWARD_PREFIX`。但是，现在我们还有一个新的咖喱参数叫做`RELATIVE_LOCK_HEIGHT`。这表示进入这个等候室后，我们必须等待的时间才能花在其他事情上。

请注意，相对锁定高度是从创建硬币的时间开始计算的。如果硬币作为吸收的一部分被花费，则锁定高度会重置。理论上，如果锁定高度足够大，并且经常获胜的农民足够大，这个单例硬币可以“冻结”，直到农民足够幸运没有在指定的时间范围内赢得区块。

我们还有`Truths`，因为这仍然是一个单例硬币内部谜语。然而，最后三个参数几乎完全不同。

`spend_type` 现在是必要的，因为我们不能再根据最后一个参数选择我们正在执行的支出案例。

`p1` 因支出类型而异：
* 如果我们进行的是吸收支出，则是矿池奖励硬币的数量。
* 如果我们要前往一个新的谜语，它是我们用来在区块链中记录数据的键/值列表。

`p2` 也因支出类型而异：
* 如果我们进行的是吸收支出，则是创建矿池奖励代币的高度。
* 如果我们要去一个新的谜语，那就是那个谜语的谜语哈希

在参数差异之后，唯一的另一个主要变化是旅行函数：

```chialisp
(defun-inline travel_spend (RELATIVE_LOCK_HEIGHT new_puzzle_hash my_amount extra_data)
  (list (list ASSERT_HEIGHT_RELATIVE RELATIVE_LOCK_HEIGHT)
        (list CREATE_COIN new_puzzle_hash my_amount)
        (list AGG_SIG_ME OWNER_PUBKEY (sha256tree (list new_puzzle_hash extra_data)))
  )
)
```

首先，我们断言所需的时间已经过去（这是为了池的安全）。然后，我们创建新的指定硬币。请注意，我们无法选择数量，因此必须转义到新的单例硬币。

最后，我们像以前一样对额外的数据进行签名，但这次我们包含了 `destination_puzhash`，这样恶意的人就无法通过替换一个值来窃取我们的单例硬币。

这些是唯一的变化！

将这些作为单独但相似的谜题可能看起来很不规则。有很多代码重复，这通常是不好的做法。很有可能有办法将这两个谜语组合成一个谜语，但成本会急剧增加。归根结底，拥有重复的代码比让区块链陷入不必要的昂贵交易要好。

有趣的是，当我们自己池化时，我们实际上希望处于锁高度为零的等候室状态。理由是，由于我们将单例硬币“借”给自己，我们真的不需要提前离开的保险，这就是候诊室的全部意义所在。这减少了不必要的支出，从长远来看为我们节省了更多的成本。


<details>
<summary>原文参考</summary>

- ## The Waiting Room

Before we start talking about this puzzle, let's examine the full source as usual:

```chialisp
(mod (
        POOL_PUZZLE_HASH
        P2_SINGLETON_PUZZLE_HASH
        OWNER_PUBKEY
        POOL_REWARD_PREFIX
        RELATIVE_LOCK_HEIGHT
        Truths
        spend_type
        p1
        p2
      )

  ; POOL_PUZZLE_HASH is commitment to the pool's puzzle hash
  ; P2_SINGLETON_PUZZLE_HASH is the puzzlehash for your pay_to_singleton puzzle
  ; OWNER_PUBKEY is the farmer pubkey which signs the exit puzzle_hash
  ; POOL_REWARD_PREFIX is network-specific data (mainnet vs testnet) that helps determine if a coin is a pool reward
  ; RELATIVE_LOCK_HEIGHT is how long it takes to leave

  ; spend_type is: 0 for absorbing money, 1 to escape
  ; if spend_type is 0
    ; p1 is pool_reward_amount - the value of the coin reward - this is passed in so that this puzzle will still work after halvenings
    ; p2 is pool_reward_height - the block height that the reward was generated at. This is used to calculate the coin ID.
  ; if spend_type is 1
    ; p1 is key_value_list - signed extra data that the wallet may want to publicly announce for syncing purposes
    ; p2 is destination_puzhash - the location that the escape spend wants to create itself to

  (include condition_codes.clvm)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  (defun-inline calculate_pool_reward (pool_reward_height P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX pool_reward_amount)
    (sha256 (logior POOL_REWARD_PREFIX (logand (- (lsh (q . 1) (q . 128)) (q . 1)) pool_reward_height)) P2_SINGLETON_PUZZLE_HASH pool_reward_amount)
  )

  (defun absorb_pool_reward (POOL_PUZZLE_HASH my_inner_puzzle_hash my_amount pool_reward_amount pool_reward_id)
    (list
        (list CREATE_COIN my_inner_puzzle_hash my_amount)
        (list CREATE_COIN POOL_PUZZLE_HASH pool_reward_amount)
        (list CREATE_PUZZLE_ANNOUNCEMENT pool_reward_id)
        (list ASSERT_COIN_ANNOUNCEMENT (sha256 pool_reward_id '$'))
    )
  )

  (defun-inline travel_spend (RELATIVE_LOCK_HEIGHT new_puzzle_hash my_amount extra_data)
    (list (list ASSERT_HEIGHT_RELATIVE RELATIVE_LOCK_HEIGHT)
          (list CREATE_COIN new_puzzle_hash my_amount)
          (list AGG_SIG_ME OWNER_PUBKEY (sha256tree (list new_puzzle_hash extra_data)))
    )
  )

  ; main

  (if spend_type
    (travel_spend RELATIVE_LOCK_HEIGHT p2 (my_amount_truth Truths) p1)
    (absorb_pool_reward POOL_PUZZLE_HASH
                        (my_inner_puzzle_hash_truth Truths)
                        (my_amount_truth Truths)
                        p1
                        (calculate_pool_reward p2 P2_SINGLETON_PUZZLE_HASH POOL_REWARD_PREFIX p1)
    )
  )
)
```

You may notice that is looks nearly identical to the one above.
Instead of breaking this down piece by piece, we're just going to focus on the differences.
First, the parameters:

```chialisp
(
  POOL_PUZZLE_HASH
  P2_SINGLETON_PUZZLE_HASH
  OWNER_PUBKEY
  POOL_REWARD_PREFIX
  RELATIVE_LOCK_HEIGHT
  Truths
  spend_type
  p1
  p2
)
```

We still have `POOL_PUZZLE_HASH`, `P2_SINGLETON_PUZZLE_HASH`, `OWNER_PUBKEY`, and `POOL_REWARD_PREFIX`.
However, now we also have a new curried in parameter called `RELATIVE_LOCK_HEIGHT`. This indicates the amount of time after entering this waiting room that we have to wait before we can spend away to something else.

Note that relative lock heights are calculated from the time of the coin's creation.
If the coin is spent as part of an absorb, this lock height resets.
Theoretically, with a large enough lock height and a big enough farmer who wins frequently, this singleton can be "frozen" until the farmer is lucky enough not to win a block within the specified timeframe.

We still have `Truths`, since this is still a singleton inner puzzle.
However, the final three arguments are almost completely different.

`spend_type` is now necessary because we can no longer choose which spend case we are executing based on the last argument.

`p1` is different based on the spend type:
* If we're doing the absorb spend, it's the amount of the pool reward coin.
* If we're traveling to a new puzzle, it's the key/value list that we use to record data in the blockchain.

`p2` is also different based on the spend type:
* If we're doing the absorb spend, it's the height at which the pool reward coin was created.
* If we're traveling to a new puzzle, it's the puzzle hash of that puzzle

After the argument differences, the only other major change is in the travel function:

```chialisp
(defun-inline travel_spend (RELATIVE_LOCK_HEIGHT new_puzzle_hash my_amount extra_data)
  (list (list ASSERT_HEIGHT_RELATIVE RELATIVE_LOCK_HEIGHT)
        (list CREATE_COIN new_puzzle_hash my_amount)
        (list AGG_SIG_ME OWNER_PUBKEY (sha256tree (list new_puzzle_hash extra_data)))
  )
)
```

First, we assert that the required amount of time has passed (this is for the pool's safety).
Then, we create the new specified coin.
Note that we don't get to choose the amount and, therefore, must escape to a new singleton.

Finally, we sign the extra data, just like before, but this time we include the `destination_puzhash` so that someone malicious cannot steal our singleton by substituting in a value.

Those are the only changes!

It may seem pretty irregular to have these as separate but similar puzzles.
There's a lot of code duplication which is usually bad practice.
It's very possible that there is a way to combine these two puzzles into a single puzzle, but the cost would increase dramatically.
At the end of the day, it's better to have duplicated code than to bog down the blockchain with needlessly expensive transactions.

Interestingly enough, we actually want to be in the waiting room state with a lock height of zero when we are pooling by ourselves.
The reasoning is that since we are "lending" the singleton to ourselves, we don't really need the insurance that we leave early, which is the entire point of the waiting room.
This cuts down on unnecessary spends, and saves us even more cost in the long run.

</details>

