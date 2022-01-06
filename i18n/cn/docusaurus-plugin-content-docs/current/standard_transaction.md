---
id: standard_transaction
title: 6 - 标准交易
---

> The Standard Transaction

您现在应该精通使用 Chialisp 谜语锁定硬币的多种方法。我们现在拥有了讨论 Chia 网络上标准交易格式所需的所有工具。

在您阅读本节之前，不妨先看看 Bram Cohen 的这篇[博客文章](https://www.chia.net/2021/05/27/Agrgregated-Sigs-Taproot-Graftroot.html)关于为什么标准交易是这样的。

<details>
<summary>原文参考</summary>

You should now be well versed in a number of ways to lock up a coin using a Chialisp puzzle.
We have all the tools we need now to talk about the standard transaction format on the Chia network.

Before you go through this section, it may be worth it to check out this [blog post](https://www.chia.net/2021/05/27/Agrgregated-Sigs-Taproot-Graftroot.html) by Bram Cohen on why the standard transaction is the way it is.

</details>

## 支付给“委托谜语”或“隐藏谜语”

如果您还记得[我们对硬币、支出和钱包的讨论](/docs/coins_spends_and_wallets)，我们创建了一个谜语，该谜语支付给“委托谜语”：该谜语允许求解者传入谜语和谜底以创建他们的自己的输出条件。这是我们希望标准事务具有的功能的一半。

然而，我们也希望能够在不透露谜语的情况下预先提交谜语，并让任何了解“隐藏”谜语的人花费它。

但是我们如何预先承诺这个隐藏的谜语呢？我们可以把它隐藏起来，但是如果我们执行委托支出案例，我们将不得不显示完整的谜语，包括隐藏的隐藏谜语，它不再被隐藏。我们不能再用相同的谜语锁定硬币，否则人们将能够知道谜语哈希是相同的，并在未经我们同意的情况下花费它。我们的委托支出甚至可能无法进入网络；恶意节点可以在看到我们的交易后拒绝我们的交易，然后自行发布隐藏的支出案例。

我们可以尝试通过散列隐藏的谜语来解决这个问题。这有一些类似的问题。如果您将隐藏的箱子花掉一次，人们可以在以后看到任何相同的谜语哈希并在未经您同意的情况下使用它们。此外，许多人可能会尝试使用相同的隐藏谜语。如果有人透露它，所有锁定在同一谜题上的硬币也可以被识别和花费。我们需要隐藏这个谜语，但也需要一些熵来保持它对我们来说是独一无二的。

标准交易使用的谜底是从 a) 隐藏的谜语和 b) 可以为委托支出情况签名的公钥派生出一个新的私钥：

`synthetic_offset == sha256(hidden_puzzle_hash + original_public_key)`

然后我们计算这个新私钥的公钥，并将其添加到我们现有的原始公钥中：

`synthentic_public_key == original_public_key + synthetic_offset_pubkey`

如果求解器可以正确地揭示隐藏的谜语和原始公钥，那么我们的谜语就可以导出合成公钥并确保它与输入的公钥匹配。

您可能想知道为什么我们将派生私钥中的公钥添加到原始公钥中，因为它已经是派生的一部分。这是因为我们也使用合成公钥来签署我们的委托支出。当您添加两个公钥时，生成的公钥的私钥是原始私钥的总和。如果我们不添加原始公钥，那么任何知道隐藏谜语的人都可以推导出合成私钥，然后执行委托支出！添加原始公钥可确保合成私钥仍有一个秘密部分，即使其中一半是已知的。

这种技术也很巧妙，因为它允许我们将隐藏的谜语隐藏在委托支出已经需要的一条信息中。即使是标准的隐藏谜语，也无法猜出隐藏的谜语是什么！甚至很难判断是否有隐藏的谜语。这也有助于保护隐私。例如，如果两方同意将一些带有隐藏谜语的硬币锁定在一起，您可以共享公钥并在区块链上验证该信息，而无需向网络透露任何信息。然后，如果你们都同意在任何一方不诚实的情况下都可以将硬币用于隐藏的谜语，那么您可以无信任地委托将硬币花费到正确的目的地，并且不可能说它们不仅仅是正常的日常支出。

稍后我们将查看代码，但在查看代码之前，您需要了解以下几个术语：

