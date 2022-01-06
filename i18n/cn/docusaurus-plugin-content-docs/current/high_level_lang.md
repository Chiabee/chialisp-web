---
id: high_level_lang
title: 4 - 高级语言、编译器和函数
---

> The High Level Language, Compiler, and Functions

本指南假设您已经阅读了前面的部分。强烈建议您这样做，因为高级语言直接构建在低级语言之上。

<details>
<summary>原文参考</summary>

This guide assumes that you have already read the previous parts.
It is highly recommended that you do so as the higher level language is built directly on top of the lower level language.

</details>

## CLVM vs Chialisp

到目前为止，我们一直在使用我们所说的 CLVM 来编写我们的程序。 CLVM 是序列化并直接存储在区块链上的内容，是一个共识问题。 它永远无法改变。

通常，我们会编写一种称为 **Chialisp** 的高级语言而不是 CLVM。 Chialisp 编译成 CLVM，由于它是先经过编译器，所以实际上可以更改它以添加更多功能。Chialisp 分享了程序如何与 CLVM 一起工作的许多基础知识，但也包括一些有用的功能，使编写大型程序更容易。

<details>
<summary>原文参考</summary>

- ## CLVM vs Chialisp

Until now, we have been using what we call CLVM to write our programs.
CLVM is what is serialized and stored directly on the blockchain and is a matter of consensus. It can never be changed.

Normally, we will write a higher level language called **Chialisp** instead of CLVM.
Chialisp compiles into CLVM, and since it is going through the compiler first, it can actually be changed to add more features.
Chialisp shares a lot of the fundamentals of how programs work with CLVM, but also includes some helpful functionality to make writing large programs easier.

</details>

## Run

对于高级语言，您需要注意的第一个区别是您应该调用 `run` 而不是 `brun`。 这让运行时知道它应该包含更高级别的运算符和行为。

您应该注意的第一个更高级别的功能是**不再需要引用原子！**

在这里比较 `brun` 和 `run`：

```chialisp
$ brun '(+ 200 200)'
FAIL: first of non-cons ()
$ run '(+ 200 200)'
400
```

Run 还使我们可以访问许多方便的高级运算符，我们现在将介绍这些运算符。

<details>
<summary>原文参考</summary>

- ## Run

The first difference you need to be aware of for the higher level language is that you should call `run` instead of `brun`.
This lets the runtime know that it should be including higher level operators and behavior.

The first higher level feature you should be aware of is that **it is no longer necessary to quote atoms!**

Compare `brun` and `run` here:

```chialisp
$ brun '(+ 200 200)'
FAIL: first of non-cons ()
$ run '(+ 200 200)'
400
```

Run also gives us access to a number of convenient high level operators, which we will cover now.

</details>

## list

`list` 接受任意数量的参数并将它们放在一个列表中。 这使我们不必手动创建嵌套的 `(c (A) (c (B) (q ())))` 调用，这会很快变得混乱。


```chialisp
$ run '(list 100 "test" 0xdeadbeef)'
(100 "test" 0xdeadbeef)
```

<details>
<summary>原文参考</summary>

- ## list

`list` takes any number of parameters and returns them put inside a list.
This saves us from having to manually create nested `(c (A) (c (B) (q ())))` calls, which can get messy quickly.

```chialisp
$ run '(list 100 "test" 0xdeadbeef)'
(100 "test" 0xdeadbeef)
```

</details>

## if

`if` 自动将我们的 `i` 语句放入惰性求值表中，因此我们无需担心正在求值的未使用的代码路径。

```chialisp
$ run '(if 1 (q . "success") (x))' '(100)'
"success"

$ run '(if 0 (q . "success") (x))' '(100)'
FAIL: clvm raise ()
```

<details>
<summary>原文参考</summary>

- ## if

`if` automatically puts our `i` statement into the lazy evaluation form so we do not need to worry about the unused code path being evaluated.

```chialisp
$ run '(if 1 (q . "success") (x))' '(100)'
"success"

$ run '(if 0 (q . "success") (x))' '(100)'
FAIL: clvm raise ()
```

</details>

## qq 取消引用

`qq` 允许我们使用 `unquote` 引用某些内容，其中选定的部分在内部进行评估。这样做的优点可能不是很明显，但在实践中非常有用，因为它允许我们替换预定代码的部分。

假设我们正在编写一个返回另一个硬币谜语的程序。我们知道谜语的形式是：`(c (c (q . 50) (c (q . 0xpubkey) (c (sha256 2) (q . ())))) (a 5 11))` 然而我们将希望把 0xpubkey 更改为通过我们的谜底传递给我们的值。

**注意：`@` 允许我们访问高级语言中的参数（`@` == 1）**

```chialisp
$ run '(qq (c (c (q . 50) (c (q (unquote (f @))) (c (sha256 2) ()))) (a 5 11)))' '(0xdeadbeef)'

(c (c (q . 50) (c (q . 0xdeadbeef) (c (sha256 2) ()))) (a 5 11))
```

