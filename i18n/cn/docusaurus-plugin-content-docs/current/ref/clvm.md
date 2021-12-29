---
id: clvm
title: CLVM 参考手册
sidebar_label: CLVM 参考
---

> CLVM Reference Manual

clvm 是一个小型的、严格定义的 VM，它定义了在 Chia 区块链验证期间运行的 CLVM 程序的语义。它作为高级语言的目标语言，尤其是 ChiaLisp。

<details>
<summary>原文参考</summary>

The clvm is a small, tightly defined VM that defines the semantics of CLVM programs run during Chia blockchain validation. It serves as a target language for higher level languages, especially ChiaLisp.

</details>

## 定义

* **CLVM 程序集** - CLVM 程序的文本表示。
* **CLVM Bytecode** - CLVM 程序的序列化形式。
* **ChiaLisp** - 一种建立在 CLVM 之上的高级语言。
* **CLVM Object** - CLVM 中的底层数据类型。一个原子或一个 cons 对。
* **Atom** - CLVM 中值的数据类型。原子是不可变的字节数组。原子是无类型的，用于对所有字符串、整数和键进行编码。 CLVM 中唯一不是原子的东西是 cons 对。 Atom 属性是长度和原子中的字节数。
* **cons pair** - 对其他 CLVM 对象的不可变的有序引用对。 CLVM 中的两种数据类型之一。 cons 对的语法是点对。也称为“cons cell”或“cons box”。
* **slot** - cons 盒子中的单元格之一。右或左。使用`f`（第一）或`r`（其余）访问。
* **nil** - nil 是由零长度字节数组表示的特殊值。此值表示零、空字符串、false 和空列表。 Nil 在 CLVM 程序集中表示为 `()`、`0` 或 `''`。
* **Value** - 我们使用 value 来表示抽象值，例如 `1`（整数）、`0xCAFE`（字节字符串）、`"hello"`（字符串）或 `(sha256 (q ."你好"))`（一个程序）。值由 CLVM 对象表示。
* **List** - CLVM 列表遵循 lisp 约定，即包含左侧插槽中的第一个列表元素和右侧插槽中的其余列表元素的 cons 对。
* **正确列表** - “正确”列表是一连串 cons 框，每个框的左侧插槽中包含一个值。每个正确的插槽包含另一个缺点框，如果是最后一对，则为 nil。
* **函数** - CLVM 中的函数要么是内置操作码，要么是用户定义的程序。
* **Operator** - 指定要使用的内置函数的操作码/字符串。
* **程序** - 可以执行的 CLVM 对象。
* **Opcodes** - 与保留关键字对应的原子。当在第一个位置使用预定义的操作码评估列表时，将运行该操作码的代码。
* **Keyword** - CLVM 汇编语言语法中的保留字。 CLVM 用于函数查找的字符串。
* **树** - 通过允许 cons 对的左右单元格容纳一个原子或一个 cons 对，可以从 cons 对和原子形成二叉树。原子是树的叶子。
* **函数参数** - 对列表求值时，第一个参数是函数，其他项是参数。在程序`(+ (q . 1) (q . 2))`中，引用的原子`1`和`2`是运算符`+`的参数
* **Treearg** - 这些是从程序外部传入的程序参数。它们由描述参数树中路径的整数引用。
* **参数** - 在 CLVM 的上下文之外，术语“参数”可以表示“程序参数”（例如，C 语言的“argv”）或“函数参数”等。由于这种潜在的混淆，我们避免在本文档中使用术语“参数”。考虑到 CLVM 程序查找其程序参数的方式，上下文尤其重要。

CLVM 程序必须具有明确的定义和含义，以便 Chia 块验证和共识具有确定性。程序被视为默克尔树，由其根的哈希唯一标识。程序哈希可用于验证两个程序是否相同。

<details>
<summary>原文参考</summary>

## Definitions

* **CLVM Assembly** - The textual representation of a CLVM program.
* **CLVM Bytecode** - The serialized form of a CLVM program.
* **ChiaLisp** - A higher-level language, built on top of CLVM.
* **CLVM Object** - The underlying data type in the CLVM. An atom or a cons pair.
* **Atom** - The datatype for values in the CLVM. Atoms are immutable byte arrays. Atoms are untyped and are used to encode all strings, integers, and keys. The only things in the CLVM which are not atoms are cons pairs. Atom properties are length, and the bytes in the atom.
* **cons pair** - An immutable ordered pair of references to other CLVM objects. One of two data types in the CLVM. The syntax for a cons pair is a dotted pair. Also called `cons cell` or `cons box`.
* **slot** - One of the cells in a cons box. right or left. Accessed with `f` (first) or `r` (rest).
* **nil** - nil is the special value represented by the zero length byte array. This value represents zero, the empty string, false, and the empty list. Nil is represented in CLVM assembly as `()`, `0`, or `''`.
* **Value** - We use value to mean an abstract value like `1` (an integer), `0xCAFE` (a byte string), `"hello"` (a string) or `(sha256 (q . "hello"))` (a program). Values are represented by CLVM Objects.
* **List** - CLVM lists follow the lisp convention of being a cons pair containing the first list element in the left slot and the rest of the list in the right slot.
* **Proper List** - A "proper" list is a chain of cons boxes, each containing a value in the left slot. Each right slot contains either another cons box, or nil, if it is the last pair.
* **Function** - A function in the CLVM is either a built-in opcode or a user-defined program.
* **Operator** - An opcode/string specifying a built-in function to use.
* **Program** - A CLVM object which can be executed.
* **Opcodes** - An atom corresponding to a reserved keyword. When a list is evaluated with a pre-defined opcode in the first position, the code for that opcode is run.
* **Keyword** - A reserved word in the CLVM assembly language syntax. The strings used for function lookup by the CLVM.
* **Tree** - A binary tree can be formed from cons pairs and atoms by allowing the right and left cells of a cons pair to hold either an atom, or a cons pair. Atoms are the leaves of the tree.
* **Function Parameter** - When a list is evaluated, the first argument is the function, and the other items are parameters. In the program `(+ (q . 1) (q . 2))`, the quoted atoms `1` and `2` are parameters to the operator `+`
* **Treearg** - These are program arguments passed in from outside the program. They are referenced by integers that describe a path in the argument tree.
* **Argument** - Outside the context of the CLVM, the term "argument" can mean "program argument" (the "argv" of the C language, for example), or "function argument", among other things. Because of this potential confusion, we avoid using the term "argument" in this document. The context is especially important considering the way in which CLVM programs look up their program arguments.

A CLVM program must have an unambiguous definition and meaning, so that Chia block validation and consensus is deterministic. Programs are treated as Merkle trees, uniquely identified by the hash at their root. The program hash can be used to verify that two programs are identical.

</details>

# 可读的汇编格式

> Readable assembly format

CLVM 操作的内存对象是原子和 cons 对，但为了编程方便，输入和输出采用了人类可读的字符串格式。这种格式是传递给 `brun` 工具的。 CLVM 程序和数据的这种文本表示未在区块链验证中的任何地方使用，并且对共识没有影响。有多种方法可以以人类可读的序列化格式对相同的 CLVM 对象进行编码，因此倒退以这种格式漂亮地打印 CLVM 对象需要猜测最佳表示。

