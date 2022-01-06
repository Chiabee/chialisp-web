---
id: deeper_into_clvm
title: 3 - 深入了解 CLVM
---

> Deeper into CLVM

本指南假定您了解[CLVM 基础知识](/docs/)，因此如果您还没有阅读，请在阅读前先阅读。

本指南的这一部分将介绍 ChiaLisp 如何与 Chia 网络上的交易和硬币相关联。如果有任何您不确定的术语，请务必查看[术语表](/docs/glossary)。


<details>
<summary>原文参考</summary>

This guide assumes knowledge of [the basics of CLVM](/docs/) so if you haven't read that, please do so before reading this.

This section of the guide will cover how ChiaLisp relates to transactions and coins on the Chia network.
If there are any terms that you aren't sure of, be sure to check the [glossary](/docs/glossary).

</details>

## ChiaLisp 中的惰性求值

正如我们在前面的章节中看到的，程序通常围绕 `(i A B C)` 来构建以控制流程。 ChiaLisp 将程序评估为树，首先评估叶子。 如果您不知道，这可能会导致意外问题。 考虑以下使用 `x` 的程序，如果它被评估，它会立即停止并抛出错误。

```chialisp
$ brun '(i (q . 1) (q . 100) (x (q . "still being evaluated")))'
FAIL: clvm raise (0x7374696c6c206265696e67206576616c7561746564)
```

这是因为 ChiaLisp 评估两个叶子，即使它只会遵循一个的路径。

为了解决这个问题，我们可以使用以下设计模式来替换 (i A B C)。

```chialisp
(a (i (A) (q . B) (q . C)) 1)
```

将其应用于我们上面的示例，如下所示：

```chialisp
$ brun '(a (i (q . 1) (q . (q . 100)) (q . (x (q . "still being evaluated")))) 1)'
100
```

每当您编写 `(i A B C)` 时，都值得牢记这一点。

如果您想知道这是如何工作的（以及之前的[签名锁定币](/docs/coins_spends_and_wallets#example-signature-locked-coin)是如何工作的），那么请允许我介绍求值。

<details>
<summary>原文参考</summary>

- ## Lazy Evaluation in ChiaLisp

As we've seen in earlier sections, programs are often structured around `(i A B C)` to control flow.
ChiaLisp evaluates programs as trees, where the leaves are evaluated first.
This can cause unexpected problems if you are not aware of it.
Consider the following program which uses `x` which immediately halts and throws an error if it is evaluated.

```chialisp
$ brun '(i (q . 1) (q . 100) (x (q . "still being evaluated")))'
FAIL: clvm raise (0x7374696c6c206265696e67206576616c7561746564)
```

This is because ChiaLisp evaluates both of the leaves even though it will only follow the path of one.

To get around this we can use the following design pattern to replace (i A B C).

```chialisp
(a (i (A) (q . B) (q . C)) 1)
```

Applying this to our above example looks like this:

```chialisp
$ brun '(a (i (q . 1) (q . (q . 100)) (q . (x (q . "still being evaluated")))) 1)'
100
```

It is worth keeping this in mind whenever you write an `(i A B C)`.

If you're wondering how this works (and how the [signature locked coin](/docs/coins_spends_and_wallets#example-signature-locked-coin) from before worked), then allow me to introduce Evaluate.

</details>

## 求值简介

在 [CLVM 的介绍](/docs/)中我们提到程序通常是一个列表，其中第一个元素是一个运算符，每个后续元素都是一个有效的程序。我们还可以在程序中运行带有新参数的程序。

看起来像这样：

```chialisp
(a *puzzle* *solution*)
```

让我们付诸实践。

这是一个求值程序 `(+ 2 (q . 5)))` 并使用列表 `(70 80 90)` 或 `(80 90 100)` 作为谜底的程序。


```chialisp
$ brun '(a (q . (+ 2 (q . 5))) (q . (70 80 90)))' '(20 30 40)'
75

$ brun '(a (q . (+ 2 (q . 5))) (q . (80 90 100)))' '(20 30 40)'
85
```

注意原始谜底 `(20 30 40)` 对于新的求值环境如何无关紧要。在这个例子中，我们使用 `q . ` 引用新的谜语和新的谜底，以防止过早地对其进行求值。

我们可以使用的一个巧妙技巧是我们可以根据外部谜底定义新的谜底。在下一个示例中，我们将旧谜底的第一个元素添加到我们的新谜底中。

```chialisp
$ brun '(a (q . (+ 2 (q . 5))) (c 2 (q . (70 80 90))))' '(20 30 40)'
25
```

然而，我们不仅可以影响使用它的新谜底，还可以将程序作为参数传递。

<details>
<summary>原文参考</summary>

- ## Introduction to Evaluate

In [the introduction to CLVM](/docs/) we mentioned that a program is usually a list where the first element is an operator, and every subsequent element is a valid program.
We can also run programs with new arguments inside a program.

This looks like this:

```chialisp
(a *puzzle* *solution*)
```

Let's put this into practice.

Here is a program that evaluates the program `(+ 2 (q . 5)))` and uses the list `(70 80 90)` or `(80 90 100)` as the solution.

```chialisp
$ brun '(a (q . (+ 2 (q . 5))) (q . (70 80 90)))' '(20 30 40)'
75

$ brun '(a (q . (+ 2 (q . 5))) (q . (80 90 100)))' '(20 30 40)'
85
```

Notice how the original solution `(20 30 40)` does not matter for the new evaluation environment.
In this example we use `q . ` to quote both the new puzzle and the new solution to prevent them from being prematurely evaluated.

A neat trick that we can pull is that we can define the new solution in terms of the outer solution.
In this next example we will add the first element of the old solution to our new solution.

```chialisp
$ brun '(a (q . (+ 2 (q . 5))) (c 2 (q . (70 80 90))))' '(20 30 40)'
25
```

However it's not just the new solution that we can affect using this, we can also pass programs as parameters.

</details>

## 作为参数的程序

核心 CLVM 没有用于创建用户定义函数的运算符。但是，它允许将程序作为参数传递，这可以用于类似的结果。

这是一个谜语，它使用解 `(12)` 执行 `2` （第一个解参数）中包含的程序。

```chialisp
$ brun '(a 2 (q . (12)))' '((* 2 (q . 2)))'
24
```

更进一步，我们可以让谜语运行一个新的评估，只使用旧谜底中的参数：

```chialisp
$ brun '(a 2 1)' '((* 5 (q . 2)) 10)'
20
```

我们可以使用这种技术来实现递归程序。

<details>
<summary>原文参考</summary>

- ## Programs as Parameters

The core CLVM does not have an operator for creating user defined functions.
It does, however, allow programs to be passed as parameters, which can be used for similar results.

Here is a puzzle that executes the program contained in `2` (the first solution argument) with the solution `(12)`.

```chialisp
$ brun '(a 2 (q . (12)))' '((* 2 (q . 2)))'
24
```

Taking this further we can make the puzzle run a new evaluation that only uses parameters from its old solution:

```chialisp
$ brun '(a 2 1)' '((* 5 (q . 2)) 10)'
20
```

We can use this technique to implement recursive programs.

</details>