<details>
<summary>原文参考</summary>

- ## qq unquote

`qq` allows us to quote something with selected portions being evaluated inside by using `unquote`.
The advantages of this may not be immediately obvious but are extremely useful in practice as it allows us to substitute out sections of predetermined code.

Suppose we are writing a program that returns another coin's puzzle.
We know that a puzzle takes the form: `(c (c (q . 50) (c (q . 0xpubkey) (c (sha256 2) (q . ())))) (a 5 11))`
However we will want to change 0xpubkey to a value passed to us through our solution.

**Note: `@` allows us to access the arguments in the higher level language (`@` == 1)**

```chialisp
$ run '(qq (c (c (q . 50) (c (q (unquote (f @))) (c (sha256 2) ()))) (a 5 11)))' '(0xdeadbeef)'

(c (c (q . 50) (c (q . 0xdeadbeef) (c (sha256 2) ()))) (a 5 11))
```

</details>

## 使用 Mod 编译为 CLVM

重要的是要记住，在实践中智能币将使用较低级别的语言运行，因此上述操作员都不会在网络上工作。 然而，我们*可以*做的是将它们编译成较低级别的语言。 这就是 `mod` 的用武之地。 `mod` 是一个操作符，它让运行时知道它需要编译代码而不是实际运行它。

`(mod A B)` 需要两个或多个参数。 第一个用于命名传入的参数，最后一个是要编译的更高级别的脚本。

下面我们将参数命名为 `arg_one` 和 `arg_two` ，然后在主程序中访问 `arg_one`。

```chialisp
$ run '(mod (arg_one arg_two) (list arg_one))'
(c 2 ())
```

如您所见，它以编译后的低级形式返回我们的程序。

```chialisp
$ brun '(c 2 ())' '(100 200 300)'
(100)
```

您可能想知道在变量名称和源代码之间，`mod` 还采用了哪些其他参数。

<details>
<summary>原文参考</summary>

- ## Compiling to CLVM with Mod

It is important to remember that in practice smart coins will run using the lower level language, so none of the above operators will work on the network.
What we *can* do however is compile them down to the lower level language.
This is where `mod` comes in.
`mod` is an operator that lets the runtime know that it needs to be compiling the code rather than actually running it.

`(mod A B)` takes two or more parameters. The first is used to name parameters that are passed in, and the last is the higher level script which is to be compiled.

Below we name our arguments `arg_one` and `arg_two` and then access `arg_one` inside our main program

```chialisp
$ run '(mod (arg_one arg_two) (list arg_one))'
(c 2 ())
```

As you can see it returns our program in compiled lower level form.

```chialisp
$ brun '(c 2 ())' '(100 200 300)'
(100)
```

You may be wondering what other parameters `mod` takes, between variable names and source code.

</details>

## 函数、宏和常量

在高级语言中，我们可以在我们的程序之前使用 `defun`、`defun-inline`、`defmacro` 和 `defconstant` 定义函数、宏和常量。

我们可以在主要源代码之前定义任意数量的这些。通常一个程序的结构是这样的：

```chialisp
(mod (arg_one arg_two)
  (defconstant const_name value)
  (defun function_name (parameter_one parameter_two) *function_code*)
  (defun another_function (param_one param_two param_three) *function_code*)
  (defun-inline utility_function (param_one param_two) *function_code*)
  (defmacro macro_name (param_one param_two) *macro_code*)

  (main *program*)
)
```

需要注意的几点：

- 函数可以在它们的代码中引用自己，但宏和内联不能，因为它们是在编译时插入的。
- 函数和宏都可以引用其他函数、宏和常量。
- 引用其参数的宏必须用不带引号的参数进行准引号
- 小心引用其他宏的宏中的无限循环。
- 评论可以用分号书写。
- 内联函数通常比常规函数更具成本效益，除非重用计算参数：`(defun-inline foo (X) (+ X X)) (foo (* 200 300))` 将执行两次昂贵的乘法。

<details>
<summary>原文参考</summary>

- ## Functions, Macros and Constants

In the higher level language we can define functions, macros, and constants before our program by using `defun`, `defun-inline`, `defmacro` and `defconstant`.

We can define as many of these as we like before the main source code.
Usually a program will be structured like this:

```chialisp
(mod (arg_one arg_two)
  (defconstant const_name value)
  (defun function_name (parameter_one parameter_two) *function_code*)
  (defun another_function (param_one param_two param_three) *function_code*)
  (defun-inline utility_function (param_one param_two) *function_code*)
  (defmacro macro_name (param_one param_two) *macro_code*)

  (main *program*)
)
```

A few things to note:

- Functions can reference themselves in their code but macros and inlines cannot as they are inserted at compile time.
- Both functions and macros can reference other functions, macros and constants.
- Macros that refer to their parameters must be quasiquoted with the parameters unquoted
- Be careful of infinite loops in macros that reference other macros.
- Comments can be written with semicolons
- Inline functions are generally more cost effective than regular functions except when reusing calculated arguments: `(defun-inline foo (X) (+ X X)) (foo (* 200 300))` will perform the expensive multiplication twice