* **隐藏的谜语**：一个“隐藏的谜语”，可以揭示并用作解锁基础资金的另一种方式
* **合成密钥偏移量**：使用隐藏谜语和 `original_public_key` 作为输入以加密方式生成的私钥
* **合成公钥**：公钥（curried in）是 `original_public_key` 和 `synthetic_key_offset` 对应的公钥之和
* **原始公钥**：公钥，其中对应私钥的知识代表硬币的所有权
* **委托谜语**：委托谜语，如 `graftroot`，它应该返回所需的条件。
* **谜底**：委托或隐藏谜语的谜底

<details>
<summary>原文参考</summary>

- ## Pay to "Delegated Puzzle" or "Hidden Puzzle"

If you remember from [our discussion of coins, spends, and wallets](/docs/coins_spends_and_wallets) we created a puzzle that paid to a "delegated puzzle": a puzzle that allows the solver to pass in a puzzle and solution to create their own conditions for the output.
This is one half of the functionality we want our standard transaction to have.

However, we also want the ability to pre-commit to a puzzle without revealing it, and let anybody with the knowledge of the "hidden" puzzle spend it.

But how do we pre-commit to this hidden puzzle?  We can curry it in, but if we perform the delegated spend case we will have to reveal the full puzzle including the curried in hidden puzzle and it will no longer be hidden.
We can't lock up a coin with the same puzzle anymore, or else people will be able to tell that the puzzle hash is the same and spend it without our consent.
Our delegated spend might not even make it to the network; a malicious node can just deny our transaction after seeing it and then publish the hidden spend case on their own.

We can attempt to solve this by hashing the hidden puzzle.
This has some similar problems.
If you spend the hidden case even once, people can see any identical puzzle hashes later and spend them without your consent.
Furthermore, many people may try to use the same hidden puzzle.
If anyone reveals it, all coins locked up with that same puzzle can also be identified and spent.
We need the puzzle to be hidden, but also have some entropy that keeps it unique to us.

The solution that the standard transaction uses is to derive a new private key from a) the hidden puzzle and b) the public key that can sign for the delegated spend case:

`synthetic_offset == sha256(hidden_puzzle_hash + original_public_key)`

We then calculate the public key of this new private key, and add it to our existing original public key:

`synthentic_public_key == original_public_key + synthetic_offset_pubkey`

If the solver can correctly reveal BOTH the hidden puzzle and the original public key, then our puzzle can derive the synthetic public key and make sure that it matches the one that is curried in.

You may wonder why we add the public key from our derived private key to the original public key when it's already part of the derivation.
This is because we use the synthetic public key to sign for our delegated spends as well.
When you add two public keys, the private key for the resulting public key is the sum of the original private keys.
If we didn't add the original public key then anyone who knew the hidden puzzle could derive the synthetic private key and could then perform delegated spends!  Adding original public key ensures that there is still a secret component of the synthetic private key, even though half of can be known.

This technique is also neat because it allows us to hide the hidden puzzle in a piece of information that was already necessary for the delegated spend.
It's impossible to guess what the hidden puzzle is, even if it's a standard hidden puzzle!  It's even hard to tell if there's a hidden puzzle at all.
This can also contribute to privacy.
For example, if two parties agree to lock up some coins with a hidden puzzle together, you can share pubkeys and verify that information on the blockchain without revealing anything to the network.
Then, if you both agree that the coins *can* be spent with the hidden puzzle if either party is dishonest, you can trustlessly delegated spend the coins to the correct destinations and it's impossible to tell that they are not just normal everyday spends.

We'll look at the code in a moment, but here's a few terms to know before you look at it:

* **hidden puzzle**: a "hidden puzzle" that can be revealed and used as an alternate way to unlock the underlying funds
* **synthetic key offset**: a private key cryptographically generated using the hidden puzzle and `original_public_key` as inputs
* **synthetic public key**: the public key (curried in) that is the sum of `original_public_key` and the public key corresponding to `synthetic_key_offset`
* **original public key**: a public key, where knowledge of the corresponding private key represents ownership of the coin
* **delegated puzzle**: a delegated puzzle, as in "graftroot", which should return the desired conditions.
* **solution**: the solution to the delegated or hidden puzzle

</details>

## Chialisp

这是完整的源代码，然后我们将对其进行分解：

