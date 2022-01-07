---
id: singletons
title: 单例硬币
---

> Singletons

Chia 生态系统中最重要的谜语之一是 **单例硬币**。这个谜语确保任何人看到它都可以看到它具有其他硬币没有的唯一 ID。各方可以决定接受消息或承诺来自该唯一 ID 并保证控制单例硬币的一方不会双重浸渍或冒充其他人。

这个谜语是一个[外部谜语](/docs/common_functions)，用于包装池化谜语、NFT 和去中心化身份。如果需要独特性，任何内部谜语都可以用这个谜语包裹。

<details>
<summary>原文参考</summary>

One of the most important puzzles in the Chia ecosystem is the **singleton**.
This is a puzzle that assures anyone who looks at can see that it has a unique ID that no other coin has.
Parties can decide to accept messages or commitments from that unique ID with the assurance that the party who controls the singleton is not double dipping or impersonating someone else.

This puzzle is an [outer puzzle](/docs/common_functions) and is used to wrap pooling puzzles, NFTs, and decentralized identities.
Any inner puzzle can be wrapped with this puzzle if it has necessity for uniqueness.

</details>

## 设计选择

在创建这个谜语时做出了一些设计选择，所以现在让我们回顾一下：

* **单例硬币总是奇数。** 为了确保单例不会复制自己，它需要某种方式来验证其子实例不包含多个新的单例硬币。它通过验证只有一个孩子是奇数来做到这一点的。它要么是一个新的单例硬币，要么是 **熔体值**，将被忽略（稍后会详细介绍）。之所以选择奇数，是因为您有时可能希望单例硬币创建其他非单例硬币（例如 DID 的 0 数量消息），而让单例硬币为奇数只会使这更容易。硬币可以是 10 的倍数，因此您可以发送完整的 XCH，而不是 XCH 和 mojo。无论您从奇数中减去多少偶数，最终结果始终是奇数。
* **单例硬币总是包裹着他们奇怪的孩子。** 这将一些单例硬币的功能从内部的谜语中抽象出来。如果一个内部谜语创造了一个奇怪的硬币，它不必担心使它成为一个单例硬币，[外部谜语会处理这个](https://chialisp.com/docs/common_functions/#outer-and-inner-puzzles)。它还可以防止内部谜语因忘记包装其奇数输出而意外熔化单例硬币。
* **一个特定的魔法熔体值决定了单例硬币是否包裹了它的孩子。** 如果你想销毁一个单例硬币并使用它的数量来创建一个新的非单例硬币，你需要输出一个 `CREATE_COIN` 条件，使用金额 `-113`。当单例硬币外部谜语看到该条件时，它会将其过滤掉。这个数量是任意选择的并且是负的，因为负数量的硬币不存在。因此，这不太可能是内部谜语的意外输出。选择此选项是为了防止内部谜语因忘记创建奇数输出而意外熔化单例硬币。它必须故意指定熔化单例硬币。请记住，熔化值*确实*算作奇数输出，但被忽略，因此需要燃烧一个硬币价值才能熔化（所有输出必须是偶数）。通常无论如何都会收取交易费用，因此它可能只是其中的一部分。你还应该警惕你给予你的单例硬币的部分控制权。如果他们可以自由地创造条件，他们就有可能违背你的意愿熔化你的单例硬币。

<details>
<summary>原文参考</summary>

- ## Design choices

A few design choices were made in the creation of this puzzle so let's go over them now:

* **Singletons are always odd.**  In order to assure that a singleton does not duplicate itself, it needs some way to verify that its children do not consist of more than one new singleton.
It does this by verifying that only one of its children is odd.
It is either a new singleton, or it is the **melt value** and will be ignored (more on that later).
Odd was chosen over even because you may want to have singletons create other, non-singleton coins at times (like 0 amount messages for the DID) and having the singleton be odd just makes this easier.
Coins can be multiples of 10 so you can send a full XCH rather than an XCH and a mojo.
No matter how many even amounts you subtract from an odd amount, the end result will always be odd.
* **Singletons always wrap their odd child.** This abstracts some of the singleton functionality away from the inner puzzles.
If an inner puzzle creates an odd coin, it doesn't have to worry about making it a singleton, the [outer puzzle will take care of that](https://chialisp.com/docs/common_functions/#outer-and-inner-puzzles).
It also prevents an inner puzzle from accidentally melting the singleton by forgetting to wrap its odd output.
* **A specific magic melt value determines whether the singleton wraps its child.** If you would like to destroy a singleton and use its amount to create a new non-singleton coin, you need to output a `CREATE_COIN` condition that uses the amount `-113`.
When the singleton outer puzzle sees that condition, it filters it out.
This amount was arbitrarily chosen and is negative because a coin with negative amount cannot exist. Therefore, it is an unlikely accidental output of an inner puzzle.
This is chosen to prevent an inner puzzle from accidentally melting a singleton by forgetting to create an odd output.
It must deliberately specify to melt the singleton.
Keep in mind that the melt value *does* count as the odd output, but is ignored, creating the need to burn one mojo of coin value in order to melt (all of the outputs must be even).
Usually there will be a transaction fee anyways so it likely just becomes part of that.
You should also be wary of the amount of control you give to people who you partially lend your singleton to.
If they can freely create conditions, it may be possible for them to melt your singleton against your wishes.

</details>

## 启动器

我们需要确保只创建一个具有相同 ID 的单例硬币，这出奇的困难。问题的关键是我们无法控制创建单例硬币。我们可以依赖于 id 中的柯里化，但对于某人来说，通过以相同的 id 柯里化来创建完全相同的单例是很容易的。相反，我们可以使用对所有后代都是唯一的父代币 ID，但是，您仍然可以在第一次支出中创建多个单例硬币。

这在技术上可以通过爬上链并检查第一个非单例硬币以查看它是否有多个子单例硬币来检测，但这是低效的，我们希望我们的所有逻辑都包含在谜语中。

相反，我们可以使用的是一个**启动器**，它是一个特定的谜语，它只做一件事：创建一个单例硬币。然后，我们需要将这个启动器谜语哈希放入我们的单例硬币中，并让第一个单例硬币宣称它实际上来自一个其谜语哈希是发射器谜语哈希的父级。然后，当人们查看我们的单例硬币时，他们可以看到启动器谜语哈希是他们所知道的仅创建一个单例硬币的谜语哈希。他们不需要回到原来的父代那里去验证，因为单例硬币谜语从一开始就解决了这个问题！

那么启动器是什么样子的呢？这是来源：


```chialisp
(mod (singleton_full_puzzle_hash amount key_value_list)

  (include condition_codes.clvm)

  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ; main
  (list (list CREATE_COIN singleton_full_puzzle_hash amount)
        (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list singleton_full_puzzle_hash amount key_value_list))))
)
```

基本上是两行，所以还不错吧？ 您可能会注意到的第一件事是我们不将任何东西加进去。我们实际上不能将任何东西加进去，因为我们希望这个谜语哈希在所有单例硬币中保持不变。这样，即使有人不熟悉我们，他们也知道如果我们来自这个特定的启动器谜语哈希，我们可以相信是一个独特的硬币。

大多数情况下，您只需输入 `CREATE_COIN` 参数，谜语就会为您创建单例硬币。棘手的部分是公告的创建。由于这些参数没有被输入，我们需要以某种方式使它们免受恶意全节点的操纵。我们不能用公钥来签署它们，否则我们的谜语哈希不再是静态的。我们对这个难题的谜底是从这个难题中创建一个公告，其父在同一块中宣称。通常，父代币将成为标准硬币。在标准硬币中，我们在构成条件的谜语上签名。如果我们创建一个 `ASSERT_COIN_ANNOUNCEMENT` 条件，我们也会隐式签名。这意味着我们可以通过声明此公告来隐式签署所有启动器谜底值。如果这些值中的任何一个发生更改，则创建启动器的硬币将失败，因此将永远不会创建启动器！

最后要注意的是看似无用的 `key_value_list`，它作为参数传入并宣布。这样做的目的是向区块链观察者传达信息。有时，您希望能够在谜语揭晓之前了解有关谜语的信息。我们可以在链上获取此信息的唯一方法是从父谜语的揭示中获取，因此有时将无用参数作为谜底的一部分很有用，以便更轻松地跟踪谜语的链上状态。请记住，您需要为每个字节支付费用，因此请保持简洁。

<details>
<summary>原文参考</summary>

- ## The Launcher

We need to ensure that only one singleton is created with the same ID.
This is surprisingly difficult.
The crux of the issue is that we have no control over the coin that creates the singleton.
We could rely on a curried in id, but it's easy enough for someone to create the exact same singleton by currying in the same id.
Instead, we could use the parent coin ID which would be unique to all of its descendants, however, you could still create multiple singletons in that first spend.

This is technically detectable by crawling up the chain and checking the first non-singleton coin to see if it had multiple singleton children, but this is inefficient and we would like all of our logic to be contained to the puzzles.

Instead, what we can use is a **launcher** which is a specific puzzle that does exactly one thing: create a single singleton.
We then need to curry this launcher puzzle hash into our singleton and have the first singleton assert that it, in fact, came from a parent whose puzzle hash was the launcher puzzle hash.
Then, when people look at our singleton, they can see that the launcher puzzle hash is the hash of what they know to be a puzzle that creates only one singleton.
They don't need to go back to the original parent and verify because the singleton puzzle takes care of that right from the start!

So what does the launcher look like?  Here's the source:

```chialisp
(mod (singleton_full_puzzle_hash amount key_value_list)

  (include condition_codes.clvm)

  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ; main
  (list (list CREATE_COIN singleton_full_puzzle_hash amount)
        (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list singleton_full_puzzle_hash amount key_value_list))))
)
```

Essentially two lines, so not too bad right?  One of the first things you may notice is that we don't curry anything in.
We actually cannot curry anything in because we want this puzzle hash to be constant among all singletons.
That way, even if someone isn't familiar with us, they know that if we came from this specific launcher puzzle hash, we can be trusted to be a unique singleton.

For the most part, you simply put in `CREATE_COIN` parameters and the puzzle creates the singleton for you.
The tricky part is the announcement creation.
Since these parameters are not curried in, we somehow need them to be immune from the manipulations of malicious full nodes.
We cannot curry in a pubkey to sign them, or else our puzzle hash is no longer static. Our solution to this conundrum is to create an announcement from this puzzle that its parent asserts in the same block.
Usually, the parent is going to be a standard coin. In the standard coin, we sign the puzzle that makes the conditions.
If we create an `ASSERT_COIN_ANNOUNCEMENT` condition, we implicitly sign that too. That means we can implicitly sign all of the launcher solution values through asserting this announcement.
If any of those values are changed, the coin that creates the launcher will fail and thus the launcher will never be created!

The last thing to note is the seemingly useless `key_value_list` that is passed in as an argument and announced.
The purpose for this is to communicate information to blockchain observers.
Sometimes you want to be able to know information about a puzzle before it is revealed.
The only way we can get this information on chain is from the parent's puzzle reveal so sometimes it is useful to have useless parameters be part of the solution in order to make it easier to follow the puzzle's on chain state.
Remember that you pay cost for every byte though so keep it concise.

</details>

## 单例硬币顶层

这是完整的来源，我们将在之后分解它：

```chialisp
(mod (
       SINGLETON_STRUCT
       INNER_PUZZLE
       lineage_proof
       my_amount
       inner_solution
     )

     ;; SINGLETON_STRUCT = (MOD_HASH . (LAUNCHER_ID . LAUNCHER_PUZZLE_HASH))

     ; SINGLETON_STRUCT, INNER_PUZZLE are curried in by the wallet

  ; This puzzle is a wrapper around an inner smart puzzle which guarantees uniqueness.
  ; It takes its singleton identity from a coin with a launcher puzzle which guarantees that it is unique.

  (include condition_codes.clib)
  (include curry-and-treehash.clib)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ; "assert" is a macro that wraps repeated instances of "if"
  ; usage: (assert A0 A1 ... An R)
  ; all of A0, A1, ... An must evaluate to non-null, or an exception is raised
  ; return the value of R (if we get that far)

  (defmacro assert items
    (if (r items)
        (list if (f items) (c assert (r items)) (q . (x)))
        (f items)
    )
  )

  (defun-inline mod_hash_for_singleton_struct (SINGLETON_STRUCT) (f SINGLETON_STRUCT))
  (defun-inline launcher_id_for_singleton_struct (SINGLETON_STRUCT) (f (r SINGLETON_STRUCT)))
  (defun-inline launcher_puzzle_hash_for_singleton_struct (SINGLETON_STRUCT) (r (r SINGLETON_STRUCT)))

  ;; return the full puzzlehash for a singleton with the innerpuzzle curried in
  ; puzzle-hash-of-curried-function is imported from curry-and-treehash.clinc
  (defun-inline calculate_full_puzzle_hash (SINGLETON_STRUCT inner_puzzle_hash)
     (puzzle-hash-of-curried-function (mod_hash_for_singleton_struct SINGLETON_STRUCT)
                                      inner_puzzle_hash
                                      (sha256tree SINGLETON_STRUCT)
     )
  )

  ; assembles information from the solution to create our own full ID including asserting our parent is a singleton
  (defun create_my_ID (SINGLETON_STRUCT full_puzzle_hash parent_parent parent_inner_puzzle_hash parent_amount my_amount)
    (sha256 (sha256 parent_parent (calculate_full_puzzle_hash SINGLETON_STRUCT parent_inner_puzzle_hash) parent_amount)
            full_puzzle_hash
            my_amount)
  )

  ;; take a boolean and a non-empty list of conditions
  ;; strip off the first condition if a boolean is set
  ;; this is used to remove `(CREATE_COIN xxx -113)`
  ;; pretty sneaky, eh?
  (defun strip_first_condition_if (boolean condition_list)
    (if boolean
      (r condition_list)
      condition_list
    )
  )

  (defun-inline morph_condition (condition SINGLETON_STRUCT)
    (list (f condition) (calculate_full_puzzle_hash SINGLETON_STRUCT (f (r condition))) (f (r (r condition))))
  )

  ;; return the value of the coin created if this is a `CREATE_COIN` condition, or 0 otherwise
  (defun-inline created_coin_value_or_0 (condition)
    (if (= (f condition) CREATE_COIN)
        (f (r (r condition)))
        0
    )
  )

  ;; Returns a (bool . bool)
  (defun odd_cons_m113 (output_amount)
    (c
      (= (logand output_amount 1) 1) ;; is it odd?
      (= output_amount -113) ;; is it the escape value?
    )
  )

  ; Assert exactly one output with odd value exists - ignore it if value is -113

  ;; this function iterates over the output conditions from the inner puzzle & solution
  ;; and both checks that exactly one unique singleton child is created (with odd valued output),
  ;; and wraps the inner puzzle with this same singleton wrapper puzzle
  ;;
  ;; The special case where the output value is -113 means a child singleton is intentionally
  ;; *NOT* being created, thus forever ending this singleton's existence

  (defun check_and_morph_conditions_for_singleton (SINGLETON_STRUCT conditions has_odd_output_been_found)
      (if conditions
        (morph_next_condition SINGLETON_STRUCT conditions has_odd_output_been_found (odd_cons_m113 (created_coin_value_or_0 (f conditions))))
        (if has_odd_output_been_found
            0
            (x)  ;; no odd output found
        )
      )
   )

   ;; a continuation of `check_and_morph_conditions_for_singleton` with booleans `is_output_odd` and `is_output_m113`
   ;; precalculated
   (defun morph_next_condition (SINGLETON_STRUCT conditions has_odd_output_been_found (is_output_odd . is_output_m113))
       (assert
          (not (all is_output_odd has_odd_output_been_found))
          (strip_first_condition_if
             is_output_m113
             (c (if is_output_odd
                    (morph_condition (f conditions) SINGLETON_STRUCT)
                    (f conditions)
                )
                (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (r conditions) (any is_output_odd has_odd_output_been_found))
             )
          )
      )
   )

  ; this final stager asserts our ID
  ; it also runs the innerpuz with the innersolution with the "truths" added
  ; it then passes that output conditions from the innerpuz to the morph conditions function
  (defun stager_three (SINGLETON_STRUCT lineage_proof my_id full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (c (list ASSERT_MY_COIN_ID my_id) (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (a INNER_PUZZLE (c (truth_data_to_truth_struct my_id full_puzhash innerpuzhash my_amount lineage_proof SINGLETON_STRUCT) inner_solution)) 0))
  )

  ; this checks whether we are an eve spend or not and calculates our full coin ID appropriately and passes it on to the final stager
  ; if we are the eve spend it also adds the additional checks that our parent's puzzle is the standard launcher format and that out parent ID is the same as our singleton ID

  (defun stager_two (SINGLETON_STRUCT lineage_proof full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (stager_three
      SINGLETON_STRUCT
      lineage_proof
      (if (is_not_eve_proof lineage_proof)
          (create_my_ID
            SINGLETON_STRUCT
            full_puzhash
            (parent_info_for_lineage_proof lineage_proof)
            (puzzle_hash_for_lineage_proof lineage_proof)
            (amount_for_lineage_proof lineage_proof)
            my_amount
          )
          (if (=
                (launcher_id_for_singleton_struct SINGLETON_STRUCT)
                (sha256 (parent_info_for_eve_proof lineage_proof) (launcher_puzzle_hash_for_singleton_struct SINGLETON_STRUCT) (amount_for_eve_proof lineage_proof))
              )
              (sha256 (launcher_id_for_singleton_struct SINGLETON_STRUCT) full_puzhash my_amount)
              (x)
          )
      )
      full_puzhash
      innerpuzhash
      my_amount
      INNER_PUZZLE
      inner_solution
    )
  )

  ; this calculates our current full puzzle hash and passes it to stager two
  (defun stager_one (SINGLETON_STRUCT lineage_proof my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (stager_two SINGLETON_STRUCT lineage_proof (calculate_full_puzzle_hash SINGLETON_STRUCT my_innerpuzhash) my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
  )


  ; main

  ; if our value is not an odd amount then we are invalid
  ; this calculates my_innerpuzhash and passes all values to stager_one
  (if (logand my_amount 1)
    (stager_one SINGLETON_STRUCT lineage_proof (sha256tree INNER_PUZZLE) my_amount INNER_PUZZLE inner_solution)
    (x)
  )
)
```

不是很多吗？让我们从论点开始：

```chialisp
(
  SINGLETON_STRUCT
  INNER_PUZZLE
  lineage_proof
  my_amount
  inner_solution
)
```

`SINGLETON_STRUCT` 是三样东西的集合：
* 该模块的树哈希
* 启动币 ID（这作为单例硬币的唯一ID）
* 启动器谜语哈希

它们被组合成一个结构的原因是因为它们几乎通过每个函数传递。 如果它们作为单个变量传递，直到解构它们，它会增加可读性和优化。

`INNER_PUZZLE` 是这个包装谜语的内部谜语。

`lineage_proof` 采用以下两种格式之一：

* `（parent_parent_coin_info parent_inner_puzzle_hash parent_amount）`
* `（parent_parent_coin_info parent_amount）` 您可能想知道，鉴于相似性，为什么不使用第一种格式？ 我们使用单独的格式是因为我们使用结构的长度来提示我们这是否是 **eve 花费**。eve 花费是单例创建后的第一次花费。我们使用这个谱系证明验证我们的父项是单例硬币。然而，在第一次花费中，父项不是单例硬币，我们实际上执行了不同的路径，我们验证我们的父项是单例硬币启动器。

`my_amount` 是花费的硬币数量，将通过 ASSERT_MY_COIN_ID 隐式声明。

`inner_solution` 是内部谜语的谜底。

接下来，让我们看看我们的主要入口点：

```chialisp
(if (logand my_amount 1)
  (stager_one SINGLETON_STRUCT lineage_proof (sha256tree INNER_PUZZLE) my_amount INNER_PUZZLE inner_solution)
  (x)
)
```

这里的控制流程非常简单。如果我们不是奇数，我们就加注，如果是，我们将所有东西都传递到下一个阶段（加上内部谜语的额外哈希值）。需要注意的一件小事是，单例实际上可以是偶数，但永远无法使用。要么这个人输入真实的数量，谜语就会增加，或者他们输入一个虚假的数量，ASSERT_MY_ID 将失败。如果攻击者要启动一个偶数单例或创建一个作为该子单例硬币的偶数的单例硬币，它会成功，但会永远卡住。

```chialisp
(defun stager_one (SINGLETON_STRUCT lineage_proof my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (stager_two SINGLETON_STRUCT lineage_proof (calculate_full_puzzle_hash SINGLETON_STRUCT my_innerpuzhash) my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
)
```

我们现在继续讨论几个“stagers”中的第一个。这些函数的目的是计算多次使用一次的值。在下一阶段，我们将使用完整的谜语哈希三次，因此最好计算一次并将其传递给下一个函数。

```chialisp
(defun stager_two (SINGLETON_STRUCT lineage_proof full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (stager_three
    SINGLETON_STRUCT
    lineage_proof
    (if (is_not_eve_proof lineage_proof)
        (create_my_ID
          SINGLETON_STRUCT
          full_puzhash
          (parent_info_for_lineage_proof lineage_proof)
          (puzzle_hash_for_lineage_proof lineage_proof)
          (amount_for_lineage_proof lineage_proof)
          my_amount
        )
        (if (=
              (launcher_id_for_singleton_struct SINGLETON_STRUCT)
              (sha256 (parent_info_for_eve_proof lineage_proof) (launcher_puzzle_hash_for_singleton_struct SINGLETON_STRUCT) (amount_for_eve_proof lineage_proof))
            )
            (sha256 (launcher_id_for_singleton_struct SINGLETON_STRUCT) full_puzhash my_amount)
            (x)
        )
    )
    full_puzhash
    innerpuzhash
    my_amount
    INNER_PUZZLE
    inner_solution
  )
)
```

这个阶段看起来很多，但实际上它所做的只是计算下一个要使用的函数的当前硬币 ID。在我们开始查看之前请注意，谱系证明经常被传递给不属于此文件的函数。这些是我们将在下一阶段讨论的 `singleton_truths.clib` 库的一部分。现在，只要知道它正在从谱系证明中访问正确的值，并且比编写诸如 `(f (r lineage_proof)) (f (r (r lineage_proof)))` 之类的东西要干净得多，而没有指示它们是什么意思是。

第一个 if 语句检查 `lineage_proof` 是否表明这不是 eve 花费（三个证明元素而不是两个）。如果不是 eve 花费，它会使用 `lineage_proof` 中的信息计算我们的 ID，以生成我们的父 ID。

如果它*是* eve 花费，则有一个额外的检查来验证我们拥有的启动器 ID 和启动器谜语哈希（均在 `SINGLETON_STRUCT` 中）是否正确。我们通过根据谱系证明中的信息和启动器谜语哈希计算启动器 ID 来实现这一点。然后我们宣称它在价值上等于柯里。这是非常重要的一步，因为它确保了这个单例硬币之后的每个单例硬币都可以信任启动器 ID 和谜语哈希，因为它将从这个 “eve” 单例硬币中强行输入，并且每个子单例硬币都知道 eve 单例硬币检查了它。

在 eve 单例验证了启动器信息后，它现在可以信任启动器 ID 作为其父 ID，并通过在最后阶段的 `full_puzhash` 和 `my_amount` 中散列来创建自己的 ID。再说说最后的“stager”：

```chialisp
(defun stager_three (SINGLETON_STRUCT lineage_proof my_id full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (c (list ASSERT_MY_COIN_ID my_id) (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (a INNER_PUZZLE (c (truth_data_to_truth_struct my_id full_puzhash innerpuzhash my_amount lineage_proof SINGLETON_STRUCT) inner_solution)) 0))
)
```

这个阶段是条件最终出现的地方。首先，它在前面添加了一个 `ASSERT_MY_COIN_ID` ，以便我们一直假设为真的所有谜底值都由网络隐式宣称。我们将这个条件添加到 `check_and_morph_conditions_for_singleton` 的输出中，它将从内部谜语中获取输出并检查单例硬币特定的东西（只有一个奇数输出，包装子单例硬币等）

请注意，在我们使用它来解决内部谜语之前，我们在谜底中预先添加了一些东西。我们正在使用一个来自 `singleton_truths.clib` 的函数，它获取所有列出的信息并将其组合成一个单一的结构来传递给内部谜语。这允许内部谜语使用单例硬币已经在自己的谜语中计算和验证的信息，而几乎不需要额外的成本！

请记住，这意味着内部谜语需要知道它正在进入单例硬币，否则其所有谜底参数都将向右移动。然而，一个现有的内部谜语可以很容易地适应，以使用一个浅外层：`(a (q . INNER_PUZZLE) (r 1))`，它在解开内部谜语之前剥离第一个谜底的值。

```chialisp
(defun check_and_morph_conditions_for_singleton (SINGLETON_STRUCT conditions has_odd_output_been_found)
  (if conditions
    (morph_next_condition SINGLETON_STRUCT conditions has_odd_output_been_found (odd_cons_m113 (created_coin_value_or_0 (f conditions))))
    (if has_odd_output_been_found
        0
        (x)  ;; no odd output found
    )
  )
)

(defun morph_next_condition (SINGLETON_STRUCT conditions has_odd_output_been_found (is_output_odd . is_output_m113))
   (assert
      (not (all is_output_odd has_odd_output_been_found))
      (strip_first_condition_if
         is_output_m113
         (c (if is_output_odd
                (morph_condition (f conditions) SINGLETON_STRUCT)
                (f conditions)
            )
            (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (r conditions) (any is_output_odd has_odd_output_been_found))
         )
      )
  )
)
```

这部分有点独特，因为它通过相互传递值来递归。我们的主要入口点是通过第一个块：`check_and_morph_conditions_for_singleton`，它首先检查我们是否还有条件。如果我们没有，我们检查是否已经设置了 `has_odd_output_been_found` 标志，如果没有设置，则提高。

如果我们确实有剩余条件，我们会将它们与检查第一个条件的结果一起传递给下一个函数，以查看它是输出为奇数的 `CREATE_COIN` 还是熔体值。

在 `morph_next_condition` 中，我们首先宣称我们没有找到第二个奇数输出。如果有，我们加注。如果我们还没有遇到奇怪的输出，我们将前往控制流的一个相当混乱的部分。最外层的函数调用本质上是等待最终的递归输出，如果找到，则去除熔化条件。该递归输出是通过采用第一个条件生成的，如果它是奇数，则将其包装在一个单例硬币外部谜语中，然后将其余条件传递回 `check_and_morph_conditions_for_singleton`，并设置 `has_odd_output_been_found` 标志（如果相关）。

<details>
<summary>原文参考</summary>

- ## The Singleton Top Layer

Here's the full source, we'll break it down after:

```chialisp
(mod (
       SINGLETON_STRUCT
       INNER_PUZZLE
       lineage_proof
       my_amount
       inner_solution
     )

     ;; SINGLETON_STRUCT = (MOD_HASH . (LAUNCHER_ID . LAUNCHER_PUZZLE_HASH))

     ; SINGLETON_STRUCT, INNER_PUZZLE are curried in by the wallet

  ; This puzzle is a wrapper around an inner smart puzzle which guarantees uniqueness.
  ; It takes its singleton identity from a coin with a launcher puzzle which guarantees that it is unique.

  (include condition_codes.clib)
  (include curry-and-treehash.clib)
  (include singleton_truths.clib)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ; "assert" is a macro that wraps repeated instances of "if"
  ; usage: (assert A0 A1 ... An R)
  ; all of A0, A1, ... An must evaluate to non-null, or an exception is raised
  ; return the value of R (if we get that far)

  (defmacro assert items
    (if (r items)
        (list if (f items) (c assert (r items)) (q . (x)))
        (f items)
    )
  )

  (defun-inline mod_hash_for_singleton_struct (SINGLETON_STRUCT) (f SINGLETON_STRUCT))
  (defun-inline launcher_id_for_singleton_struct (SINGLETON_STRUCT) (f (r SINGLETON_STRUCT)))
  (defun-inline launcher_puzzle_hash_for_singleton_struct (SINGLETON_STRUCT) (r (r SINGLETON_STRUCT)))

  ;; return the full puzzlehash for a singleton with the innerpuzzle curried in
  ; puzzle-hash-of-curried-function is imported from curry-and-treehash.clinc
  (defun-inline calculate_full_puzzle_hash (SINGLETON_STRUCT inner_puzzle_hash)
     (puzzle-hash-of-curried-function (mod_hash_for_singleton_struct SINGLETON_STRUCT)
                                      inner_puzzle_hash
                                      (sha256tree SINGLETON_STRUCT)
     )
  )

  ; assembles information from the solution to create our own full ID including asserting our parent is a singleton
  (defun create_my_ID (SINGLETON_STRUCT full_puzzle_hash parent_parent parent_inner_puzzle_hash parent_amount my_amount)
    (sha256 (sha256 parent_parent (calculate_full_puzzle_hash SINGLETON_STRUCT parent_inner_puzzle_hash) parent_amount)
            full_puzzle_hash
            my_amount)
  )

  ;; take a boolean and a non-empty list of conditions
  ;; strip off the first condition if a boolean is set
  ;; this is used to remove `(CREATE_COIN xxx -113)`
  ;; pretty sneaky, eh?
  (defun strip_first_condition_if (boolean condition_list)
    (if boolean
      (r condition_list)
      condition_list
    )
  )

  (defun-inline morph_condition (condition SINGLETON_STRUCT)
    (list (f condition) (calculate_full_puzzle_hash SINGLETON_STRUCT (f (r condition))) (f (r (r condition))))
  )

  ;; return the value of the coin created if this is a `CREATE_COIN` condition, or 0 otherwise
  (defun-inline created_coin_value_or_0 (condition)
    (if (= (f condition) CREATE_COIN)
        (f (r (r condition)))
        0
    )
  )

  ;; Returns a (bool . bool)
  (defun odd_cons_m113 (output_amount)
    (c
      (= (logand output_amount 1) 1) ;; is it odd?
      (= output_amount -113) ;; is it the escape value?
    )
  )

  ; Assert exactly one output with odd value exists - ignore it if value is -113

  ;; this function iterates over the output conditions from the inner puzzle & solution
  ;; and both checks that exactly one unique singleton child is created (with odd valued output),
  ;; and wraps the inner puzzle with this same singleton wrapper puzzle
  ;;
  ;; The special case where the output value is -113 means a child singleton is intentionally
  ;; *NOT* being created, thus forever ending this singleton's existence

  (defun check_and_morph_conditions_for_singleton (SINGLETON_STRUCT conditions has_odd_output_been_found)
      (if conditions
        (morph_next_condition SINGLETON_STRUCT conditions has_odd_output_been_found (odd_cons_m113 (created_coin_value_or_0 (f conditions))))
        (if has_odd_output_been_found
            0
            (x)  ;; no odd output found
        )
      )
   )

   ;; a continuation of `check_and_morph_conditions_for_singleton` with booleans `is_output_odd` and `is_output_m113`
   ;; precalculated
   (defun morph_next_condition (SINGLETON_STRUCT conditions has_odd_output_been_found (is_output_odd . is_output_m113))
       (assert
          (not (all is_output_odd has_odd_output_been_found))
          (strip_first_condition_if
             is_output_m113
             (c (if is_output_odd
                    (morph_condition (f conditions) SINGLETON_STRUCT)
                    (f conditions)
                )
                (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (r conditions) (any is_output_odd has_odd_output_been_found))
             )
          )
      )
   )

  ; this final stager asserts our ID
  ; it also runs the innerpuz with the innersolution with the "truths" added
  ; it then passes that output conditions from the innerpuz to the morph conditions function
  (defun stager_three (SINGLETON_STRUCT lineage_proof my_id full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (c (list ASSERT_MY_COIN_ID my_id) (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (a INNER_PUZZLE (c (truth_data_to_truth_struct my_id full_puzhash innerpuzhash my_amount lineage_proof SINGLETON_STRUCT) inner_solution)) 0))
  )

  ; this checks whether we are an eve spend or not and calculates our full coin ID appropriately and passes it on to the final stager
  ; if we are the eve spend it also adds the additional checks that our parent's puzzle is the standard launcher format and that out parent ID is the same as our singleton ID

  (defun stager_two (SINGLETON_STRUCT lineage_proof full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (stager_three
      SINGLETON_STRUCT
      lineage_proof
      (if (is_not_eve_proof lineage_proof)
          (create_my_ID
            SINGLETON_STRUCT
            full_puzhash
            (parent_info_for_lineage_proof lineage_proof)
            (puzzle_hash_for_lineage_proof lineage_proof)
            (amount_for_lineage_proof lineage_proof)
            my_amount
          )
          (if (=
                (launcher_id_for_singleton_struct SINGLETON_STRUCT)
                (sha256 (parent_info_for_eve_proof lineage_proof) (launcher_puzzle_hash_for_singleton_struct SINGLETON_STRUCT) (amount_for_eve_proof lineage_proof))
              )
              (sha256 (launcher_id_for_singleton_struct SINGLETON_STRUCT) full_puzhash my_amount)
              (x)
          )
      )
      full_puzhash
      innerpuzhash
      my_amount
      INNER_PUZZLE
      inner_solution
    )
  )

  ; this calculates our current full puzzle hash and passes it to stager two
  (defun stager_one (SINGLETON_STRUCT lineage_proof my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
    (stager_two SINGLETON_STRUCT lineage_proof (calculate_full_puzzle_hash SINGLETON_STRUCT my_innerpuzhash) my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
  )


  ; main

  ; if our value is not an odd amount then we are invalid
  ; this calculates my_innerpuzhash and passes all values to stager_one
  (if (logand my_amount 1)
    (stager_one SINGLETON_STRUCT lineage_proof (sha256tree INNER_PUZZLE) my_amount INNER_PUZZLE inner_solution)
    (x)
  )
)
```

Quite a bit isn't it?  Let's start with the arguments:

```chialisp
(
  SINGLETON_STRUCT
  INNER_PUZZLE
  lineage_proof
  my_amount
  inner_solution
)
```

`SINGLETON_STRUCT` is a collection of three things:
* The tree hash of this module
* The launcher coin ID (this acts as the unique ID for the singleton)
* The launcher puzzle hash

The reason they are grouped into a single structure is because they are passed through almost every function. It increases readability and optimization if they are passed through as a single variable until it is time to deconstruct them.

`INNER_PUZZLE` is the inner puzzle to this wrapper puzzle.

`lineage_proof` takes one of two formats:
* `(parent_parent_coin_info parent_inner_puzzle_hash parent_amount)`
* `(parent_parent_coin_info parent_amount)`
You may wonder, given the similarity, why not just use the first format?  We use the separate formats because we use the length of the structure to tip us off to whether or not this is the **eve spend**.
The eve spend is the first spend of a singleton after its creation.
We use this lineage proof to verify that our parent was a singleton.
However, in the first spend, the parent is not a singleton and we actually execute a different path where we verify that our parent was a singleton launcher instead.

`my_amount` is the amount of the coin being spent and will be asserted implicitly through ASSERT_MY_COIN_ID.

`inner_solution` is the solution the to inner puzzle.

Next, let's look at our main entry point:

```chialisp
(if (logand my_amount 1)
  (stager_one SINGLETON_STRUCT lineage_proof (sha256tree INNER_PUZZLE) my_amount INNER_PUZZLE inner_solution)
  (x)
)
```

The control flow here is very simple.
If we're not odd, we raise, if we are, we pass everything through to the next stage (with the additional hash of the inner puzzle).
One small thing to note is that a singleton can actually be even, but it will never be able to be spent.
Either the person will pass in the true amount and the puzzle will raise, or they will pass in a phony amount and the ASSERT_MY_ID will fail. If an attacker were to launch an even singleton or create one as one the even children of the singleton, it would succeed, but be stuck forever.

```chialisp
(defun stager_one (SINGLETON_STRUCT lineage_proof my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (stager_two SINGLETON_STRUCT lineage_proof (calculate_full_puzzle_hash SINGLETON_STRUCT my_innerpuzhash) my_innerpuzhash my_amount INNER_PUZZLE inner_solution)
)
```

We now move on to the first of a few "stagers". The purpose of these functions is to calculate values that are used multiple times only once.
In the next stage we use our full puzzle hash three times so it's best to calculate it once and pass it to the next function instead.

```chialisp
(defun stager_two (SINGLETON_STRUCT lineage_proof full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (stager_three
    SINGLETON_STRUCT
    lineage_proof
    (if (is_not_eve_proof lineage_proof)
        (create_my_ID
          SINGLETON_STRUCT
          full_puzhash
          (parent_info_for_lineage_proof lineage_proof)
          (puzzle_hash_for_lineage_proof lineage_proof)
          (amount_for_lineage_proof lineage_proof)
          my_amount
        )
        (if (=
              (launcher_id_for_singleton_struct SINGLETON_STRUCT)
              (sha256 (parent_info_for_eve_proof lineage_proof) (launcher_puzzle_hash_for_singleton_struct SINGLETON_STRUCT) (amount_for_eve_proof lineage_proof))
            )
            (sha256 (launcher_id_for_singleton_struct SINGLETON_STRUCT) full_puzhash my_amount)
            (x)
        )
    )
    full_puzhash
    innerpuzhash
    my_amount
    INNER_PUZZLE
    inner_solution
  )
)
```

This stage looks like a lot, but really all it's doing is calculating the current coin ID for the next function to use.
Note before we start looking at it that the lineage proof is frequently being passed to functions that are not part of this file.
These are part of the `singleton_truths.clib` library which we will discuss in the next stage.
For now, just know that it is accessing the correct values from the lineage proof and is a lot cleaner than writing things like `(f (r lineage_proof)) (f (r (r lineage_proof)))` with no indication of what they mean.

The first if statement checks if `lineage_proof` indicates that this is not the eve spend (three proof elements instead of two).
If it is not the eve spend, it calculates our ID using the information in the `lineage_proof` to generate our parent ID.

If it *is* the eve spend, there is an extra check which verifies that the launcher ID and launcher puzzle hash we have (both inside the `SINGLETON_STRUCT`) are correct. We do so by calculating the launcher ID from information in our lineage proof and the launcher puzzle hash.
We then assert that it is equal to the curried in value.
This is an extremely important step because it ensures that every singleton after this singleton can trust the launcher ID and puzzle hash since it will be forcefully curried in from this "eve" singleton and every child singleton knows that the eve singleton checked it.

After the eve singleton has verified the launcher info, it can now trust the launcher ID as its parent ID and create its own ID by hashing in the `full_puzhash` from the last stage and `my_amount`.
Let's talk about the final "stager":

```chialisp
(defun stager_three (SINGLETON_STRUCT lineage_proof my_id full_puzhash innerpuzhash my_amount INNER_PUZZLE inner_solution)
  (c (list ASSERT_MY_COIN_ID my_id) (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (a INNER_PUZZLE (c (truth_data_to_truth_struct my_id full_puzhash innerpuzhash my_amount lineage_proof SINGLETON_STRUCT) inner_solution)) 0))
)
```

This stage is where the conditions will end up coming out of.
First, it prepends an `ASSERT_MY_COIN_ID` so that all of the solution values we have been assuming to be true up until this point are implicitly asserted by the network.
We prepend this condition to the output of `check_and_morph_conditions_for_singleton` which will take the output from the inner puzzle and check for singleton specific things (only one odd output, wrap the child singleton, etc.)

Notice that we are prepending something to the solution before we use it to solve the inner puzzle.
We are using a function from `singleton_truths.clib` that takes all of the listed information and combines it into a single structure to pass to the inner puzzle.
This allows the inner puzzle to use information that the singleton has already calculated and verified in its own puzzle at almost no additional cost!

Keep in mind that this means an inner puzzle needs to know that it is going inside a singleton or else all of its solution arguments will be shifted to the right.
An existing inner puzzle can be very easily adapted, however, to fit inside a singleton using a shallow outer layer of: `(a (q . INNER_PUZZLE) (r 1))` which strips off the first value of the solution before solving the inner puzzle.

```chialisp
(defun check_and_morph_conditions_for_singleton (SINGLETON_STRUCT conditions has_odd_output_been_found)
  (if conditions
    (morph_next_condition SINGLETON_STRUCT conditions has_odd_output_been_found (odd_cons_m113 (created_coin_value_or_0 (f conditions))))
    (if has_odd_output_been_found
        0
        (x)  ;; no odd output found
    )
  )
)

(defun morph_next_condition (SINGLETON_STRUCT conditions has_odd_output_been_found (is_output_odd . is_output_m113))
   (assert
      (not (all is_output_odd has_odd_output_been_found))
      (strip_first_condition_if
         is_output_m113
         (c (if is_output_odd
                (morph_condition (f conditions) SINGLETON_STRUCT)
                (f conditions)
            )
            (check_and_morph_conditions_for_singleton SINGLETON_STRUCT (r conditions) (any is_output_odd has_odd_output_been_found))
         )
      )
  )
)
```

This section is a bit unique in that it recurses by handing values back and forth to each other.
Our main entry point is through the first block: `check_and_morph_conditions_for_singleton` which checks first if we still have conditions.
If we don't, we check to see if the `has_odd_output_been_found` flag has been set and raise if it hasn't been.

If we do have remaining conditions, we pass them to the next function along with the results of checking the first condition to see if it is a `CREATE_COIN` whose output is odd or the melt value.

In `morph_next_condition` we first assert that we have not found a second odd output.
If we have, we raise.
If we have not already run into an odd output, we head to a rather confusing section of the control flow.
The outermost function call essentially waits for the final recursive output and strips out the melt condition if it was found.
That recursive output is generated by taking the first condition, wrapping it in a singleton outer puzzle if it's odd, and then passing the rest of the conditions back to `check_and_morph_conditions_for_singleton` with the `has_odd_output_been_found` flag set if relevant.

</details>

## 支付给单例硬币

既然您了解了单例硬币的运作方式，我们现在可以看一个“支付给”单例硬币或以只有特定单例硬币的所有者才能解锁硬币的方式锁定硬币的示例。这个想法是你输入必要的信息来计算单例硬币的谜语哈希，然后断言来自单例硬币的声明说现在是时候领取锁定在谜语中的资金了。由于谜语哈希对于该硬币来说将是唯一的（由于包含了启动器 ID），因此只有该单例硬币才能创建适当的公告。这是谜语：

```chialisp
(mod (
       SINGLETON_MOD_HASH
       LAUNCHER_ID
       LAUNCHER_PUZZLE_HASH
       singleton_inner_puzzle_hash
       my_id
     )

  ; SINGLETON_MOD_HASH is the mod-hash for the singleton_top_layer puzzle
  ; LAUNCHER_ID is the ID of the singleton we are commited to paying to
  ; LAUNCHER_PUZZLE_HASH is the puzzle hash of the launcher
  ; singleton_inner_puzzle_hash is the innerpuzzlehash for our singleton at the current time
  ; my_id is the coin_id of the coin that this puzzle is locked into

  (include condition_codes.clvm)
  (include curry-and-treehash.clinc)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ;; return the full puzzlehash for a singleton with the innerpuzzle curried in
  ; puzzle-hash-of-curried-function is imported from curry-and-treehash.clinc
  (defun-inline calculate_full_puzzle_hash (SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH inner_puzzle_hash)
     (puzzle-hash-of-curried-function SINGLETON_MOD_HASH
                                      inner_puzzle_hash
                                      (sha256tree (c SINGLETON_MOD_HASH (c LAUNCHER_ID LAUNCHER_PUZZLE_HASH)))
     )
  )

  (defun-inline claim_rewards (SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash my_id)
    (list
        (list ASSERT_PUZZLE_ANNOUNCEMENT (sha256 (calculate_full_puzzle_hash SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash) my_id))
        (list CREATE_COIN_ANNOUNCEMENT '$')
        (list ASSERT_MY_COIN_ID my_id))
  )

  ; main
  (claim_rewards SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash my_id)
)
```

Most of this puzzle should be self explanatory especially if you've gone through the puzzles above.
Let focus on just the conditions we are creating from the `claim_rewards` function:

```chialisp
(list
    (list ASSERT_PUZZLE_ANNOUNCEMENT (sha256 (calculate_full_puzzle_hash SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash) my_id))
    (list CREATE_COIN_ANNOUNCEMENT '$')
    (list ASSERT_MY_COIN_ID my_id)
)
```

我们都在宣称来自单例硬币的声明并为其创建一个声明。这个宣称是我们只想被一个非常具体的单例硬币声明的事实的基础。由于启动器 ID 被压缩到单例硬币的谜语哈希中，它对每个单例硬币都是唯一的，因此只能由我们指定其启动器 ID 的单例硬币声明。我们不能使用单例硬币 ID，因为如果我们将其输入，单例硬币可能会花费，然后这个谜语就无法解开了！

我们创建的公告只是为了让单例硬币宣称我们也被花费了。这是必要的，因为[节点可能会尝试排除这笔花费](https://chialisp.com/docs/security#replay-attacks)导致单例硬币在没有获得这些奖励的情况下消费。由于这个硬币不能被签名，我们必须以某种方式确保如果它被排除在外，整个花费组合就会失败。我们使用 `$` 因为它是一个字节并且有点相关。

<details>
<summary>原文参考</summary>

- ## Pay to Singleton

Now that you understand how a singleton functions, we can now look at an example of "paying to" a singleton or locking up a coin in such a way that only the owner of a specific singleton can unlock it.
The idea is that you curry in the necessary information to calculate the singleton's puzzle hash and then assert an announcement from the singleton that says that it is time to claim the funds locked up in the puzzle. Since the puzzle hash will be unique to that singleton (due to the launcher ID being curried in), only that singleton will be able to create the appropriate announcement. Here's the puzzle:

```chialisp
(mod (
       SINGLETON_MOD_HASH
       LAUNCHER_ID
       LAUNCHER_PUZZLE_HASH
       singleton_inner_puzzle_hash
       my_id
     )

  ; SINGLETON_MOD_HASH is the mod-hash for the singleton_top_layer puzzle
  ; LAUNCHER_ID is the ID of the singleton we are commited to paying to
  ; LAUNCHER_PUZZLE_HASH is the puzzle hash of the launcher
  ; singleton_inner_puzzle_hash is the innerpuzzlehash for our singleton at the current time
  ; my_id is the coin_id of the coin that this puzzle is locked into

  (include condition_codes.clvm)
  (include curry-and-treehash.clinc)

  ; takes a lisp tree and returns the hash of it
  (defun sha256tree (TREE)
      (if (l TREE)
          (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
          (sha256 1 TREE)
      )
  )

  ;; return the full puzzlehash for a singleton with the innerpuzzle curried in
  ; puzzle-hash-of-curried-function is imported from curry-and-treehash.clinc
  (defun-inline calculate_full_puzzle_hash (SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH inner_puzzle_hash)
     (puzzle-hash-of-curried-function SINGLETON_MOD_HASH
                                      inner_puzzle_hash
                                      (sha256tree (c SINGLETON_MOD_HASH (c LAUNCHER_ID LAUNCHER_PUZZLE_HASH)))
     )
  )

  (defun-inline claim_rewards (SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash my_id)
    (list
        (list ASSERT_PUZZLE_ANNOUNCEMENT (sha256 (calculate_full_puzzle_hash SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash) my_id))
        (list CREATE_COIN_ANNOUNCEMENT '$')
        (list ASSERT_MY_COIN_ID my_id))
  )

  ; main
  (claim_rewards SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash my_id)
)
```

Most of this puzzle should be self explanatory especially if you've gone through the puzzles above.
Let focus on just the conditions we are creating from the `claim_rewards` function:

```chialisp
(list
    (list ASSERT_PUZZLE_ANNOUNCEMENT (sha256 (calculate_full_puzzle_hash SINGLETON_MOD_HASH LAUNCHER_ID LAUNCHER_PUZZLE_HASH singleton_inner_puzzle_hash) my_id))
    (list CREATE_COIN_ANNOUNCEMENT '$')
    (list ASSERT_MY_COIN_ID my_id)
)
```

We are both asserting an announcement from the singleton and creating one for it.
The assertion is fundamental to the fact that we only want to be claimed by a very specific singleton.
Due to the launcher ID being curried into the singleton's puzzle hash, it will be unique to every singleton and can thereby only be claimed by the singleton whose launcher ID we specify.
We cannot use the singleton's coin ID, because if we curried that in, the singleton could spend and then this puzzle becomes unsolvable!

The announcement that we create is simply for the singleton to assert that we are also being spent.
This is necessary due to the fact that [nodes may try and exclude this spend](https://chialisp.com/docs/security#replay-attacks) causing the singleton to spend without claiming these rewards.
Since this coin cannot be signed, we must ensure somehow that if it is excluded, the whole spend bundle fails.
We use `'$'` because it's one byte and somewhat relevant.

The coin ID assertion is simply to ensure that we are being told the truth about our id. Otherwise, we could piggy back on another claim by using that coin's ID and asserting the announcement that the singleton creates for it.

</details>