人类可读表示中的原子可以表示为直接引用的字符串。 它们也可以表示为十进制整数（`100`）或十六进制文字（`0x64`）。

Cons 对可以使用点作为中缀运算符来表示，如下所示：`(3 . 4)`，它对应于包含 3 和 4 的 cons 对。更常见的数据表示是列表，它被写成围绕空格的括号 - 分隔的值列表。

正确的列表是从链接的 cons 对构建的，并假设一个 nil 终止符。 例如，`(3 4 5)` 表示与`(3 . (4 . (5 . nil))) 相同的东西。

请注意，`0`、`''` 和`()` 都被解析为相同的值，但0x0 不是。

<details>
<summary>原文参考</summary>

The in-memory objects the CLVM operates on are atoms and cons pairs, but for programming convenience there is a human readable string format for input and output. This format is what is passed to the `brun` tool. This text representation of CLVM programs and data is not used anywhere in blockchain validation and has no impact on consensus. There are multiple ways of encoding the same CLVM object in the human readable serialization format, so going backwards to pretty print a CLVM object in this format requires guessing as to the best representation.

Atoms in the human readable representation can be represented as directly quoted strings. They can also be expressed as decimal integers (`100`), or hex literals (`0x64`).

Cons pairs can be represented using dot as an infix operator like so: `(3 . 4)`, which corresponds to a cons pair containing 3 and 4. A more common representation of data is lists, which are written as parentheses surrounding a space-delimited list of values.

Proper lists are built from linked cons pairs, and assume a nil terminator. For example `(3 4 5)` represents the same thing as `(3 . (4 . (5 . nil)))`.

Note that `0`, `''`, and `()` all are parsed to the same value, but 0x0 is not.

</details>

# 项目评估

> Program Evaluation

CLVM 汇编的语法类似于 Lisp。它是一个带括号的 [前缀符号](https://en.wikipedia.org/wiki/Polish_notation)，在从左到右阅读时将运算符放在参数之前。

CLVM 实现的语言语义类似于 Lisp。程序被表示为二叉树。树的根是程序树中嵌套最少的对象，内部函数调用递归嵌入其中。在下面的例子中，外括号代表 cons 框，它是树 `(+ (q . 1) (q . 2))` 的根。

每当一个程序被调用时，它总是有一个上下文或环境，它是一个 CLVM 对象。这个对象保存了传递给程序的所有参数。这是 `run` 和 `brun` 的第二个命令行参数。默认环境为零。

如果程序是一个原子，则执行参数查找，并返回参数。请参阅下面的 [treeargs](#treeargs)。

如果程序的根是 cons 对，则计算所有参数（包含在 cons 框的右侧插槽中），然后进行函数调用并返回该函数调用的结果。左边的对象决定了要调用的函数，右边的对象决定它传递的参数。

如果正在执行的列表最左侧位置的对象是列表中的运算符，则调用运算符时无需先评估参数。

如果 CLVM 在“严格模式”下运行，未知的操作码将中止程序。这是 CLVM 在内存池检查和块验证期间运行的模式。在开发人员测试期间，CLVM 可能会在“非严格”模式下运行，这允许使用未知操作码并将其视为无操作。

引用操作码是特殊的。当它被解释器识别时，它会导致任何在权利上的东西都被未评估地返回。所有其他函数首先传递评估右侧内容的结果。

编译好的 CLVM 程序可以被认为是一棵二叉树。

这是函数调用（或“函数调用”）的示例。 `(+ (q . 1) (q . 2))`。该函数是操作码 `+`，这是 clvm 运行时内置的一个函数。

`(+ (q . 1) (q . 2))`

```
      [ ]
     /   \
    +     [ ]
         /   \
      [q, 1]  [ ]
             /   \
         [q, 2]  nil
```

第一次还原后

`(+ 1 2)`

```
      [ ]
     /   \
    +     [ ]
         /   \
        1     [ ]
             /   \
            2    nil
```

二次还原后，以及`+`函数应用

```
3
```

通过首先评估叶节点，然后递归地评估它们的父节点来评估程序树。 函数的参数总是在调用函数之前求值。 CLVM 对象不需要以特定顺序进行评估，但所有子节点必须在其父节点之前进行评估。

如果项目是带引号的值，则返回该值。

如果该项目是一个原子，则该原子将作为 Treearg 查找。

如果要评估的项目是列表，则评估所有参数，然后将评估后的参数传递给函数

函数的所有参数在传递给该函数之前都会进行评估。

<details>
<summary>原文参考</summary>

The syntax of CLVM assembly is similar to Lisp. It is a parenthesized [prefix notation](https://en.wikipedia.org/wiki/Polish_notation) that puts the operator before the arguments when reading left to right.

The semantics of the language implemented by the CLVM is similar to Lisp. A program is represented as a binary tree. The root of the tree is the least nested object in the program tree, with inner function calls embedded recursively inside of it. In the following example, the outer parentheses represent the cons box that is the root of the tree `(+ (q . 1) (q . 2))`.

Whenever a program is called it always has a context, or environment, which is a CLVM object. This object holds all the arguments passed into the program. This is the second command line argument to `run` and `brun`. The default environment is nil.

If the program is an atom then an argument lookup is performed, and the argument is returned. Please see [treeargs](#treeargs), below.

If the the root of the program is a cons pair then all of the parameters (contained in the right slot of the cons box) are evaluated, then a function call is made and the result of that function call is returned. The object on the left determines the function to call and the object on the right determines what arguments it is passed.

If the object in the leftmost position of a list being executed is an operator in a list, the operator is called without first evaluating the parameters.

If the CLVM is running in "strict mode", an unknown opcode will abort the program. This is the mode CLVM is run in during mempool checking and block validation. During developer testing, the CLVM may be run in "non-strict" mode, which allows for unknown opcodes to be used and treated as no-ops.

The quote opcode is special. When it is recognized by the interpreter, it causes whatever is on the right to be returned unevaluated. All other functions are passed the results of evaluating what's on the right first.

A compiled CLVM program can be thought of as a binary tree.

Here is an example of a function invocation (or "function call"). `(+ (q . 1) (q . 2))`. The function is the opcode `+`, a function built-in to the clvm runtime.


`(+ (q . 1) (q . 2))`

```
      [ ]
     /   \
    +     [ ]
         /   \
      [q, 1]  [ ]
             /   \
         [q, 2]  nil
```

After First Reduction

`(+ 1 2)`

```
      [ ]
     /   \
    +     [ ]
         /   \
        1     [ ]
             /   \
            2    nil
