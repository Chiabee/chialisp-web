---
id: debugging
title: 9 - 调试
---

> Debugging

由于 Chialisp 程序的性质，通常很难确定到底哪里出了问题。由于 Chialisp 在运行之前被序列化为 CLVM，因此您收到的错误在您编写错误代码的上下文中通常看起来毫无意义。现在让我们回顾一些技巧，您可以使用它们来更轻松地捕捉程序中的错误。

<details>
<summary>原文参考</summary>

Due to the nature of Chialisp programs, it can often be difficult to determine where exactly something is going wrong.
Since Chialisp is serialized to CLVM before it is run, errors that you receive will often appear to make little sense within the context in which you wrote the faulty code.
Let's go over some tricks now that you can use to make catching the bugs in your program a little easier.

</details>

## 详细输出

`run` 和 `brun` 都有一个用于打印详细输出的 `-v` 标志。此输出*非常*详细，并显示程序在完成或退出之前所做的每个评估。让我们看一个例子：

```chialisp
brun '(c (sha256 0xdeadbeef) ())' '()' -v

FAIL: path into atom ()

(a 2 3) [((c (sha256 0xdeadbeef) ()))] => (didn't finish)

3 [((c (sha256 0xdeadbeef) ()))] => ()

2 [((c (sha256 0xdeadbeef) ()))] => (c (sha256 0xdeadbeef) ())

(c (sha256 0xdeadbeef) ()) [()] => (didn't finish)

() [()] => ()

(sha256 0xdeadbeef) [()] => (didn't finish)

0xdeadbeef [()] => (didn't finish)
```

每个详细的输出都以 `(a 2 3)` 开头，它简单地代表了正在运行的整个谜题和完整的解决方案。 如果您正在调试，这可能会输出 `（未完成）`。 我们可以追踪 `（未完成）` 的出现，直到找到最严重的失败评估。在这个例子中，我们看到它试图将 `0xdeadbeef` 作为程序运行以访问解决方案中的值。 解决方案只是`()`，显然不够深，所以会抛出错误。我们应该在将原子传递给 `sha256` 之前引用它。

<details>
<summary>原文参考</summary>

- ## Verbose output

Both `run` and `brun` have a `-v` flag for printing verbose outputs.
This output is *very* verbose and shows every evaluation that the program made before it finished or exited.
Let take a look at an example:

```chialisp
brun '(c (sha256 0xdeadbeef) ())' '()' -v

FAIL: path into atom ()

(a 2 3) [((c (sha256 0xdeadbeef) ()))] => (didn't finish)

3 [((c (sha256 0xdeadbeef) ()))] => ()

2 [((c (sha256 0xdeadbeef) ()))] => (c (sha256 0xdeadbeef) ())

(c (sha256 0xdeadbeef) ()) [()] => (didn't finish)

() [()] => ()

(sha256 0xdeadbeef) [()] => (didn't finish)

0xdeadbeef [()] => (didn't finish)
```