</details>

## 阶乘

```chialisp
(mod (arg_one)
  ; function definitions
  (defun factorial (input)
    (if (= input 1) 1 (* (factorial (- input 1)) input))
  )

  ; main
  (factorial arg_one)
)
```

我们可以将这些文件保存为可以从命令行运行的 .clvm 文件。 将上面的示例保存为 `factorial.clvm` 允许我们执行以下操作。

```chialisp
$ run factorial.clvm
(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i (= 5 (q . 1)) (q 1 . 1) (q 18 (a 2 (c 2 (c (- 5 (q . 1)) ()))) 5)) 1) 1))

$ brun '(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i (= 5 (q . 1)) (q 1 . 1) (q 18 (a 2 (c 2 (c (- 5 (q . 1)) ()))) 5)) 1) 1))' '(5)'
120
```

<details>
<summary>原文参考</summary>

- ## Factorial

```chialisp
(mod (arg_one)
  ; function definitions
  (defun factorial (input)
    (if (= input 1) 1 (* (factorial (- input 1)) input))
  )

  ; main
  (factorial arg_one)
)
```

We can save these files to .clvm files which can be run from the command line.
Saving the above example as `factorial.clvm` allows us to do the following.

```chialisp
$ run factorial.clvm
(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i (= 5 (q . 1)) (q 1 . 1) (q 18 (a 2 (c 2 (c (- 5 (q . 1)) ()))) 5)) 1) 1))

$ brun '(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i (= 5 (q . 1)) (q 1 . 1) (q 18 (a 2 (c 2 (c (- 5 (q . 1)) ()))) 5)) 1) 1))' '(5)'
120
```

</details>

## 对列表进行平方

现在让我们做一个使用宏的例子。编写宏时，必须用不带引号的参数对它进行准引号。

我们也可以借此机会展示一下编译器的另一个特性。您可以命名列表中的每个参数，也可以命名列表本身。这适用于您命名参数的任何地方，并允许您处理不确定大小的列表。

这里我们定义了一个宏来平方参数，然后定义一个函数来平方列表。

```chialisp
(mod (args)

  (defmacro square (input)
    (qq (* (unquote input) (unquote input)))
  )

  (defun sqre_list (my_list)
    (if my_list
      (c (square (f my_list)) (sqre_list (r my_list)))
      my_list
    )
  )

  (sqre_list args)
)
```

编译并运行此代码会导致：

```chialisp
$ run square_list.clvm
(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i 5 (q 4 (* 9 9) (a 2 (c 2 (c 13 ())))) (q . 5)) 1) 1))

$ brun '(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i 5 (q 4 (* 9 9) (a 2 (c 2 (c 13 ())))) (q . 5)) 1) 1))' '((10 9 8 7))'
(100 81 64 49)
```

<details>
<summary>原文参考</summary>

- ## Squaring a List

Now lets do an example which uses macros as well.
When writing a macro it must be quasiquoted with the parameters being unquoted.

We can also take this time to show another feature of the compiler.
You can name each parameter in a list or you can name the list itself.
This works at any place where you name parameters, and allows you to handle lists where you aren't sure of the size.

Here we define a macro to square a parameter and then a function to square a list.

```chialisp
(mod (args)

  (defmacro square (input)
    (qq (* (unquote input) (unquote input)))
  )

  (defun sqre_list (my_list)
    (if my_list
      (c (square (f my_list)) (sqre_list (r my_list)))
      my_list
    )
  )

  (sqre_list args)
)
```

Compiling and running this code results in this:

```chialisp
$ run square_list.clvm
(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i 5 (q 4 (* 9 9) (a 2 (c 2 (c 13 ())))) (q . 5)) 1) 1))

$ brun '(a (q 2 2 (c 2 (c 5 ()))) (c (q 2 (i 5 (q 4 (* 9 9) (a 2 (c 2 (c 13 ())))) (q . 5)) 1) 1))' '((10 9 8 7))'
(100 81 64 49)
```

</details>

## 结论

您现在应该拥有编写自己的 Chialisp 程序所需的上下文和知识。从[我们对硬币的讨论](/docs/coins_spends_and_wallets/)中记住，这些程序在区块链上运行并指示区块链如何处理硬币的价值。

如果您有其他问题，请随时在 [Keybase](https://keybase.io/team/chia_network.public) 上提问。

<details>
<summary>原文参考</summary>

- ## Conclusion

You should now have the context and knowledge needed to write your own Chialisp programs.
Remember from [our discussion of coins](/docs/coins_spends_and_wallets/) that these programs run on the blockchain and instruct the blockchain what to do with the coin's value.

If you have further questions feel free to ask on [Keybase](https://keybase.io/team/chia_network.public).

</details>