```

After Second Reduction, and `+` function application

```
3
```

Program trees are evaluated by first evaluating the leaf nodes, then their parents, recursively.
Arguments to functions are always evaluated before the function is called.
CLVM objects need not be evaluated in a specific order, but all child nodes must be evaluated before their parent.

If the item is a quoted value, the value is returned.

If the item is an atom, the atom is looked up as a Treearg.

If the item to be evaluated is a list, all of the parameters are evaluated and then the evaluated parameters are passed to the function

All arguments of a function are evaluated before being passed to that function.

</details>

## 类型

CLVM 对象的两种类型是 *cons pair* 和 *atom*。 它们可以通过 **listp** 操作码来区分。 CLVM 语言中的原子不携带其他类型信息。 然而，类似于 CPU 的机器代码指令，函数以特定的可预测方式解释原子。 因此，每个函数为其每个参数强加一个类型。

原子的值——它的长度，以及它的字节值——总是定义明确且明确的。 因为原子没有类型信息，原子的含义是在对其应用函数时确定的。 在以下示例中，作为字符串读入的原子被视为整数。

`brun '(+ (q . "helo") (q . 1))'` => `"help"`

在这个例子中，一个作为整数读入的原子被附加到一个字符串中。

`brun'(concat (q . "hello") (q . 49))'` => `"hello1"`

### 原子作为字节数组

任意长度的无符号整数。如果需要更多位来对不同长度的原子执行操作，则原子实际上会向左扩展零字节。

### 无符号整数

任意长度的无符号整数。如果需要更多位来对不同长度的原子执行操作，则原子实际上会向左扩展零字节。

### 有符号整数

字节数组表现为二进制补码有符号整数。最高位表示负数。底层表示很重要，因为单个字节可以通过其他操作查看。

这种类型有可能将多个表示视为相同的值。例如，0xFF 和 0xFFFF 都编码“-1”。将返回的原子视为有符号整数的整数算术运算将返回负数的最小表示，例如。 `-1` 的 `0xFF`

这些整数是字节对齐的。例如，`0xFFF` 被解释为两个字节 `0x0F`、`0xFF`，值为 `4095`。

当预期将数字解释为字符串时，这也可能导致数字的意外表示。

如果正整数的第一个字节 >= `0x80`（最高有效位是 1），那么当操作符输出类型是有符号整数时，它将在前面加上 `0x00`。如果没有该前置字节，则在设置了高位的情况下，正值将显示为负。当使用 int 操作的输出作为字符串操作的输入时，您可能会遇到这种情况。

### BLS点

这种类型表示 [此处](https://electriccoin.co/blog/new-snark-curve/) 描述的有限域上椭圆曲线上的一个点。

这些值是不透明的值，长度为 48 个字节。 `pubkey_for_exp` 的输出是 BLS 点。 `point_add` 的输入和输出是 BLS 点。

<details>
<summary>原文参考</summary>

- ## Types

The two types of CLVM Object are *cons pair* and *atom*. They can be distinguished by the **listp** opcode. Atoms in the CLVM language do not carry other type information. However, similarly to the machine code instructions for a CPU, functions interpret atoms in specific predictable ways. Thus, each function imposes a type for each of its arguments.

The value of an atom - its length, and the values of its bytes - are always well defined and unambiguous. Because atoms have no type information, the meaning of an atom is determined when a function is applied to it. In the following example, an atom that was read in as a string is treated as an integer.

`brun '(+ (q . "helo") (q . 1))'` => `"help"`

And in this example, an atom that was read in as an integer is appended to a string.

`brun '(concat (q . "hello") (q . 49))'` => `"hello1"`


- ### Atoms as Byte Arrays

The atom is treated as an array of bytes, with a length. No specific semantics are assumed, except as specified in the instruction.

- ### Unsigned Integer

An unsigned integer of arbitrary length. If more bits are required to perform an operation with atoms of different length, the atom is virtually extended with zero bytes to the left.

- ### Signed Integer

The byte array behaves as a two's complement signed integer. The most significant bit denotes a negative number. The underlying representation matters, because the individual bytes are viewable through other operations.

This type has the potential for multiple representations to be treated as the same value. For example, 0xFF and 0xFFFF both encode `-1`. Integer arithmetic operations that treat returned atoms as signed integers will return the minimal representation for negative numbers, eg. `0xFF` for `-1`

These integers are byte-aligned. For example, `0xFFF` is interpreted as the two bytes `0x0F`,`0xFF`, with value `4095`.

This can also cause unexpected representations of numbers when they are expected to be interpreted as strings..

If a positive integer's first byte is >= `0x80` (the most significant bit is 1) then it will be prepended with a `0x00` when the operator output type is a signed integer. Without that prepended byte, a positive value would appear negative in the case that the high bit is set.
You are likely to encounter this when using the output of an int operation as the input of a string operation.

- ### BLS Point

This type represents a point on an elliptic curve over finite field described [here](https://electriccoin.co/blog/new-snark-curve/).

These values are opaque values, 48 bytes in length. The outputs of `pubkey_for_exp` are BLS points. The inputs and outputs of `point_add` are BLS points.

</details>

## Treeargs ：程序参数和参数查找

对于在确定性机器上运行的具有不同行为的程序，它必须能够具有不同的起始状态。 CLVM 程序的起始状态是程序参数列表—— treearg。

当计算一个不带引号的整数时，它被替换为程序 Treearg 中的相应值/CLVM 对象。 如果未找到该参数，则返回 `nil`。

作为对通过调用 **first** 和 **rest** 遍历参数树的改进，参数通过参数编号从参数列表中引用。 该数字是通过将从参数树的根到该 CLVM 对象的左右 cons 槽的路径转换为一系列 1 和 0 来得出的。 从最低有效位开始读取表示路径的数字。 参数树的根数为 `1`。 当路径完成时，最后一个 `1` 被附加到数字的最高位。

### 参数编号的说明

我们将 s 表达式视为二叉树，其中叶节点是原子，而 cons 对是具有两个孩子的节点。然后我们对路径进行编号，如下所示：

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

这种古怪的编号使实现变得简单。

编号从树的根部开始。 路径索引设置为 1，表示整个参数树。 当我们下降时，位被附加到路径索引的右侧，0 代表左边，1 代表右边。

查看实现[这里](https://github.com/Chia-Network/clvm_tools/blob/main/clvm_tools/NodePath.py)

<details>
<summary>原文参考</summary>

- ## Treeargs : Program Arguments, and Argument Lookup

For a program running on a deterministic machine to have different behaviours, it must be able to have different starting states. The starting state for a CLVM program is the program argument list - the treearg.

When an unquoted integer is evaluated, it is replaced with the corresponding value/CLVM Object from the program Treearg. If the argument is not found, `nil` is returned.

As an improvement over walking the argument tree via calls to **first** and **rest**, arguments are referenced from the argument list by their argument number. This number is derived by translating a path of left and right cons slots followed from the root of the argument tree to that CLVM Object, into a series of ones and zeros. The number representing the path is read starting at the least significant bit. The number of the root of the argument tree is `1`. When the path is complete, a final `1` is appended to the msb of the number.

- ### Illustration of argument numbering

We treat an s-expression as a binary tree, where leaf nodes are atoms, and cons pairs
are nodes with two children. We then number the paths as follows:

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

This quirky numbering makes the implementation simple.

Numbering starts at the root of the tree. The path index is set to 1, which represents the entire argument tree.
Bits are appended to the right of the path index as we descend, 0 for left, and 1 for right.

See the implementation [here](https://github.com/Chia-Network/clvm_tools/blob/main/clvm_tools/NodePath.py)

</details>

## 引用

在大多数编程语言中，对文字求值会返回值本身。在 CLVM 中，原子在求值时（在列表的任何位置，除了第一个位置）的含义是对参数树中值的引用。 xxx

因此，当您打算编写时：

`(+ 1 2)` => `3`

你必须写：`(+ (q . 1) (q . 2))` => `3`

nil 是自引用。

### 编译：Atom 语法

尽管原子只有一种底层表示，但在编译时会识别出不同的语法，并且在从程序文本到 CLVM 对象的转换过程中，这些原子语法的解释也不同。

Nil、十进制零和空字符串都计算为同一个原子。

`(q . ())` => `()`

`(q . 0)` => `()`

`(q . "")` => `()`

这与单个零字节不同。

`(q . 0x0)` => `0x00`

#### 字符串、符号、十六进制字符串和数字的等效性

`"A"` 与 `A` 是同一个原子

```chialisp
(q . "A") => 65
(q . A) => 65
(q . 65) => 65
(q . 0x41) => 65
```

但是，对于内置插件而言，情况并非如此。`"q"` 与 `q` 不一样

`"q"` is not the same as `q`
```chialisp
(q . q) => 1
(q . "q") => 113
```

<details>
<summary>原文参考</summary>

- ## Quoting

In most programming languages, evaluating a literal returns the value itself.
In CLVM, the meaning of an atom at evaluation time (at any position of the list except the first), is a reference to a value in the argument tree. xxx

Therefore, when you intend to write:

`(+ 1 2)` => `3`

You must instead write:
`(+ (q . 1) (q . 2))` => `3`

nil is self-quoting.

### Compilation: Atom Syntax

Although there is only one underlying representation of an atom, different syntaxes are recognized during compile time, and those atom syntaxes are interpreted differently during the translation from program text to CLVM Objects.

Nil, decimal zero and the empty string all evaluate to the same atom.

`(q . ())` => `()`

`(q . 0)` => `()`

`(q . "")` => `()`

which is not the same as a single zero byte.

`(q . 0x0)` => `0x00`

#### Equivalence of Strings, symbols, hex strings, and numbers

`"A"` is the same atom as `A`

```chialisp
(q . "A") => 65
(q . A) => 65
(q . 65) => 65
(q . 0x41) => 65
```

However, the same is not true for Built-ins.
`"q"` is not the same as `q`
```chialisp
(q . q) => 1
(q . "q") => 113
```

</details>

## 运算符也是原子..

编写程序时，列表中的第一个参数被解释为运算符。但是，此运算符也存储为 unsigned int。这可能导致歧义和令人困惑的输出：

`(r (q . (1 2 3)))` => `(a 3)`

由于 `2` 位于列表的开头，`brun` 假定它是运算符并查找其对应的表示，在本例中为 `a`。它是程序的正确输出，只是以一种意想不到的方式显示出来。

<details>
<summary>原文参考</summary>

- ## Operators are atoms too..



When you write a program, the first argument in the list is interpreted as an operator.
However, this operator is also stored as an unsigned int.
This can lead to ambiguity and confusing outputs:

`(r (q . (1 2 3)))` => `(a 3)`

Since `2` is at the beginning of the list, `brun` assumes it is the operator and looks up its corresponding representation, which in this case is `a`.
It is the correct output of the program, it is just displayed in an unexpected way.

</details>

## Errors

在运行 clvm 程序时，会进行检查以确保 CLVM 不会进入未定义状态。 当程序违反这些运行时检查之一时，就会说它导致了错误。

* 评估列表中的第一个元素不是有效函数。 示例：`("hello" (q . 1))` => `FAIL：未实现的运算符“hello”`
* 参数数量错误。 示例：`(lognot (q . 1) (q . 2))` => `失败：lognot 需要 1 个参数`
* 程序评估超过最大成本见[成本](/docs/ref/clvm#costs)
* 执行了太多的分配
* 参数检查，例如负索引 `run'(substr "abc" -1 -)'` 失败：substr ("abc" -1 17) 的索引无效

错误将导致程序中止。

<details>
<summary>原文参考</summary>

- ## Errors

While running a clvm program, checks are made to ensure the CLVM does not enter an undefined state. When a program violates one of these runtime checks, it is said to have caused an error.

* First element in an evaluated list is not a valid function. Example: `("hello" (q . 1))` => `FAIL: unimplemented operator "hello"`
* Wrong number of arguments. Example: `(lognot (q . 1) (q . 2))` => `FAIL: lognot requires 1 arg`
* Program evaluation exceeds max cost see [Costs](/docs/ref/clvm#costs)
* Too many allocations have been performed
* Argument checking e.g. negative index `run '(substr "abc" -1 -)'` FAIL: invalid indices for substr ("abc" -1 17)

An error will cause the program to abort.

</details>

# 操作员总结

> Operator Summary

## 内置操作码

操作码是内置于 CLVM 的函数。它们可用于任何正在运行的程序。

## 列出运算符

**c** *cons* `(c A B)` 正好接受两个操作数并返回一个包含两个对象的 cons 对（左边的 A，右边的 B）

示例：`'(c (q . "A") (q . ()))'` => `(65)`

**f** *first* `(f X)` 只接受一个必须是 cons 对的操作数，并返回左半部分

**r** *rest* `(r X)` 只接受一个必须是 cons 对的操作数，并返回右半部分

**l** *listp* `(l X)` 只接受一个操作数，如果它是一个原子则返回 `()`，如果它是一个 cons 对，则返回 `1`。 与大多数其他 lisps 相比，nil 不是 CLVM 中的列表。

<details>
<summary>原文参考</summary>

- ## The built-in opcodes

Opcodes are functions built in to the CLVM. They are available to any running program.

- ## List Operators

**c** *cons* `(c A B)` takes exactly two operands and returns a cons pair with the two objects in it (A in the left, B in the right)

Example: `'(c (q . "A") (q . ()))'` => `(65)`

**f** *first* `(f X)` takes exactly one operand which must be a cons pair, and returns the left half

**r** *rest* `(r X)` takes exactly one operand which must be a cons pair, and returns the right half

**l** *listp* `(l X)` takes exactly one operand and returns `()` if it is an atom or `1` if it is a cons pair. In contrast to most other lisps, nil is not a list in CLVM.

</details>

## 控制流

**a** *apply* `(a P A)` 使用参数 A 运行程序 P。请注意，这将在新环境中执行 P。使用整数来引用解决方案中的值将引用 A 中的值。

**i** *if* `(i A B C)` 正好采用三个操作数 `A`、`B`、`C`。如果`A`是`()`，则返回`C`。否则，返回`B`。在评估 *if* 之前先评估 B 和 C。

**x** *raise exception* `(x X Y ...)` 接受任意数量的参数（甚至零）。立即失败，将参数列表传递到 (python) 异常中。在评估此指令后，不会运行其他 CLVM 指令。

**=** *equal* `(= A B)` 如果 `A` 和 `B` 都是原子并且都相等，则返回 1。否则`()`。不要用它来测试两个程序是否相同。这是由它们的树哈希决定的。 Nil 测试等于零，但 nil 不等于单个零字节。

**>** *大于* `(> A B)` 如果 `A` 和 `B` 都是原子并且 A 大于 B，则返回 1，将两者解释为有符号整数的补码。否则`()`。 `(> A B)` 在中缀语法中表示 `A > B`。

**>s** *大于字节* `(>s A B)` 如果 `A` 和 `B` 都是原子并且 A 大于 B，则返回 1，将两者解释为无符号字节数组。否则`()`。与 strcmp 相比。
`(>s "a" "b")` => `()`

**not** `(not A)` 如果 `A` 的计算结果为 `()`，则返回 1。否则，返回`()`。

**all** `(all A B ...)` 接受任意数量的参数（甚至零）。如果任何参数的计算结果为“()”，则返回“()”。否则，返回 1。

**any** `(any A B ...)` 接受任意数量的参数（甚至零）。如果任何参数的计算结果不是 `()`，则返回 1。否则，返回`()`。

<details>
<summary>原文参考</summary>

- ## Control Flow

**a** *apply* `(a P A)` run the program P with the arguments A. Note that this executes P in a new environment. Using integers to reference values in the solution will reference values in A.

**i** *if* `(i A B C)` takes exactly three operands `A`, `B`, `C`. If `A` is `()`, return `C`. Otherwise, return `B`. Both B and C are evaluated before *if* is evaluated.

**x** *raise exception* `(x X Y ...)` takes an arbitrary number of arguments (even zero). Immediately fail, with the argument list passed up into the (python) exception. No other CLVM instructions are run after this instruction is evaluated.

**=** *equal* `(= A B)` returns 1 if `A` and `B` are both atoms and both equal. Otherwise `()`. Do not use this to test if two programs are identical. That is determined by their tree hash. Nil tests equal to zero, but nil is not equal to a single zero byte.

**>** *greater than* `(> A B)` returns 1 if `A` and `B` are both atoms and A is greater than B, interpreting both as two's complement signed integers. Otherwise `()`. `(> A B)` means `A > B` in infix syntax.

**>s** *greater than bytes* `(>s A B)` returns 1 if `A` and `B` are both atoms and A is greater than B, interpreting both as an array of unsigned bytes. Otherwise `()`. Compare to strcmp.
`(>s "a" "b")` => `()`

**not** `(not A)` returns 1 if `A` evaluates to `()`. Otherwise, returns `()`.

**all** `(all A B ...)` takes an arbitrary number of arguments (even zero). Returns `()` if any of the arguments evaluate to `()`. Otherwise, returns 1.

**any** `(any A B ...)` takes an arbitrary number of arguments (even zero). Returns 1 if any of the arguments evaluate to something other than `()`. Otherwise, returns `()`.

</details>

## 常数

**q** *quote* 形式`(q . X)` 求值时返回X，*未*求值。示例：`(q . "A")` => `65`

## 整数运算符

算术运算符 `+`、`-`、`*`、`/` 和 `divmod` 将它们的参数视为有符号整数。

**`+`** `(+ a0 a1 ...)` 接受任意数量的整数操作数并将它们相加。 如果没有给定参数，则返回零。

**`-`** `(- a0 a1 ...)` 接受一个或多个整数操作数并将 a0 添加到其余操作数的负数。 给出零个参数返回 0。

**`*`** `(* a0 a1 ...)` 接受任意数量的整数操作数并返回乘积。

**`/`** `(/ A B)` 将两个整数相除并返回落地商

### 四舍五入

```chialisp
(/ 1  2) => ()
(/ 2  2) => 1
(/ 4  2) => 2
```

### 负数的除法

请注意，有余数的除法总是向负无穷大舍入，而不是向零舍入。

```chialisp
(/ -3 2) => -2
(/ 3 2) => 1
```

这意味着`-a / b` 并不总是等于`-(a / b)`

**divmod** `(divmod A B)` 接受两个整数并返回一个包含地板商和余数的 cons-box。

```chialisp
(divmod 10 3)
   => (3 . 1)