Every verbose output starts with `(a 2 3)` which simply represents the whole puzzle being run with whole solution. If you're debugging, this will likely have an output of `(didn't finish)`. We can trace the appearances of `(didn't finish)` down until we find the deepest failure to evaluate.
In this example, we see that it is trying to run `0xdeadbeef` as a program to access a value in the solution.
The solution is just `()` which is obviously not deep enough, so it throws an error.
We should have quoted the atom before we passed it to `sha256`.

</details>

## 常见错误

### 进入原子的路径

此错误可能是您运行新程序时最常见的错误。 这意味着您试图遍历索引比树更深的树。

这通常试图传达的是，您尝试引用的变量有问题。 确保检查您的参数是否正确地从一个函数传递到下一个函数，并且您的所有代码都在正确的范围内引用它们。 也许你调用了一个函数并且没有传递足够的参数。 也许函数期待一个程序，而你给了它一个原子。 您可以查看详细输出以查看哪些评估未完成，以了解哪些部分可能会失败。

### 第一个/其余的 non-cons

出现此错误，clvm 试图告诉您，您已尝试在原子上使用 `f` 或 `r` 而不是 cons 盒子。这又通常是由于参数未对齐。确保您知道什么 每个变量在传递给另一个函数时都可以是：原子、cons 盒子或两者之一。如果可以是任何一个，请确保在对其执行列表操作之前检查它是否是 cons。 有时这可能是由于评估一个意外长度的列表并在您期望它之前遇到了 `()` 。此外，仔细检查您程序中的所有评估是否都在正确的时间发生。也许一个程序被评估为 一个原子太快了。

### 列表上的 sha256

此错误具有相当的描述性，但重要的是要突出显示这种情况最常发生的时间。通常在构建程序时，您会希望使用其他数据散列对某种 CLVM 程序的承诺。通常这是通过树散列来完成的 程序使用 [sha256tree](https://chialisp.com/docs/common_functions#sha256tree1) 然后以这种方式提交。但是，由于许多应用程序的复杂性和移动部分，您可能会忘记哪些元素是 程序以及哪些元素只是树哈希。此错误通常表明您在应该传入树哈希时传入了程序。转到应用程序中对 `sha256` 的每一个引用，你可能会找到罪魁祸首。

<details>
<summary>原文参考</summary>

- ## Common errors

- ### path into atom

This error is perhaps the most common error that will come up when you run a new program.
It means that you have tried to traverse a tree with an index that is deeper than the tree is.


What this is usually trying to convey is that something is wrong with a variable that you are trying to reference.
Make sure to check your arguments are being properly passed from one function to the next and that all of your code is referencing them within the correct scope.
Maybe you called a function and didn't pass it enough parameters.
Maybe the function was expecting a program and you gave it an atom.
You can look in the verbose output to see what evaluations didn't finish to get a clue of what part might be failing.

- ### first/rest of non-cons

With this error, clvm is trying to tell you that you have attempted to use `f` or `r` on an atom instead of a cons box.
This is, again, usually due to a misalignment of arguments.
Make sure you know what every variable is allowed to be when it gets passed to another function: an atom, a cons box, or either.
If it can be either, make sure you check if it is a cons before performing list operations on it. Sometimes this can be caused by evaluating a list of an unexpected length and running into `()` before you expect it.
Also, double check that all of the evaluation in your program is happening at the right time.
Perhaps a program was evaluated into an atom too soon.

- ### sha256 on list

This error is fairly descriptive, but it is important to highlight when this most commonly occurs.
Often when building a program, you will want to hash a commitment to some kind of CLVM program with some other data.
Usually this is done by tree hashing the program using [sha256tree](https://chialisp.com/docs/common_functions#sha256tree1) and then committing to it that way.
However, with the complexity and moving pieces of a lot of applications, you may lose track of which elements are programs and which elements are just tree hashes.
This error often indicates that you are passing in a program when you should be passing in a tree hash.
Go to every reference of `sha256` in your application and you can probably find the culprit.

</details>

## 使用 `(x)` 来记录

通常，您希望能够在程序执行过程中看到变量的值。 大多数语言都有某种日志语句来执行此操作，但在 Chialisp 中实现它有点不可能，因为它是评估而不是运行。 您可以使用的解决方法之一是将您要调试的语句包装在 `x` 中。 raise 运算符在引发时采用一个可选参数来记录。 假设您正在尝试调试这行代码：

```chialisp
(list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
```

您可以尝试注释掉该行并创建一个新的加注以退出一些信息：

```chialisp
; (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
(x (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data))))
```

请记住，评估将在创建提升消息之前进行。有时最好只列出参数列表：

```chialisp
; (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
(x (list coin-info coin-data))
```

当您尝试调试一系列按顺序发生的支出时，还会出现一个警告。 也许谜语第一次运行，第二次失败。 如果你在执行过程中加注，你可能也会导致你的第一个谜语出错，这不会让你进入第二个谜语。 在这样的场景中，尝试找出支出之间的差异并将加薪包裹在一个 `if` 中，以便您可以安全地通过第一个谜语。

<details>
<summary>原文参考</summary>

- ## Using `(x)` to log

Oftentimes, you would like to be able to see the values of a variable in the middle of a program execution.
Most languages have some sort of log statement with which to do this, but it's somewhat impossible to implement in Chialisp since it's evaluated rather than run.
One of the workarounds you can use is to wrap the statement you are looking to debug in `x`.
The raise operator takes an optional argument to log when it raises.
Let's say you are trying to debug this line of code:

```chialisp
(list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
```

You can try commenting out that line and creating a new raise to exit out with some information:

```chialisp
; (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
(x (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data))))
```

Keep in mind that evaluation will happen before the raise message gets created.
Sometimes it's better to just raise a list of the arguments:

```chialisp
; (list CREATE_COIN_ANNOUNCEMENT (sha256tree (list coin-info coin-data)))
(x (list coin-info coin-data))
```

There is also a caveat that occurs when you are trying to debug a series of spends that happen sequentially.
Maybe the puzzle runs the first time and fails the second time.
If you raise during execution, you may cause your first puzzle error out too, which will not get you to the second puzzle.
In scenarios like these, try to figure out a difference between the spends and wrap the raise in an `if` so that you can pass safely through the first puzzle.

</details>

## main.sym

当你在包含常量或函数的 `mod` 上使用 `run` 时，编译器会自动生成一个名为 `main.sym` 的文件。这个文件包含从常量或函数名称到它们在字节码中的表示的映射。当你正在使用 `brun` 运行程序，您可以使用 `-y` 标志指定符号文件。然后，当您看到错误或打印详细输出时，您将看到人类可读的文本，而不是正在使用的整数或字节码参考它。

这在处理冗长的输出时特别有用。 您可以向上滚动日志，直到您识别出未完成的代码片段。 如果没有符号表，识别起来可能会困难得多。

重要的是，符号表将无法识别内联函数或宏，因为它们是在编译时插入的。 如果您正在调试，最好将内联函数更改为函数，以便您可以在符号表中识别它们。

<details>
<summary>原文参考</summary>

- ## main.sym

When you use `run` on a `mod` that contains constants or functions, the compiler will automatically generate a file called `main.sym`.
This file contains mappings from the constant or function names to their representations in the bytecode.
When you are running the program with `brun`, you can specify the symbol file with the `-y` flag.
Then, when you see errors or print verbose outputs, you will see human readable text rather than the integer or bytecode that is being used to refer to it.

This is particularly useful when dealing with long verbose outputs.
You can scroll up the log until you recognize a snippet of code that isn't finishing.
Without the symbol table, it may be much more difficult to recognize.

Importantly, the symbol table will not be able to identify inline functions or macros since they are inserted at compile time.
If you are debugging, it's probably a good idea to change inline functions into functions so that you can recognize them in the symbol table.

</details>

## `opd` 和 `opc`

[clvm_tools 仓库](https://github.com/Chia-Network/clvm_tools)中还有两条与 CLVM 序列化相关的命令形式。 有时查看序列化编译会很有帮助。例如，当评估一个程序的成本时，它会为谜语揭示中的每个字节收取成本。您被激励确保拼图揭示尽可能小。

如果您想查看序列化的输出，可以使用 `opc` 来编译或 **组装** CLVM：

```chialisp
opc '(q "hello" . "world")'
ff01ff8568656c6c6f85776f726c64
```

此外，其他语言（如 Python）通常也以序列化格式处理 CLVM。如果您正在为谜语编写驱动程序代码，则可能需要调试包含一些序列化 CLVM 的支出包。在这种情况下，将序列化程序**反汇编**为人类可读的形式会很有用。

```chialisp
opd ff01ff8568656c6c6f85776f726c64
(q "hello" . "world")
```

对于大型程序，以人类可读的形式可能不会更清晰，但您通常仍然可以区分某些模式。例如，柯里化的参数相对容易挑选，它们通常可以为您提供调试程序所需的关键信息。

<details>
<summary>原文参考</summary>

- ## `opd` and `opc`

There are two more commands in the [clvm_tools repository](https://github.com/Chia-Network/clvm_tools) that are related to the serialization of CLVM.
When the program is run on the blockchain, it is run in its serialized form. It can sometimes be helpful to see that serialized compilation. For example, when the cost of a program is evaluated, it is charged cost for every byte in the puzzle reveal.
You are incentivized to make sure that puzzle reveal is as small as possible.

If you would like to see the serialized output, you can use `opc` to compile or **assemble** the CLVM:

```chialisp
opc '(q "hello" . "world")'
ff01ff8568656c6c6f85776f726c64
```

In addition, other languages like Python usually also handle CLVM in its serialized format.
If you are writing driver code for your puzzles, you may need to debug a spend bundle that contains some serialized CLVM. In this scenario, it can be useful to **disassemble** the serialized program into the human readable form.

```chialisp
opd ff01ff8568656c6c6f85776f726c64
(q "hello" . "world")
```

With large programs, it may not be much clearer in the human readable form, but you can often still distinguish certain patterns. Curried arguments, for example, are relatively easy to pick out and they can often give you the crucial information you need to debug your programs.

</details>

## 结论

有时调试 Chialisp 会令人沮丧。 由于 lisp 处理数据结构的方式的性质，程序通常会继续使用不正确的值，只是在稍后的地方出错，而不会提供初始损坏的线索。 例如，变量拼写错误通常会导致变量被评估为字符串，如果它被散列成某些东西，则无法分辨！

建议您充分掌握 CLVM，因为它是 Chialisp 中发生的所有过程的基础。 它将更容易在您的脑海中构建正在发生的评估以及它们可能会意外发生的原因的图片。

希望通过这些技巧，您可以节省一些时间并更快地获得智能硬币。

<details>
<summary>原文参考</summary>

- ## Conclusion

Debugging Chialisp at times can be frustrating. Due to the nature of how lisp handles data structures, programs will often continue on with incorrect values only to error out at a later spot that gives no clue to the initial breakage. For example, a variable typo will often result in the variable being evaluated as a string, and if that gets hashed into something it's impossible to tell!

It is recommended that you have a strong grasp of CLVM since it underlies all of the processes that happen in Chialisp. It will make it easier to build the picture in your head of the evaluations that are happening and why they may be happening unexpectedly.

Hopefully with these tricks you can save yourself a bit of time and get your smart coins out quicker.

</details>