```chialisp
(mod

    (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle solution)

    ; "assert" is a macro that wraps repeated instances of "if"
    ; usage: (assert A0 A1 ... An R)
    ; all of A0, A1, ... An must evaluate to non-null, or an exception is raised
    ; return the last item (if we get that far)

    (defmacro assert (items)
        (if (r items)
            (list if (f items) (c assert (r items)) (q . (x)))
            (f items)
        )
    )

    (include condition_codes.clvm)
    (include sha256tree.clvm)

    ; "is_hidden_puzzle_correct" returns true iff the hidden puzzle is correctly encoded

    (defun-inline is_hidden_puzzle_correct (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
      (=
          SYNTHETIC_PUBLIC_KEY
          (point_add
              original_public_key
              (pubkey_for_exp (sha256 original_public_key (sha256tree delegated_puzzle)))
          )
      )
    )

    ; "possibly_prepend_aggsig" is the main entry point

    (defun-inline possibly_prepend_aggsig (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle conditions)
      (if original_public_key
          (assert
              (is_hidden_puzzle_correct SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
              conditions
          )
          (c (list AGG_SIG_ME SYNTHETIC_PUBLIC_KEY (sha256tree delegated_puzzle)) conditions)
      )
    )

    ; main entry point

    (possibly_prepend_aggsig
        SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle
        (a delegated_puzzle solution))
)
```

这可能需要消化很多，所以让我们一块一块地分解它。首先，让我们谈谈论点：

```
(SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle solution)
```

所有这些术语都在上面定义。当我们解谜时：
* `SYNTHETIC_PUBLIC_KEY` 是柯里化的
* 如果是隐藏支出，则传入 `original_public_key`，如果是委托支出，则传入 `()`
* `delegated_puzzle` 是隐藏的谜语，如果是隐藏的花费，或者是委托的谜语，如果是委托的花费
* `solution` 是任何传入 `delegated_puzzle` 的谜底

与大多数 Chialisp 程序一样，我们将从底部开始查看实现：

```chialisp
(possibly_prepend_aggsig
    SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle
    (a delegated_puzzle solution))
```

这里没什么大不了的，我们主要只是将参数传递给 `possively_prepend_aggsig` 来启动程序。唯一需要注意的是，我们在传入之前使用谜底计算委托的谜语。这将产生一个条件列表，只要谜题的其余部分检查出来，我们就会输出这些条件。

```chialisp
(defun-inline possibly_prepend_aggsig (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle conditions)
  (if original_public_key
      (assert
          (is_hidden_puzzle_correct SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle) ; hidden case
          conditions
      )
      (c (list AGG_SIG_ME SYNTHETIC_PUBLIC_KEY (sha256tree delegated_puzzle)) conditions) ; delegated case
  )
)
```

这个函数是主要的控制流逻辑，它决定了我们是在进行“隐藏”还是“委托”支出。第一行只是检查是否传入了一个 `original_public_key`。在委托支出中，我们为该参数传递 `()`，由于它的计算结果为 false，它可以很好地作为一个开关来确定我们在做什么。

如果支出是隐藏支出，我们将大部分参数传递给 `is_hidden_puzzle_correct`，只要它没有失败，我们就返回给我们的任何条件。如果支出是委托支出，我们会在委托谜语的哈希上添加一个来自公钥的签名要求。

```chialisp
(defun-inline is_hidden_puzzle_correct (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
  (=
      SYNTHETIC_PUBLIC_KEY
      (point_add
          original_public_key
          (pubkey_for_exp (sha256 original_public_key (sha256tree delegated_puzzle)))
      )
  )
)
```

这是上一节中解释的 Chialisp 表示。 私钥是任意 32 字节，因此我们将使用 `sha256`（其输出是 32 字节）来确保我们的私钥是从 `original_public_key` 和隐藏谜语的哈希派生的。我们将结果散列传递给 `pubkey_for_exp`，它将我们的私钥变成公钥。然后，我们 `point_add` 这个生成的公钥到我们的原始公钥以获得我们的合成公钥。如果它等于一个柯里，则此函数通过，否则返回 `()` 并且上一个函数的 `assert` 引发。

<details>
<summary>原文参考</summary>

- ## The Chialisp

Here's the full source and then we'll break it down:

```chialisp
(mod

    (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle solution)

    ; "assert" is a macro that wraps repeated instances of "if"
    ; usage: (assert A0 A1 ... An R)
    ; all of A0, A1, ... An must evaluate to non-null, or an exception is raised
    ; return the last item (if we get that far)

    (defmacro assert (items)
        (if (r items)
            (list if (f items) (c assert (r items)) (q . (x)))
            (f items)
        )
    )

    (include condition_codes.clvm)
    (include sha256tree.clvm)

    ; "is_hidden_puzzle_correct" returns true iff the hidden puzzle is correctly encoded

    (defun-inline is_hidden_puzzle_correct (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
      (=
          SYNTHETIC_PUBLIC_KEY
          (point_add
              original_public_key
              (pubkey_for_exp (sha256 original_public_key (sha256tree delegated_puzzle)))
          )
      )
    )

    ; "possibly_prepend_aggsig" is the main entry point

    (defun-inline possibly_prepend_aggsig (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle conditions)
      (if original_public_key
          (assert
              (is_hidden_puzzle_correct SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
              conditions
          )
          (c (list AGG_SIG_ME SYNTHETIC_PUBLIC_KEY (sha256tree delegated_puzzle)) conditions)
      )
    )

    ; main entry point

    (possibly_prepend_aggsig
        SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle
        (a delegated_puzzle solution))
)
```