```

<details>
<summary>原文参考</summary>

- ## Constants

**q** *quote* The form `(q . X)` when evaluated returns X, which is *not* evaluated.
Example: `(q . "A")` => `65`

- ## Integer Operators

The arithmetic operators `+`, `-`, `*`, `/` and `divmod` treat their arguments as signed integers.

**`+`** `(+ a0 a1 ...)` takes any number of integer operands and sums them. If given no arguments, zero is returned.

**`-`** `(- a0 a1 ...)` takes one or more integer operands and adds a0 to the negative of the rest. Giving zero arguments returns 0.

**`*`** `(* a0 a1 ...)` takes any number of integer operands and returns the product.

**`/`** `(/ A B)` divides two integers and returns the floored quotient


- ### Rounding

```chialisp
(/ 1  2) => ()
(/ 2  2) => 1
(/ 4  2) => 2
```

- ### Division of negative numbers

The treatment of negative dividend and divisors is as follows:
```chialisp
(/ -1 1) => -1
(/ 1 -1) => -1
(/ -1 -1) =>  1
```

- ### Flooring of negative nubmers

Note that a division with a remainder always rounds towards negative infinity, not toward zero.
```chialisp
(/ -3 2) => -2
(/ 3 2) => 1
```
This means that `-a / b` is not always equal to `-(a / b)`

**divmod** `(divmod A B)` takes two integers and returns a cons-box containing the floored quotient and the remainder.
```chialisp
(divmod 10 3)
   => (3 . 1)
