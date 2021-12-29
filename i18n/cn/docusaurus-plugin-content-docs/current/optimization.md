---
id: optimization
title: 10 - 优化
---

> Optimization

在将智能硬币部署到网络之前，您应该仔细检查代码以找到优化其执行的方法。请记住，您编写的代码将部署在数百万个节点上； 如果它很慢，你就会减慢整个网络的速度。 这就是为什么 Chia 网络上的区块约束依赖于必须在该区块中运行的 Chialisp 的[程序执行成本](https://chialisp.com/docs/ref/clvm/#costs)。如果你想写一个更大的、运行缓慢的硬币，你每次想要花费它都需要支付更多的费用。让我们来看看一些你可以用来优化你的谜题的技巧。

<details>
<summary>原文参考</summary>

Before you deploy a smart coin to the network, you should closely examine the code to find ways to optimize its execution.
Remember, the code you write is going to be deployed on millions of nodes; if it is slow, you slow down the whole network. This is why the block constraint on the Chia Network is dependent on the [program execution cost](https://chialisp.com/docs/ref/clvm/#costs) of Chialisp that has to be run in that block.
If you want to write a bigger, slow running coin, you're going to need to pay more fees every time you want to spend it.
Let's go over some techniques you can use to optimize your puzzles.

</details>

## 尽量减少花费

归根结底，最大的成本消耗之一将是您必须花费硬币的频率。很常见的是，您会找到构建硬币的方法，参与者需要花费硬币才能与之交互。硬币在这样做时可能会穿越多个状态。每次必须花费硬币时，它都会作为您的基本程序成本的乘数。即使您没有通过代码遍历昂贵的路径，仍然必须揭示完整的谜题，并且很可能存在 `CREATE_COIN` 和 `AGG_SIG_ME` 条件，这通常代表了大部分成本。

尽可能少的签名和签名操作也很重要。通常最好的做法是收集程序中需要签名的所有内容，将它们散列在一起，并要求对该散列进行单一签名。有时，您也可以巧妙地发布公告，其中未签名的硬币可以从签名的硬币中断言其相关信息。要有创意，但始终记住要仔细检查每条重要信息是否已签名或声明。

<details>
<summary>原文参考</summary>

- ## Minimize the number spends

At the end of the day, one of the biggest drains on cost is going to be how often you have to spend the coin.
Quite commonly, you will find ways to build coins where participants are required to spend the coin in order to interact with it.
The coin may traverse through multiple states as they do so.
Every time the coin has to be spent, it acts as a multiplier for your base program cost.
Even if you are not traversing an expensive path through the code, the full puzzle must still be revealed and there will most likely be `CREATE_COIN` and `AGG_SIG_ME` conditions which often represent a large chunk of the cost.

It is also important that you have as few signatures and signature operations as possible.
It is usually best practice to collect everything in your program that needs signing, hash it all together and ask for a single signature on that hash.
You can also sometimes be clever with announcements where an unsigned coin can assert its relevant information from a signed coin.
Be creative, but always remember to double check that every piece of important information is signed or asserted.

</details>

## `defun` vs `defun-inline`

在大多数情况下，最好使用内联函数而不是常规函数。 内联函数在编译时被调用的地方插入，这将消除函数调用开销，并且不会在代码中单独存储函数。

有一种可能的情况并非如此。 如果您使用带有已计算参数的内联函数，则每次引用该参数时您最终都会为该计算付费：


```chialisp
(defun-inline add_to_self (x) (+ x x))

(add_to_self (* 200 200))
```

上面的代码片段将导致以下扩展：

```chialisp
(+ (* 200 200) (* 200 200))
```

如您所见，昂贵的乘法运算现在已经执行了两次！

<details>
<summary>原文参考</summary>

- ## `defun` vs `defun-inline`

In most instances, it is better to use inline functions rather than regular functions.
Inline functions get inserted where they are called at compile time which will eliminate the function call overhead and will not store the function separately in the code.

There is a potential scenario where this is not true.
If you are using an inline function with an argument that has been calculated, you will end up paying for that calculation every time the argument is referenced:

```chialisp
(defun-inline add_to_self (x) (+ x x))

(add_to_self (* 200 200))
```

The above code snippet will result in the following expansion:

```chialisp
(+ (* 200 200) (* 200 200))
```

As you can see, the expensive multiplication operation has now been performed twice!

</details>

## 熟悉所有操作员

请务必查看 [参考部分](https://chialisp.com/docs/ref/clvm) 以了解您可以使用的每个运算符及其成本。 您可能想使用的许多常见运算符的成本高得惊人，最好避开。

例如，您可能希望根据数字是偶数还是奇数进行不同的评估：

```chialisp
(if (r (divmod value 2))
  ; do odd things
  ; do even things
)
```

*请注意，if 利用了 0 == () 这一事实。 这种技术在遍历列表时也很方便。 列表中的最后一项始终为 ()，其计算结果为 false，因此在这种情况下，您可以中断递归。*

然而，`divmod` 是一个非常昂贵的操作，一旦操作完成，我们必须添加一个 `r` 来访问余数。 相反，我们可以只使用 `logand` 来评估最后一位：

```chialisp
(if (logand value 1)
  ; do odd things
  ; do even things
)
```

我们现在已经为自己节省了至少 50% 的代码块成本！

<details>
<summary>原文参考</summary>

- ## Familiarize yourself with all of the operators

Make sure to check out the [reference section](https://chialisp.com/docs/ref/clvm) to find out every operator that you can use and what they cost.
A lot of common operators that you might be tempted to use have a surprisingly high cost and are best to steer clear of.

For example, you may want to evaluate differently based on whether a number is even or odd:

```chialisp
(if (r (divmod value 2))
  ; do odd things
  ; do even things
)
```
*Note that the if takes advantage of the fact that 0 == (). This technique is handy when recursing through lists too.
The last item in a list is always () which evaluates to false, so in that case you can break the recursion.*

However, `divmod` is a pretty expensive operation, and we have to add an `r` to access the remainder once the operation has completed.
Instead, we can just use `logand` to evaluate just the last bit:

```chialisp
(if (logand value 1)
  ; do odd things
  ; do even things
)
```

We have now saved ourselves at least 50% of the cost of this code block!

</details>

## 保持参数数量小

这个技巧对优化和可读性都有好处。 当程序运行时，它需要支付成本来从环境中查找一个值。 这不是一个很大的成本，但它变得越大，它必须进入环境树以搜索该值的深度。 如果您可以保持较小的参数数量，则每次程序在其评估中使用参数时，您都可以削减成本。

一种方法是批量处理总是在同一个地方结束的参数。 下面是一个例子：

```chialisp
(mod (
      CURRIED_PUBKEY
      some_data
      some_other_data
      some_more_data
      even_more_data
      pubkey
      my_amount
      my_id
     )

     (import "condition_codes.clvm")
     (import "sha256tree.clvm")

     (defun-inline agg_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data)
        (AGG_SIG_ME CURRIED_PUBKEY (sha256tree (list some_data some_other_data some_more_data even_more_data)))
     )

     (defun-inline assert_amount_and_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount)
        (c (ASSERT_MY_AMOUNT my_amount) (agg_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data))
     )

     (defun-inline assert_id_and_amount_and_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount my_id)
        (c ASSERT_MY_ID my_id (assert_amount_and_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount))
     )

     (assert_id_and_amount_and_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount my_id)
)
```

你可以看到代码从可读性的角度来看有点失控，在访问 `my_amount` 和 `my_id` 时，我们必须深入环境树来读取它们的值。相反，我们应该将所有数据批处理到一个列表中。

```chialisp
(mod (
      CURRIED_PUBKEY
      all_data
      pubkey
      my_amount
      my_id
     )

     (import "condition_codes.clvm")
     (import "sha256tree.clvm")

     (defun-inline agg_sig (CURRIED_PUBKEY all_data)
        (AGG_SIG_ME CURRIED_PUBKEY (sha256tree all_data))
     )

     (defun-inline assert_amount_and_sig (CURRIED_PUBKEY all_data my_amount)
        (c (ASSERT_MY_AMOUNT my_amount) (agg_sig CURRIED_PUBKEY all_data))
     )

     (defun-inline assert_id_and_amount_and_sig (CURRIED_PUBKEY all_data my_amount my_id)
        (c ASSERT_MY_ID my_id (assert_amount_and_sig CURRIED_PUBKEY all_data my_amount))
     )

     (assert_id_and_amount_and_sig CURRIED_PUBKEY all_data my_amount my_id)
)
```

这更简洁，我们可以依靠求解器为我们将相关数据放入列表中，因此我们也可以减去该成本。这通常有点小众，但在尝试通过内部谜题传递数据时变得很重要。如果最外面的拼图想要与最里面的拼图通信，它就必须通过其间拼图的每一步传递它需要的所有数据。如果将它打包成一个包，那就容易多了。

<details>
<summary>原文参考</summary>

- ## Keep argument numbers small

This tip is both good for optimization and readability.
As the program is running, it needs to pay cost to look up a value from the environment.
It is not a large cost, but it gets larger the deeper it has to go into the environment tree to search for the value.
If you can keep the argument numbers small, you can trim off cost every time your program uses an argument in its evaluation.

One way to do this is to batch arguments that always end up in the same place together.
Here's an example:

```chialisp
(mod (
      CURRIED_PUBKEY
      some_data
      some_other_data
      some_more_data
      even_more_data
      pubkey
      my_amount
      my_id
     )

     (import "condition_codes.clvm")
     (import "sha256tree.clvm")

     (defun-inline agg_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data)
        (AGG_SIG_ME CURRIED_PUBKEY (sha256tree (list some_data some_other_data some_more_data even_more_data)))
     )

     (defun-inline assert_amount_and_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount)
        (c (ASSERT_MY_AMOUNT my_amount) (agg_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data))
     )

     (defun-inline assert_id_and_amount_and_sig (CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount my_id)
        (c ASSERT_MY_ID my_id (assert_amount_and_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount))
     )

     (assert_id_and_amount_and_sig CURRIED_PUBKEY some_data some_other_data some_more_data even_more_data my_amount my_id)
)
```

You can see the code is a little bit out of control from a readability standpoint, and when accessing `my_amount` and `my_id` we have to go deep into the environment tree to read their values.
Instead, we should just batch all of our data into a list to start with.

```chialisp
(mod (
      CURRIED_PUBKEY
      all_data
      pubkey
      my_amount
      my_id
     )

     (import "condition_codes.clvm")
     (import "sha256tree.clvm")

     (defun-inline agg_sig (CURRIED_PUBKEY all_data)
        (AGG_SIG_ME CURRIED_PUBKEY (sha256tree all_data))
     )

     (defun-inline assert_amount_and_sig (CURRIED_PUBKEY all_data my_amount)
        (c (ASSERT_MY_AMOUNT my_amount) (agg_sig CURRIED_PUBKEY all_data))
     )

     (defun-inline assert_id_and_amount_and_sig (CURRIED_PUBKEY all_data my_amount my_id)
        (c ASSERT_MY_ID my_id (assert_amount_and_sig CURRIED_PUBKEY all_data my_amount))
     )

     (assert_id_and_amount_and_sig CURRIED_PUBKEY all_data my_amount my_id)
)
```

This is much more concise, and we can rely on the solver to put the relevant data in a list for us, so we can subtract that cost as well.
This is usually somewhat niche, but it becomes important when trying to pass data down through inner puzzles.
If the outermost puzzle wants to communicate with the innermost puzzle, it will have to pass all the data it needs through every step of the puzzles in between.
That is much easier if it is packaged into a single bundle.

</details>

## 不要反射性地使用函数

通常，使用通用函数可能会成为一种习惯问题，您最终可能会在它实际上造成不必要的复杂性的地方使用它。一个很好的例子是[sha256tree](/docs/common_functions#sha256tree1)。由于该函数适用于 cons 盒子或原子，因此您可能会想在单个原子上使用它（也许您将其柯里化为一个函数）。该函数需要以这种方式工作，因为它会递归并且总是会遇到原子正如它所做的那样。但是，使用它来仅散列一个原子实际上会给程序增加不必要的成本。不仅增加了函数调用开销，而且还添加了检查以查看它是原子还是列表，即使你知道它是一个原子！ 一种更具成本效益的方法是手动散列它，就像在树中散列一样：`(sha256 1 some_atom)`。

<details>
<summary>原文参考</summary>

- ## Don't use functions by reflex

Oftentimes, using a common function can become a matter of habit and you can end up using it where it actually creates more complexity than is necessary.
A good example is [sha256tree](/docs/common_functions#sha256tree1).
Since the function works on either cons boxes or atoms, you may be tempted to use it on a single atom (maybe you're currying it into a function).
The function needs to work this way because it recurses and will always run into atoms as it does so.
However, using it to hash only an atom actually adds unnecessary cost to the program.
Not only do you add the function call overhead, but you also add the check to see if it's an atom or a list, even though you know its an atom!  A more cost effective method is to manually hash it like it would be hashed in a tree: `(sha256 1 some_atom)`.

</details>

## 结论

您可以进行的许多优化可能因为节省的少量成本而显得愚蠢。 但是，如果您希望您的代币被广泛使用，那么每天都会有成千上万的用户为此付费。 随着时间的推移，它可能会浪费很多钱。 在将代码部署到网络之前，花时间检查代码并确保可以节省尽可能多的成本非常重要。

<details>
<summary>原文参考</summary>

- ## Conclusion

A lot of optimizations that you can make may seem silly for the small amount of cost that they save.
However, if you expect your coin to become widely used, then there will be thousands of users paying for that in fees every day.
Over time it can add up to a lot of money wasted.
It's important to take the time to review your code and make sure that you can save as much cost as possible before you deploy it to the network.

</details>