That's probably a lot to digest so let's break it down piece by piece.
First, let's talk about the arguments:

```
(SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle solution)
```

All of these terms are defined above.
When we solve this puzzle:
* `SYNTHETIC_PUBLIC_KEY` is curried in
* We pass in `original_public_key` if it's the hidden spend or `()` if it's the delegated spend
* `delegated_puzzle` is the hidden puzzle if it's the hidden spend, or the delegated puzzle if it's the delegated spend
* `solution` is the solution to whatever is passed into `delegated_puzzle`

As with most Chialisp programs, we'll start looking at the implementation from the bottom:

```chialisp
(possibly_prepend_aggsig
    SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle
    (a delegated_puzzle solution))
```

There's nothing much going on here, we're mostly just passing arguments to `possibly_prepend_aggsig` to start the program.
The only thing to note is that we're evaluating the delegated puzzle with the solution before passing it in.
This will result in a list of conditions that we will output as long as the rest of the puzzle checks out.

```chialisp
(defun-inline possibly_prepend_aggsig (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle conditions)
  (if original_public_key
      (assert
          (is_hidden_puzzle_correct SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle) ; hidden case
          conditions
      )
      (c (list AGG_SIG_ME SYNTHETIC_PUBLIC_KEY (sha256tree delegated_puzzle)) conditions) ; delegated case
  )
)
```

This function is the main control flow logic that determines whether we're doing the "hidden" or "delegated" spend.
The first line just checks if an `original_public_key` was passed in.
In the delegated spend, we pass `()` for that argument, and since that evaluates to false, it works great as a switch to determine what we're doing.

If the spend is the hidden spend, we pass most of our parameters to `is_hidden_puzzle_correct` and, as long as it doesn't fail, we just return whatever conditions are given to us.
If the spend is the delegated spend, we prepend a signature requirement from the curried in public key on the hash of the delegated puzzle.

```chialisp
(defun-inline is_hidden_puzzle_correct (SYNTHETIC_PUBLIC_KEY original_public_key delegated_puzzle)
  (=
      SYNTHETIC_PUBLIC_KEY
      (point_add
          original_public_key
          (pubkey_for_exp (sha256 original_public_key (sha256tree delegated_puzzle)))
      )
  )
)
```

This is the Chialisp representation of what was explained in the section above.
A private key is any 32 bytes so we're going to use `sha256` (whose output is 32 bytes) to make sure our private key is derived from the `original_public_key` and the hash of the hidden puzzle.
We pass the resulting hash to `pubkey_for_exp` which turns our private key into a public key.
Then, we `point_add` this generated public key to our original pubkey to get our synthetic public key.
If it equals the curried in one, this function passes, otherwise it returns `()` and the `assert` from the previous function raises.

</details>

## 结论

这个谜语保护了 Chia 网络上几乎所有的硬币。当您使用 Chia Network 钱包软件时，它会在区块链上爬行，寻找以这种特定格式锁定的硬币。它正在寻找的 `SYNTHETIC_PUBLIC_KEY` 实际上使用了一个隐藏的 `(=)` 谜语，这显然是无效的并且会立即失败。这是因为大多数 Chia 用户不需要香草交易的隐藏谜语功能。但是，通过内置功能，它可以在以后实现更酷的功能。这个谜语也为您可能写的任何智能硬币提供了一个奇妙的内在谜语。

<details>
<summary>原文参考</summary>

- ## Conclusion

This puzzle secures almost all of the coins on the Chia network.
When you use the Chia Network wallet software, it is crawling the blockchain looking for coins locked up with this specific format.
The `SYNTHETIC_PUBLIC_KEY` it is looking for is actually using a hidden puzzle of `(=)` which is obviously invalid and fails immediately.
This is because most users of Chia don't need the hidden puzzle functionality for vanilla transactions.
But, by having the capabilities built in, it enables much cooler functionality later on.
This puzzle also makes for a fantastic inner puzzle of any smart coins you may write.

</details>