```

</details>

## Bit 操作

`logand`、`logior` 和 `logxor` 对任意数量的参数进行操作 nil 作为这些函数的参数被视为零。 如果 A 或 B 不是原子，则失败。 较短的原子被符号扩展到与较长的原子相等的长度。

`logand`、`logior` 和 `logxor` 接受 0 个或多个参数。 有一个隐含的 *identity* 参数，它是所有参数都适用的值。 如果给出 0 个参数，则将仅返回标识。

**logand** `(logand A B ...)` 一个或多个原子的按位 **AND**。 标识为 `-1`。

```chialisp
(logand -128 0x7fffff)
   => 0x7fff80
```

第一个参数是 `0x80`（因为它是二进制补码）。 它是负数，它将用 1 进行符号扩展。 符号扩展后，计算变为 `0xfff80` 和 `0x7fffff` = `0x7fff80`。

**logior** `(logior A B ...)` 一个或多个原子的按位逻辑 **OR**。 身份是 `0`。

```chialisp
(logior -128 0x7fffff)
   => -1
```

扩展第一个参数的符号变成`0xffff80`，与`0x7fffff` 的OR 变成`0xffffff`，在二进制补码中为-1。 请注意，生成的原子将使用“-1”的最小编码，即“0xff”。

**logxor** `(logxor A B ...)` 任意数量原子的按位 **XOR**。 身份是“0”。

```chialisp
(logxor -128 0x7fffff)
   => 0x80007f
