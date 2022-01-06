---
id: basics
title: 1 - CLVM 基础知识
slug: /
sidebar_label: 1 - CLVM 基础知识
---

> CLVM Basics

CLVM 是 Chia 网络使用的 ChiaLisp 的编译的最小版本。 Chialisp 编译成 CLVM，因此了解它的工作原理很重要。完整的运算符集记录在[此处](/docs/ref/clvm)。

本指南将涵盖该语言的基础知识，并作为程序结构的介绍。您应该能够通过运行 [clvm_tools](https://github.com/Chia-Network/clvm_tools) 的版本来跟进。按照自述文件中的说明进行安装。

<details>
<summary>原文参考</summary>

CLVM is the compiled, minimal version of ChiaLisp that is used by the Chia network.
Chialisp compiles into CLVM so it's important to understand how it works.
The full set of operators is documented [here](/docs/ref/clvm).

This guide will cover the basics of the language and act as an introduction to the structure of programs.
You should be able to follow along by running a version of [clvm_tools](https://github.com/Chia-Network/clvm_tools).
Follow the instructions in the README to install it.

</details>

## CLVM 值

CLVM 由 [cons 框](https://en.wikipedia.org/wiki/Cons) 和[原子](https://www.gnu.org/software/emacs/manual/html_node/eintr/Lisp-Atoms.html#:~:text=Technically%20speaking%2C%20a%20list%20in,nothing%20in%20it%20at%20all.)构建而成。这些被称为 CLVM 对象。cons 框是一对 CLVM 对象。 cons 框中的项目可以是原子或另一个 cons 框。

### 原子

原子是一串字节。这些字节可以解释为有符号大端整数和字节字符串，具体取决于使用它的运算符。

CLVM 中的所有原子都是不可变的。所有对原子执行计算的运算符都会为结果创建新原子。

原子可以用三种不同的方式打印，十进制、十六进制和字符串。十六进制值以 `0x` 为前缀，字符串用 `"` 引用。整数的打印方式不影响其底层值。十进制打印的原子 `100` 与十六进制打印的 `0x64` 相同。同样，值 `0x68656c6c6f` 与 `hello`。

将原子解释为整数时，重要的是要记住它们是有符号的。为了表示一个正整数，最高有效位可能不被设置。因此，正整数前面有一个 0 字节，以防设置下一个字节中的最高有效位。

### Cons 框

Cons 框表示为一个括号，其中两个元素以 `.` 分隔。例如：

```chialisp
(200 . "hello")

("hello" . ("world" . "!!!"))
```

是合法的 cons 框，但以下不是。

```chialisp
(200 . 300 . 400)
```

cons 框总是有两个元素。但是，我们可以将 cons 框链接在一起以构建列表。

<details>
<summary>原文参考</summary>

- ## CLVM values

CLVM is built out of [cons boxes](https://en.wikipedia.org/wiki/Cons) and [atoms](https://www.gnu.org/software/emacs/manual/html_node/eintr/Lisp-Atoms.html#:~:text=Technically%20speaking%2C%20a%20list%20in,nothing%20in%20it%20at%20all.). These are referred to as CLVM Objects.
A cons box is a pair of CLVM Objects. The items in a cons box can either be an atom or another cons box.

- ### Atoms

An atom is a string of bytes. These bytes can be interpreted both as a signed big-endian integer and a byte string, depending on the operator using it.

All atoms in CLVM are immutable. All operators that perform computations on atoms create new atoms for the result.

Atoms can be printed in three different ways, decimal, hexadecimal and as a string. Hexadecimal values are prefixed by `0x`, and strings are quoted in `"`.
The way the integer is printed does not affect its underlying value.
The atom `100` printed in decimal is the same as `0x64` printed in hexadecimal. Likewise the value `0x68656c6c6f` is the same as `"hello"`.

When interpreting atoms as integers, it's important to remember that they are signed. In order to represent a positive integer, the most significant bit may not be set. Because of this, positive integers have a 0 byte prepended to them, in case the most significant bit in the next byte is set.

- ### Cons Boxes

Cons boxes are represented as a parentheses with two elements separated by a `.`.
For example:
```chialisp
(200 . "hello")

("hello" . ("world" . "!!!"))
```
Are legal cons boxes, but the following is not.
```chialisp
(200 . 300 . 400)
```
A cons box always has two elements.
However, we can chain cons boxes together to construct lists.

</details>

## Lists

list 用括号括起来，list 中的每个条目都是单行的，值之间没有句点。列表比 cons 框更常用，因为它们更通用。

```chialisp
(200 300 "hello" "world")
```

您还可以嵌套 list。

```chialisp
("hello" ("nested" "list") ("world"))
```

请记住，list 是以空原子 () 结尾的连续 cons 框的表示。以下表达式是相等的：

```chialisp
(200 . (300 . (400 . ())))

(200 300 400)
```

<details>
<summary>原文参考</summary>

- ## Lists

Lists are enclosed by parentheses and each entry in the list is single spaced with no period between values.
Lists are much more commonly used than cons boxes as they are more versatile.

```chialisp
(200 300 "hello" "world")
```
You can also nest lists.
```chialisp
("hello" ("nested" "list") ("world"))
```

Remember a list is a representation of consecutive cons boxes terminated in a null atom `()`.
The following expressions are equal:
```chialisp
(200 . (300 . (400 . ())))

(200 300 400)
```

</details>

## 引用

要将原子解释为值而不是程序，需要用 `q` 引用它。引用的值形成一个 cons 框，其中第一项是 `q` 运算符。例如，这个程序只是值 `100`：

```chialisp
(q . 100)
```

请注意，在更高级别的 Chialisp 语言中，不需要引用值。

<details>
<summary>原文参考</summary>

- ## Quoting

To interpret an atom as a value, rather than a program, it needs to be quoted with `q`. Quoted values form a cons box where the first item is the `q` operator.
For example, this program is just the value `100`:
```chialisp
(q . 100)
```

Note that in the higher level Chialisp language, values do not need to be quoted.

</details>

## list 和程序

list 是括号内的一个或多个元素的任何以空格分隔的有序组。例如：`(70 80 90 100)`、`(0xf00dbabe 48 "hello")` 和 `(90)` 都是有效 list。

列表甚至可以包含其他列表，例如 `("list" "list" ("sublist" "sublist" ("sub-sublist")) "list")`。 

程序是可以使用 CLVM 评估的列表子集。**一个程序实际上只是一个用[波兰语表示](https://en.wikipedia.org/wiki/Polish_notation)的列表**。

**为了使列表成为有效程序：**

- **1. list 中的第一项必须是有效的运算符**
- **2. 第一个之后的每个项目都必须是一个有效的程序**

规则 2 是文字值和非程序 list *必须* 使用 `q . ` 引用的原因。

```chialisp
$ brun '(q . (80 90 100))'
(80 90 100)
```

现在我们知道我们可以在程序中放置程序，我们可以创建程序，例如：

```chialisp
$ brun '(i (= (q . 50) (q . 50)) (+ (q . 40) (q . 30)) (q . 20))' '()'
70
```
*（更多关于后面使用的运算符）*

CLVM 中的程序倾向于以这种方式构建。较小的程序组合在一起以创建更大的程序。建议您在括号匹配的编辑器中创建您的程序！


<details>
<summary>原文参考</summary>

- ## Lists and Programs

A list is any space-separated, ordered group of one or more elements inside brackets.
For example: `(70 80 90 100)`, `(0xf00dbabe 48 "hello")`, and `(90)` are all valid lists.

Lists can even contain other lists, such as `("list" "list" ("sublist" "sublist" ("sub-sublist")) "list")`.

Programs are a subset of lists which can be evaluated using CLVM. **A program is actually just a list in [polish notation](https://en.wikipedia.org/wiki/Polish_notation).**

**In order for a list to be a valid program:**

- **1. The first item in the list must be a valid operator**
- **2. Every item after the first must be a valid program**

Rule 2 is why literal values and non-program lists *must* be quoted using `q . `.

```chialisp
$ brun '(q . (80 90 100))'
(80 90 100)
```

And now that we know we can have programs inside programs we can create programs such as:

```chialisp
$ brun '(i (= (q . 50) (q . 50)) (+ (q . 40) (q . 30)) (q . 20))' '()'
70
```
*(More on the operators that are used later)*

Programs in CLVM tend to get built in this fashion.
Smaller programs are assembled together to create a larger program.
It is recommended that you create your programs in an editor with brackets matching!

</details>

## list 运算符

`f` 返回传递列表中的第一个元素。

```chialisp
$ brun '(f (q . (80 90 100)))'
80
```

`r` 返回列表中除第一个元素之外的所有元素。

```chialisp
$ brun '(r (q . (80 90 100)))'
(90 100)
```

`c` 将一个元素添加到列表中

```chialisp
$ brun '(c (q . 70) (q . (80 90 100)))'
(70 80 90 100)
```

我们可以使用这些组合来访问或替换列表中我们想要的任何元素：


```chialisp
$ brun '(c (q . 100) (r (q . (60 110 120))))'
(100 110 120)

$ brun '(f (r (r (q . (100 110 120 130 140)))))'
120
```

<details>
<summary>原文参考</summary>

- ## List Operators

`f` returns the first element in a passed list.

```chialisp
$ brun '(f (q . (80 90 100)))'
80
```

`r` returns every element in a list except for the first.

```chialisp
$ brun '(r (q . (80 90 100)))'
(90 100)
```

`c` prepends an element to a list

```chialisp
$ brun '(c (q . 70) (q . (80 90 100)))'
(70 80 90 100)
```

And we can use combinations of these to access or replace any element we want from a list:

```chialisp
$ brun '(c (q . 100) (r (q . (60 110 120))))'
(100 110 120)

$ brun '(f (r (r (q . (100 110 120 130 140)))))'
120
```

</details>

## 数学

CLVM 中不支持浮点数，仅支持整数。 CLVM 中的整数没有硬性大小限制。也支持负值。

数学运算符是`+`、`-`、`*` 和`/`。

```chialisp
$ brun '(- (q . 6) (q . 5))'
1

$ brun '(* (q . 2) (q . 4) (q . 5))'
40

$ brun '(+ (q . 10) (q . 20) (q . 30) (q . 40))'
100

$ brun '(/ (q . 20) (q . 11))'
1
```

*注意 `/`返回**落地**商。 CLVM 也不同于大多数语言，因为它下限为负无穷大而不是零。在尝试对负数进行除法时，这可能会产生一些意想不到的结果。*


```chialisp
brun '(/ (q . 3) (q . 2))'
1

brun '(/ (q . 3) (q . -2))'
-2

brun '(/ (q . -3) (q . 2))'
-2

brun '(/ (q . -3) (q . -2))'
1
```

您可能已经注意到，上面的乘法示例在 list 中采用了两个以上的参数。 这是因为许多运算符可以采用可变数量的参数。 `+` 和 `*` 是可交换的，因此参数的顺序无关紧要。 对于非交换操作，`(- 100 30 20 5)` 等价于`(- 100 (+ 30 20 5))`。 类似地，`(/ 120 5 4 2)` 等价于`(/ 120 (* 5 4 2))`。

```chialisp
$ brun '(- (q . 5) (q . 7))'
-2


$ brun '(+ (q . 3) (q . -8))'
-5
```

要使用十六进制数字，只需在它们前面加上 `0x`。

```chialisp
$ brun '(+ (q . 0x000a) (q . 0x000b))'
21
```

最后的数学运算符是相等的，它的作用类似于其他语言中的 ==。

```chialisp
$ brun '(= (q . 5) (q . 6))'
()

$ brun '(= (q . 5) (q . 5))'
1
```

正如你在上面看到的，这种语言将一些数据解释为布尔值。

<details>
<summary>原文参考</summary>

- ## Math

There are no support for floating point numbers in CLVM, only integers. There is no hard size limit on integers in CLVM. There is also support for negative values.

The math operators are `+`, `-`, `*`, and `/`.

```chialisp
$ brun '(- (q . 6) (q . 5))'
1

$ brun '(* (q . 2) (q . 4) (q . 5))'
40

$ brun '(+ (q . 10) (q . 20) (q . 30) (q . 40))'
100

$ brun '(/ (q . 20) (q . 11))'
1
```

*Note that `/` returns the* ***floored*** *quotient. CLVM is also different from most languages in that it floors to negative infinity rather than zero.
This can create some unexpected results when trying to divide negative numbers.*

```chialisp
brun '(/ (q . 3) (q . 2))'
1

brun '(/ (q . 3) (q . -2))'
-2

brun '(/ (q . -3) (q . 2))'
-2

brun '(/ (q . -3) (q . -2))'
1
```

You may have noticed that the multiplication example above takes more than two parameters in the list.
This is because many operators can take a variable number of parameters.
`+` and `*` are commutative so the order of parameters does not matter.
For non-commutative operations, `(- 100 30 20 5)` is equivalent to `(- 100 (+ 30 20 5))`.
Similarly, `(/ 120 5 4 2)` is equivalent to `(/ 120 (* 5 4 2))`.

```chialisp
$ brun '(- (q . 5) (q . 7))'
-2


$ brun '(+ (q . 3) (q . -8))'
-5
```

To use hexadecimal numbers, simply prefix them with `0x`.

```chialisp
$ brun '(+ (q . 0x000a) (q . 0x000b))'
21
```

The final mathematical operator is equal which acts similarly to == in other languages.

```chialisp
$ brun '(= (q . 5) (q . 6))'
()

$ brun '(= (q . 5) (q . 5))'
1
```

As you can see above this language interprets some data as boolean values.

</details>

## 布尔值

在这种语言中，空列表 `()` 的计算结果为 `False`。任何其他值的计算结果为 `True`，尽管在内部 `True` 表示为 `1`。

```chialisp
$ brun '(= (q . 100) (q . 90))'
()

$ brun '(= (q . 100) (q . 100))'
1
```

此规则的例外是 `0` ，因为 `0` 与 `()` 完全相同。


```chialisp
$ brun '(= (q . 0) ())'
1

$ brun '(+ (q . 70) ())'
70
```

<details>
<summary>原文参考</summary>

- ## Booleans

In this language an empty list `()` evaluate to `False`.
Any other value evaluates to `True`, though internally `True` is represented with `1`.

```chialisp
$ brun '(= (q . 100) (q . 90))'
()

$ brun '(= (q . 100) (q . 100))'
1
```

The exception to this rule is `0` because `0` is  exactly the same as `()`.

```chialisp
$ brun '(= (q . 0) ())'
1

$ brun '(+ (q . 70) ())'
70
```

</details>


## 流量控制

`i` 操作符采用 `(i A B C)` 的形式，并作为一个 if 语句，如果 `A` 为真，则求值为 `B`，否则求值为 `C`。

```chialisp
$ brun '(i (q . 0) (q . 70) (q . 80))'
80

$ brun '(i (q . 1) (q . 70) (q . 80))'
70

$ brun '(i (q . 12) (q . 70) (q . 80))'
70

$ brun '(i () (q . 70) (q . 80))'
80
```

请注意，就像所有子表达式一样，`B` 和 `C` 都被急切地求值。 要将评估推迟到条件之后，必须引用 `B` 和 `C`（使用 `q`），然后使用 `(a)` 进行评估。

```chialisp
$ brun '(a (q . (i (q . 0) (q . (x (q . 1337) )) (q . (q . 1)))) ())'
```

稍后会详细介绍。

<details>
<summary>原文参考</summary>

- ## Flow Control

The `i` operator takes the form `(i A B C)` and acts as an if-statement that
evaluates to `B` if `A` is True and `C` otherwise.
```chialisp
$ brun '(i (q . 0) (q . 70) (q . 80))'
80

$ brun '(i (q . 1) (q . 70) (q . 80))'
70

$ brun '(i (q . 12) (q . 70) (q . 80))'
70

$ brun '(i () (q . 70) (q . 80))'
80
```

Note that both `B` and `C` are evaluated eagerly, just like all subexpressions.
To defer evaluation until after the condition, `B` and `C` must be quoted (with
`q`), and then evaluated with `(a)`.

```chialisp
$ brun '(a (q . (i (q . 0) (q . (x (q . 1337) )) (q . (q . 1)))) ())'
```

More on this later.

</details>

## 环境变量

到目前为止，我们的程序还没有任何输入或变量，但是 CLVM 确实支持这一点。

环境是传递给谜语的值列表。您只能传递一个 CLVM 对象作为环境，但该对象可以是任意长的列表。可以使用不带引号的 `1` 来引用整个环境。

```chialisp
$ brun '1' '("this" "is the" "environement")'
("this" "is the" "environment")

$ brun '(f 1)' '(80 90 100 110)'
80

$ brun '(r 1)' '(80 90 100 110)'
(90 100 110)
```

记住列表也可以嵌套。

```chialisp
$ brun '(f (f (r 1)))' '((70 80) (90 100) (110 120))'
90

$ brun '(f (f (r 1)))' '((70 80) ((91 92 93 94 95) 100) (110 120))'
(91 92 93 94 95)
```

这些环境变量可以与所有其他运算符结合使用。

```chialisp
$ brun '(+ (f 1) (q . 5))' '(10)'
15

$ brun '(* (f 1) (f 1))' '(10)'
100
```

该程序检查第二个变量是否等于第一个变量的平方。


```chialisp
$ brun '(= (f (r 1)) (* (f 1) (f 1)))' '(5 25)'
1

$ brun '(= (f (r 1)) (* (f 1) (f 1)))' '(5 30)'
()
```

<details>
<summary>原文参考</summary>

- ## Environment Variables

Up until now our programs have not had any input or variables, however CLVM does have support for this.

An environment is a list of values passed to the puzzle.
You can only pass a single CLVM object as the environment, but this object can be an arbitrarily long list.
The entire environment can be referenced with an unquoted `1`.

```chialisp
$ brun '1' '("this" "is the" "environement")'
("this" "is the" "environment")

$ brun '(f 1)' '(80 90 100 110)'
80

$ brun '(r 1)' '(80 90 100 110)'
(90 100 110)
```

And remember lists can be nested too.

```chialisp
$ brun '(f (f (r 1)))' '((70 80) (90 100) (110 120))'
90

$ brun '(f (f (r 1)))' '((70 80) ((91 92 93 94 95) 100) (110 120))'
(91 92 93 94 95)
```

These environment variables can be used in combination with all other operators.

```chialisp
$ brun '(+ (f 1) (q . 5))' '(10)'
15

$ brun '(* (f 1) (f 1))' '(10)'
100
```

This program checks that the second variable is equal to the square of the first variable.

```chialisp
$ brun '(= (f (r 1)) (* (f 1) (f 1)))' '(5 25)'
1

$ brun '(= (f (r 1)) (* (f 1) (f 1)))' '(5 30)'
()
```

</details>

## 通过整数访问环境变量

在上面的例子中，我们只使用了 `1` ，它访问树的根并返回整个谜底列表。

```chialisp
$ brun '1' '("example" "data" "for" "test")'
("example" "data" "for" "test")
```

但是，低级语言中的每个未加引号的整数都指代谜底的一部分。

你可以想象一个由 `f` 和 `r` 组成的二叉树，其中每个节点都有编号：


```
              1
             / \
            /   \
           /     \
          /       \
         /         \
        /           \
       2             3
      / \           / \
     /   \         /   \
    4      6      5     7
   / \    / \    / \   / \
  8   12 10  14 9  13 11  15

etc.
```

```chialisp
$ brun '2' '("example" "data" "for" "test")'
"example"

$ brun '3' '("example" "data" "for" "test")'
("data" "for" "test")

$ brun '5' '("example" "data" "for" "test")'
"data"
```

这被设计为在列表中也有列表时工作。

```chialisp
$ brun '4' '(("deeper" "example") "data" "for" "test")'
"deeper"

$ brun '5' '(("deeper" "example") "data" "for" "test")'
"data"

$ brun '6' '(("deeper" "example") "data" "for" "test")'
("example")
```

等等。

<details>
<summary>原文参考</summary>

- ## Accessing Environmental Variables Through Integers

In the above examples we only used `1` which access the root of the tree and returns the entire solution list.

```chialisp
$ brun '1' '("example" "data" "for" "test")'
("example" "data" "for" "test")
```

However, every unquoted integer in the lower level language refers to a part of the solution.

You can imagine a binary tree of `f` and `r`, where each node is numbered:

```
              1
             / \
            /   \
           /     \
          /       \
         /         \
        /           \
       2             3
      / \           / \
     /   \         /   \
    4      6      5     7
   / \    / \    / \   / \
  8   12 10  14 9  13 11  15

etc.
```

```chialisp
$ brun '2' '("example" "data" "for" "test")'
"example"

$ brun '3' '("example" "data" "for" "test")'
("data" "for" "test")

$ brun '5' '("example" "data" "for" "test")'
"data"
```

And this is designed to work when there are lists inside lists too.

```chialisp
$ brun '4' '(("deeper" "example") "data" "for" "test")'
"deeper"

$ brun '5' '(("deeper" "example") "data" "for" "test")'
"data"

$ brun '6' '(("deeper" "example") "data" "for" "test")'
("example")
```

And so on.

</details>

## 结论

这标志着本指南的这一部分结束。在本节中，我们已经介绍了使用 CLVM 的许多基础知识。建议您在继续之前使用此处提供的信息进行一些尝试。

本指南并未涵盖 CLVM 中可用的所有运算符——尝试使用[此处](/docs/ref/clvm) 列出的其他一些运算符！

<details>
<summary>原文参考</summary>

- ## Conclusion

This marks the end of this section of the guide.
In this section we have covered many of the basics of using CLVM.
It is recommended you play with using the information presented here for a bit before moving on.

This guide has not covered all of the operators available in CLVM - try using some of the other ones listed [here](/docs/ref/clvm)!

</details>