```

扩展第一个参数的符号变为“0xffff80”，与“0x7fffff”异或变为“0x80007f”。 这是一个负数（在二进制补码中），它也是它的最小表示。

**lognot** `(lognot A)` A 的按位 **NOT**。所有位都被反转。

```chialisp
(lognot ()) => -1
(lognot 1) => -2
(lognot (lognot 17)) => 17
```

<details>
<summary>原文参考</summary>

- ## Bit Operations

`logand`, `logior` and `logxor` operate on any number of arguments
nil as an argument to these functions is treated as a zero.
Fail if either A or B is not an atom.
The shorter atom is sign-extended to equal length as the longer atom.

The `logand`, `logior` and `logxor` accept 0 or more parameters.
There is an implicit *identity* argument, which is the value all parameters will apply to.
The identity will just be returned in case 0 arguments are given.

**logand** `(logand A B ...)` bitwise **AND** of one or more atoms. Identiy is `-1`.

```chialisp
(logand -128 0x7fffff)
   => 0x7fff80
```

The first argument is `0x80` (since it's Two's complement). It is negative, it will be sign-extended with ones.
Once sign-extended, the computation becomes `0xffff80` AND `0x7fffff` = `0x7fff80`.

**logior** `(logior A B ...)` bitwise logical **OR** of one or more atoms. Identity is `0`.

```chialisp
(logior -128 0x7fffff)
   => -1
```

Sign extending the first argument becomes `0xffff80`, ORing that with `0x7fffff` becomes `0xffffff` which is -1 in Two's complement.
Note that the resulting atom will use the minimal encoding of `-1`, i.e. `0xff`.

**logxor** `(logxor A B ...)` bitwise **XOR** of any number of atoms. Identity is `0`.

```chialisp
(logxor -128 0x7fffff)
   => 0x80007f
```

Sign extending the first argument becomes `0xffff80`, XORing that with `0x7fffff` becomes `0x80007f`.
This is a negative number (in Two's complement) and it's also the minimal representation of it.

**lognot** `(lognot A)` bitwise **NOT** of A. All bits are inverted.

```chialisp
(lognot ()) => -1
(lognot 1) => -2
(lognot (lognot 17)) => 17
```

</details>

## 转变

移位运算符有两种变体。 算术移位（`ash`）和逻辑移位（`lsh`）。 两者均可用于向左和向右移动，方向由 *count* 参数的符号确定。 正数 *count* 左移，负数 *count* 右移。 对于 **ash** 和 **lsh**，如果 |*count*| 超过 65535，操作失败。 结果值被视为一个有符号整数，并去除任何多余的前导零字节或“0xff”字节。

**ash** `(ash A count)` 如果 *count* 为正数，返回 *A* 左移 *count* 位，否则返回 *A* 右移 |*count*| 位，符号扩展。

算术移位将要移位的值 (*A*) 视为有符号整数，当右移时，符号扩展最左边的位。

左移时，添加到值左侧的任何新字节也用符号扩展位填充。 例如：

```chialisp
(ash -1 8) ; -1 = . . . 11111111
   => -256 ; -256 = . . 1111111100000000
```

算术左移只会在需要更多位时扩展原子长度

```chialisp
(strlen (ash -1 7))
   => 1
(strlen (ash -1 8))
   => 2
(strlen (ash 255 1))
  => 2
(strlen (ash 128 1))
  => 2
(strlen (ash 127 1))
  => 2
```

负数的连续右移将导致最终值为 -1。

```chialisp
(ash -7 -1) ; -7 = . . . 11111001
   => -4
(ash -4 -1) ; -4 = . . . 11111100
   => -2
(ash -2 -1) ; -2 = . . . 11111110
   => -1
(ash -1 -1) ; -1 = . . . 11111111
   => -1
```

`-1` 任意量的右移仍然是 `-1`：

```chialisp
(ash -1 -99)
   => -1
```

**lsh** `(lsh A count)` 如果 *count* 为正数，返回 *A* 左移 *count* 位，否则返回 *A* 右移 |*count*|位，在左侧添加零位。

逻辑移位将要移位的值视为无符号整数，并且在右移时不进行符号扩展。

```chialisp
(lsh -7 -1) ; -7 = . . . 11111001
   => 124   ;    = . . . 01111100

(lsh -5 -2) ; -5 = . . . 11111011
   => 62    ;    = . . . 00111110
```

具有高位设置的原子的左移将向左扩展原子，并导致分配。

```chialisp
(lsh -1 1) ; -1 = . . . 11111111
   => 510  ;    = . . . 0000000111111110
(strlen (lsh -1 1))
   => 2
(strlen (lsh 255 1))
  => 2
(strlen (lsh 128 1))
  => 2
(strlen (lsh 127 1))
  => 2
```

<details>
<summary>原文参考</summary>

- ## Shifts

There are two variants of bit shift operators.
Arithmetic shift (`ash`) and Logical shift (`lsh`). Both can be used to shift both left and right, the direction is determined by the sign of the *count* argument.
A positive *count* shifts left, a negative *count* shifts right.
For both **ash** and **lsh**, if |*count*| exceeds 65535, the operation fails.
The resulting value is treated as a signed integer, and any redundant leading zero-bytes or `0xff` bytes are stripped.

**ash** `(ash A count)` if *count* is positive, return *A* shifted left *count* bits, else returns *A* shifted right by |*count*| bits, sign extended.

Arithmetic shift treats the value to be shifted (*A*) as a signed integer, and sign extends the left-most bits when when shifting right.

When shifting left, any new bytes added to the left side of the value are also filled with the sign-extended bit. For example:

```chialisp
(ash -1 8) ; -1 = . . . 11111111
   => -256 ; -256 = . . 1111111100000000
```

A arithmetic left shift will only extend the atom length when more bits are needed

```chialisp
(strlen (ash -1 7))
   => 1
(strlen (ash -1 8))
   => 2
(strlen (ash 255 1))
  => 2
(strlen (ash 128 1))
  => 2
(strlen (ash 127 1))
  => 2
```

Consecutive right shifts of negative numbers will result in a terminal value of -1.

```chialisp
(ash -7 -1) ; -7 = . . . 11111001
   => -4
(ash -4 -1) ; -4 = . . . 11111100
   => -2
(ash -2 -1) ; -2 = . . . 11111110
   => -1
(ash -1 -1) ; -1 = . . . 11111111
   => -1
```

A right shift of `-1` by any amount is still `-1`:
```chialisp
(ash -1 -99)
   => -1
```

**lsh** `(lsh A count)` if *count* is positive, return *A* shifted left *count* bits, else returns *A* shifted right |*count*| bits, adding zero bits on the left.

Logical shift treats the value to be shifted as an unsigned integer, and does not sign extend on right shift.

```chialisp
(lsh -7 -1) ; -7 = . . . 11111001
   => 124   ;    = . . . 01111100

(lsh -5 -2) ; -5 = . . . 11111011
   => 62    ;    = . . . 00111110
```

A left shift of an atom with the high bit set will extend the atom left, and result in an allocation.

```chialisp
(lsh -1 1) ; -1 = . . . 11111111
   => 510  ;    = . . . 0000000111111110
(strlen (lsh -1 1))
   => 2
(strlen (lsh 255 1))
  => 2
(strlen (lsh 128 1))
  => 2
(strlen (lsh 127 1))
  => 2
```

</details>

## 字符串

**substr** `(substr S I1 I2)` 返回一个包含 \[`I1`, `I2`) 范围内字节的原子。 索引 0 指的是字符串 `S` 的第一个字节。 `I2` 必须大于或等于 `I1`。 `I1` 和 `I2` 都必须大于或等于 0，并且小于或等于字符串 `S` 末尾之后的 1。

`substr` 的第三个参数是可选的。 如果省略，则返回范围 \[`I1`, `(strlen S)`)。

```chialisp
(substr "clvm" 0 4) => "clvm"
(substr "clvm" 2 4) => 30317 ; = "vm"
(substr "clvm" 4 4) => ()

(substr "clvm" 1) => "lvm"

(substr "clvm" 4 5) => FAIL
(substr "clvm" 1 0) => FAIL
(substr "clvm" -1 4) => FAIL
```

**strlen** `(strlen S)` 返回 `S` 中的字节数。

```chialisp
(strlen "clvm") => 4
(strlen "0x0") => 3
(strlen 0x0) => 1
(strlen "") => ()
(strlen 0) => ()
(strlen ()) => ()
(strlen ()) => ()
```

**concat** `(concat A ...)` 返回任意数量原子的连接。

例子：

```chialisp
(concat "Hello" " " "world")
   => "Hello world"
```

<details>
<summary>原文参考</summary>

- ## Strings

**substr** `(substr S I1 I2)` return an atom containing the bytes in range \[`I1`, `I2`). Index 0 refers to the first byte of the string `S`. `I2` must be greater than or equal to `I1`. Both `I1` and `I2` must be greater than or equal to 0, and less than or equal to one past the end of the string `S`.

The third parameter to `substr` is optional. If omitted, the range \[`I1`, `(strlen S)`) is returned.

```chialisp
(substr "clvm" 0 4) => "clvm"
(substr "clvm" 2 4) => 30317 ; = "vm"
(substr "clvm" 4 4) => ()

(substr "clvm" 1) => "lvm"

(substr "clvm" 4 5) => FAIL
(substr "clvm" 1 0) => FAIL
(substr "clvm" -1 4) => FAIL
```

**strlen** `(strlen S)` return the number of bytes in `S`.

```chialisp
(strlen "clvm") => 4
(strlen "0x0") => 3
(strlen 0x0) => 1
(strlen "") => ()
(strlen 0) => ()
(strlen ()) => ()
(strlen ()) => ()
```

**concat** `(concat A ...)` return the concatenation of any number of atoms.

Example:

```chialisp
(concat "Hello" " " "world")
   => "Hello world"
```

</details>

## 流媒体运营商

**sha256**`(sha256 A ...)` 返回其参数字节的 sha256 哈希值（作为 32 字节的 blob）。

```chialisp
(sha256 "clvm")
   => 0xcf3eafb281c0e0e49e19c18b06939a6f7f128595289b08f60c68cef7c0e00b81
(sha256 "cl" "vm")
   => 0xcf3eafb281c0e0e49e19c18b06939a6f7f128595289b08f60c68cef7c0e00b81
```

<details>
<summary>原文参考</summary>

- ## Streaming Operators

**sha256**
  `(sha256 A ...)` returns the sha256 hash (as a 32-byte blob) of the bytes of its parameters.

```chialisp
(sha256 "clvm")
   => 0xcf3eafb281c0e0e49e19c18b06939a6f7f128595289b08f60c68cef7c0e00b81
(sha256 "cl" "vm")
   => 0xcf3eafb281c0e0e49e19c18b06939a6f7f128595289b08f60c68cef7c0e00b81
```

</details>

## BLS12-381 运算符

`point_add` 和 `pubkey_for_exp` 对 BLS12-381 曲线的 G1 点进行操作。这些表示为 48 字节 == 384 位。

```chialisp
(strlen (pubkey_for_exp 1))
   => 48
```

**point_add** `(point_add a0 a1 ...)` 取任意数量的 [BLS12-381](https://electriccoin.co/blog/new-snark-curve/) G1 点并将它们相加。

例子：

```chialisp
(point_add (pubkey_for_exp 1) (pubkey_for_exp 2))
   => 0x89ece308f9d1f0131765212deca99697b112d61f9be9a5f1f3780a51335b3ff981747a0b2ca2179b96d2c0c9024e5224
```

**pubkey_for_exp** `(pubkey_for_exp A)` 将整数 A 转换为 G1 上的 BLS12-381 点。

```chialisp
(pubkey_for_exp 1)
   => 0x97f1d3a73197d7942695638c4fa9ac0fc3688c4f9774b905a14e3a3f171bac586c55e83ff97a1aeffb3af00adb22c6bb
```

<details>
<summary>原文参考</summary>

- ## BLS12-381 operators

`point_add` and `pubkey_for_exp` operate on G1 points of the BLS12-381 curve. These are represented as 48 bytes == 384 bits.

```chialisp
(strlen (pubkey_for_exp 1))
   => 48
```

**point_add**
  `(point_add a0 a1 ...)` takes an arbitrary number of [BLS12-381](https://electriccoin.co/blog/new-snark-curve/) G1 points and adds them.

Example:
```chialisp
(point_add (pubkey_for_exp 1) (pubkey_for_exp 2))
   => 0x89ece308f9d1f0131765212deca99697b112d61f9be9a5f1f3780a51335b3ff981747a0b2ca2179b96d2c0c9024e5224
```

**pubkey_for_exp**
  `(pubkey_for_exp A)` turns the integer A into a BLS12-381 point on G1.

```chialisp
(pubkey_for_exp 1)
   => 0x97f1d3a73197d7942695638c4fa9ac0fc3688c4f9774b905a14e3a3f171bac586c55e83ff97a1aeffb3af00adb22c6bb
```

</details>

## 软分叉

`softfork` 操作符至少需要一个参数成本。所以`（软分叉成本 arg_1 ... arg_n）`。

目前，`softfork` 总是返回 `0`（又名 `()` 或 nil），并获取 `cost` 的成本量。

乍一看，它似乎没什么用，因为它什么都不做，只是浪费成本。

这个想法是，在软分叉之后，参数的含义可能会改变。事实上，我们可以在这里隐藏 ChiaLisp 的全新方言，它具有计算新事物的新运算符。

例如，假设我们要添加 secp256k1 运算符，如 `+s`，以便在此 ECDSA 曲线上添加两个点以实现比特币兼容性。我们不能只在普通 clvm 中这样做，因为这会使程序 `(+s p1 p2)` 在软分叉之前和之后返回不同的值。因此，我们将其隐藏在 `softfork` 下。

`(mod (cost p1 p2 p3 p4) (softfork cost 1 (assert (= (+s p1 p2) (+s p3 p4))))`

在软分叉之前，这总是以`COST`（加上一点开销）为代价通过并返回`()`。

软分叉后，这也会以 `COST` 为代价返回 `()`...但如果 `p1 + p2 ≠ p3 + p4` 也可能失败！我们无法将总和导出到软分叉边界之外，但我们可以计算总和并将其与内部的其他事物进行比较。

还有一件事——我们计算在 `softfork` 边界内运行程序的成本，并确保它与 `COST` 完全匹配，并在错误时引发异常。这样，该程序在软分叉前后确实具有相同的成本（或者它在软分叉后失败）。

<details>
<summary>原文参考</summary>

- ## softfork

The `softfork` operator takes at least one parameter cost. So `(softfork cost arg_1 ... arg_n)`.

At the moment, `softfork` always returns `0` (aka `()` or nil), and takes `cost` amount of cost.

At first glance, it seems pretty useless since it doesn't do anything, and just wastes cost doing it.

The idea is, after a soft fork, the meaning of the arguments may change. In fact, we can hide completely new dialects of ChiaLisp inside here, that has new operators that calculate new things.

For example, suppose we want to add secp256k1 operators like `+s` for adding two points on this ECDSA curve for bitcoin compatibility. We can't just do this in vanilla clvm because that would make a program `(+s p1 p2)` return different values before and after the soft fork. So instead we hide it under `softfork`.

`(mod (cost p1 p2 p3 p4) (softfork cost 1 (assert (= (+s p1 p2) (+s p3 p4)))))`

Pre-softfork, this always passes and returns `()` at a cost of `COST` (plus a bit of overhead).

Post-softfork, this also returns `()` at a cost of `COST`... but may also fail if `p1 + p2 ≠ p3 + p4`! We can't export the sum outside the `softfork` boundary, but we can calculate the sum and compare it to another thing inside.

One more thing -- we take the cost of running the program inside the `softfork` boundary and ensure it exactly matches `COST`, and raise an exception if it's wrong. That way, the program really does have the same cost pre and post-softfork (or it fails post-softfork).

</details>

## 算术和按位标识

一些运算符有一个特殊的值，当它们以零参数调用时会返回该值。该值是该函数的标识。例如，调用带有零个参数的运算符 `+` 将返回 `0`：

`(+)` => `0`
Operator | Identity
---|---
`+`| 0
`-`| 0
`*`| 1
logand| all 1's
logior| all zeros
logxor| all zeros

请注意，`/`、`divmod` 和`lognot` 没有标识值。用零参数调用它们是错误的。

<details>
<summary>原文参考</summary>

- ## Arithmetic and Bitwise Identities

Some operators have a special value that is returned when they are called with zero arguments. This value is the identity of that function. For example, calling the operator `+` with zero arguments will return `0`:

`(+)` => `0`

Operator | Identity
---|---
`+`| 0
`-`| 0
`*`| 1
logand| all 1's
logior| all zeros
logxor| all zeros

Note that `/`, `divmod`, and `lognot` do not have an identity value. Calling them with zero arguments is an error.

</details>

## 算术

### nil 用作整数时的行为

在整数上下文中使用时，nil 的行为为零。

### 当用作可以检查 nil 的值时为零的行为

当用作可以检查 nil 的参数时，零被解释为 nil。

<details>
<summary>原文参考</summary>

- ## Arithmetic

- ### Behaviour of nil when used as an integer

When used in an integer context, nil behaves as zero.

- ### Behaviour of zero when used as a value that may be checked for nil

When used as a parameter that may be checked for nil, zero is interpreted as nil.

</details>

## 费用

当一个 clvm 程序运行时，它会产生一个成本。最小程序成本是 40。每个操作码运行后，其成本被添加到总程序成本中。当成本超过阈值时，程序终止，并且不返回任何值。此外，如果原子或原子对的数量超过 2^31，则程序终止且不返回任何值。

| operator | base cost | cost per arg | cost per byte |
| -------- | --------- | ------------ | ------------- |
| `f` | 30 | - | - |
| `i` | 33 | - | - |
| `c` | 50 | - | - |
| `r` | 30 | - | - |
| `l` | 19 | - | - |
| `q` | 20 | - | - |
| `a` | 90 | - | - |
| `=` | 117 | - | 1 |
| `+` | 99 | 320 | 3 |
| `/` | 988 | - | 4 |
| `*` | 92 | 885 | [看这里](https://github.com/Chia-Network/clvm_tools/blob/main/costs/README.md#multiplication) |
| `logand`, `logior`, `logxor` | 100 | 264| 3 |
| `lognot` | 331 | - | 3 |
| `>` | 498 | - | 2 |
| `>s` | 117 | - | 1 |
| `strlen` | 173 | - | 1 |
| `concat` | 142 | 135 | 3 |
| `divmod` | 1116 | - | 6 |
| `sha256` | 87 | 134 | 2 |
| `ash` | 596 | - | 3 |
| `lsh` | 277 | - | 3 |
| `not`, `any`, `all` | 200 | 300 | - |
| `point_add` | 101094 | 1343980 | - |
| `pubkey_for_exp` | 1325730 | - | 38 |
| | | | |
| `CREATE_COIN` | 1800000 | - | - |
| `AGG_SIG_UNSAFE`,`AGG_SIG_ME` | 1200000 | - | - |

<details>
<summary>原文参考</summary>

- ## Costs

When a clvm program is run, a cost is attributed to it. The minimum program cost is 40. After each opcode is run, its cost is added to the total program cost. When the cost exceeds a threshold, the program is terminated, and no value is returned. Also, if the number of atoms or pairs exceeds 2^31, the program is terminated and no value is returned.

| operator | base cost | cost per arg | cost per byte |
| -------- | --------- | ------------ | ------------- |
| `f` | 30 | - | - |
| `i` | 33 | - | - |
| `c` | 50 | - | - |
| `r` | 30 | - | - |
| `l` | 19 | - | - |
| `q` | 20 | - | - |
| `a` | 90 | - | - |
| `=` | 117 | - | 1 |
| `+` | 99 | 320 | 3 |
| `/` | 988 | - | 4 |
| `*` | 92 | 885 | [see here](https://github.com/Chia-Network/clvm_tools/blob/main/costs/README.md#multiplication) |
| `logand`, `logior`, `logxor` | 100 | 264| 3 |
| `lognot` | 331 | - | 3 |
| `>` | 498 | - | 2 |
| `>s` | 117 | - | 1 |
| `strlen` | 173 | - | 1 |
| `concat` | 142 | 135 | 3 |
| `divmod` | 1116 | - | 6 |
| `sha256` | 87 | 134 | 2 |
| `ash` | 596 | - | 3 |
| `lsh` | 277 | - | 3 |
| `not`, `any`, `all` | 200 | 300 | - |
| `point_add` | 101094 | 1343980 | - |
| `pubkey_for_exp` | 1325730 | - | 38 |
| | | | |
| `CREATE_COIN` | 1800000 | - | - |
| `AGG_SIG_UNSAFE`,`AGG_SIG_ME` | 1200000 | - | - |

</details>
