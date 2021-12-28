---
id: common_functions
title: 5 - Chialisp 中的常用函数
---

> Common Functions in Chialisp

当您开始编写完整的智能硬币时，您将开始意识到在许多谜题中您将需要某些通用功能。让我们来看看如何包含它们以及其中一些是什么：

<details>
<summary>原文参考</summary>

When you start to write full smart coins, you will start to realize that you will need certain common functionality in a lot of puzzles.
Let's go over how to include them and what some of them are:

</details>

## include

如果你想导入一些你经常使用的功能而不必在文件之间复制/粘贴它，你可以使用 `include`：

```chialisp
;; condition_codes.clvm
(
  (defconstant AGG_SIG_ME 50)
  (defconstant CREATE_COIN 51)
)
```

```chialisp
;;main.clvm
(mod (pubkey msg puzzle_hash amount)

  (include "condition_codes.clvm")

  (list (list AGG_SIG_ME pubkey msg) (list CREATE_COIN puzzle_hash amount))

)
```

使用 `run` 运行 main.clvm 时，请确保使用 `-i` 选项指定在哪些目录中查找可包含的文件。如果我们的 condition_codes.clvm 文件在目录 `./libraries/chialisp/` 中，那么你可以将它传递给 `run`，以便它知道在哪里可以找到它：

```
run -i ./libraries/chialisp/ main.clvm
```

另请注意，包含文件是一种特殊格式。 定义的所有内容都放入一组括号中，就像上面的 condition_codes.clvm 一样。 然后，您可以在编写程序时使用这些常量/函数中的任何一个，而无需单独导入每个常量/函数。 编译器将只包含您使用的内容，因此在尝试优化程序大小时不要担心包含大型库文件。

<details>
<summary>原文参考</summary>

- ## include

If you want to import some functionality that you use frequently without having to copy/paste it between files, you can use `include`:

```chialisp
;; condition_codes.clvm
(
  (defconstant AGG_SIG_ME 50)
  (defconstant CREATE_COIN 51)
)
```

```chialisp
;;main.clvm
(mod (pubkey msg puzzle_hash amount)

  (include "condition_codes.clvm")

  (list (list AGG_SIG_ME pubkey msg) (list CREATE_COIN puzzle_hash amount))

)
```

When running main.clvm with `run`, make sure to use the `-i` option to specify in which directories to look for includable files.
If our condition_codes.clvm file was in the directory `./libraries/chialisp/` then you would pass that to `run` so that it knows where to find it:

```
run -i ./libraries/chialisp/ main.clvm
```

Also note that the include files are a special format. Everything that is defined goes into a single set of parentheses like in condition_codes.clvm above.
You can then use any of those constants/functions when writing your program, without having to import each one individually.
The compiler will only include things that you use, so don't worry about including a large library file when attempting to optimize the size of your program.

</details>

## sha256tree

当谜语被哈希时，它们不会被简单地序列化并传递给 sha256。 相反，我们采用谜语的树哈希。

回想一下，每个 clvm 程序都可以表示为二叉树。 每个对象要么是一个原子（树的叶子），要么是一个 cons 盒子（树的一个分支）。 当我们对谜语进行散列时，我们从树的叶子开始向上哈希，连接一个 1 或一个 2 来表示它是一个原子或一个 cons 盒子。 一旦 cons 盒子被哈希，它就会成为一个新的叶子，被哈希到其父 cons 盒子中，并且该过程会递归。 这是 Chialisp 中的样子：

```chialisp
(defun sha256tree
   (TREE)
   (if (l TREE)
       (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
       (sha256 1 TREE)
   )
)
```

在 Chialisp 谜语中计算树哈希非常有用。您可以断言其他硬币的谜语，压缩谜语以便于签名，并根据一些传入的数据创建 CREATE_COIN 条件。

<details>
<summary>原文参考</summary>

- ## sha256tree

When puzzles are hashed, they are not simply serialized and passed to sha256.
Instead, we take the *tree hash* of the puzzle.

Recall that every clvm program can be represented as a binary tree.
Every object is either an atom (a leaf of the tree) or a cons box (a branch of the tree).
When we hash the puzzle we start at the leaves of the tree and hash our way up, concatenating either a 1 or a 2 to denote that it's either an atom or a cons box.
Once a cons box is hashed, it becomes a new leaf to be hashed into its parent cons box and the process recurses.
Here's what that looks like in Chialisp:

```chialisp
(defun sha256tree
   (TREE)
   (if (l TREE)
       (sha256 2 (sha256tree (f TREE)) (sha256tree (r TREE)))
       (sha256 1 TREE)
   )
)
```

It is extremely useful to calculate tree hashes within a Chialisp puzzle.
You can assert puzzles of other coins, condense puzzles for easier signing, and make CREATE_COIN conditions that are dependent on some passed in data.

</details>

## Currying

Currying 是 Chialisp 中一个极其重要的概念，它几乎负责整个状态在硬币中的存储方式。 这个想法是在散列之前将参数传递给谜题。 当你 Currying 时，你承诺解决方案的价值，这样解决难题的个人就不能改变它们。 我们来看看这是如何在 Chialisp 中实现的：

```chialisp
;; utility function used by curry
(defun fix_curry_args (items core)
 (if items
     (qq (c (q . (unquote (f items))) (unquote (fix_curry_args (r items) core))))
     core
 )
)

; (curry sum (list 50 60)) => returns a function that is like (sum 50 60 ...)
(defun curry (func list_of_args) (qq (a (q . (unquote func)) (unquote (fix_curry_args list_of_args (q . 1))))))
```

这如此有用的原因是因为您可能想要创建谜语的蓝图，但每次创建时对某些参数使用不同的值。 您不能依赖解谜器诚实、正确地传递您想要使用的信息，因此您需要确保在他们有机会解决它之前传递了它。

上面的函数可能看起来很复杂，但它真正做的就是将函数包装在一个 `a` 中，并将参数添加到 `1` 前面，它（编译到 clvm 时）将引用其余的谜语参数。 没有所有的引号，上面的代码简化为这样的：

```chialisp
(a func (c curry_arg_1 (c curry_arg_2 1)))
```

你也可以做相反的操作。给定一个程序，您可以使用简单的`(f (r (r ))) 来*uncurry* 参数列表：

```chialisp
(f (r (r curried_func)))
; (c curry_arg_1 (c curry_arg_2 1))
```

让我们以之前的密码锁定硬币为例，这次是 Chialisp 难题：

```chialisp
(mod (password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (defun check_password (password new_puzhash amount)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
      (list (list CREATE_COIN new_puzhash amount))
      (x)
    )
  )

  ; main
  (check_password password new_puzhash amount)
)
```

您可以看到密码散列被烘焙到拼图的来源中。这意味着每次要使用新密码锁定硬币时，都必须重新创建包含代码源的文件。如果我们完全概括它会更好：


```chialisp
(mod (PASSWORD_HASH password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (defun check_password (PASSWORD_HASH password new_puzhash amount)

    (if (= (sha256 password) PASSWORD_HASH)
      (list (list CREATE_COIN new_puzhash amount))
      (x)
    )
  )

  ; main
  (check_password PASSWORD_HASH password new_puzhash amount)
)
```

但是，现在我们遇到的问题是，任何人都可以输入他们喜欢的任何密码/哈希组合并解锁该硬币。当我们创建这个硬币时，我们需要提交密码哈希。在确定我们将要创建的硬币的拼图哈希之前，我们需要在哈希中使用如下内容：


```chialisp
; curry_password_coin.clvm
(mod (password_hash password_coin_mod)
  (include "curry.clvm") ; From above

  (curry password_coin_mod (list password_hash))
)
```

如果我们编译这个函数并像这样传递参数：

```
brun <curry password coin mod> '((q . 0xcafef00d) (q . <password coin mod>))'
```

我们将收到一个看起来与我们的密码硬币模块非常相似的谜题，但已扩展为包括我们传入的哈希值。您现在可以使用不同的密码哈希值运行上面的柯里化模块，它每次都会输出一个新的谜题。 然后我们可以对这个谜题进行散列，并使用返回的谜题散列创建一个硬币。

请注意，这要求我们在我们自己的链下环境中使用 `brun` 运行柯里化模块，以创建用于锁定硬币的拼图。 很多时候这种柯里化会发生在 python 或创建硬币的软件使用的任何包装语言中。 但是，在某些用例中，我们希望在拼图的范围内使用柯里化。 现在让我们看一个。

<details>
<summary>原文参考</summary>

- ## Currying

Currying is an extremely important concept in Chialisp that is responsible for almost the entirety of how state is stored in coins.
The idea is to pass in arguments to a puzzle *before* it is hashed.
When you curry, you commit to solution values so that the individual solving the puzzle cannot change them.
Let's take a look at how this is implemented in Chialisp:

```chialisp
;; utility function used by curry
(defun fix_curry_args (items core)
 (if items
     (qq (c (q . (unquote (f items))) (unquote (fix_curry_args (r items) core))))
     core
 )
)

; (curry sum (list 50 60)) => returns a function that is like (sum 50 60 ...)
(defun curry (func list_of_args) (qq (a (q . (unquote func)) (unquote (fix_curry_args list_of_args (q . 1))))))
```

The reason this is so useful is because you may want to create the blueprint of a puzzle, but use different values for certain parameters every time you create it.
You can't rely on the puzzle solver to honestly and correctly pass in the information you want to use, so you need to make sure it is passed in before they ever get the chance to solve it.

The above function may look complex, but all it's really doing is wrapping the function in an `a` and prepending the arguments to `1` which (when compiled to clvm) will refer the rest of the puzzle arguments.
Absent of all the quotes, the above code reduces to something like this:

```chialisp
(a func (c curry_arg_1 (c curry_arg_2 1)))
```

You can also do the reverse operation.
Given a program, you can *uncurry* the list of arguments, with a simple `(f (r (r )))`:

```chialisp
(f (r (r curried_func)))
; (c curry_arg_1 (c curry_arg_2 1))
```

Let's take our password locked coin example from earlier, this time as a Chialisp puzzle:

```chialisp
(mod (password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (defun check_password (password new_puzhash amount)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
      (list (list CREATE_COIN new_puzhash amount))
      (x)
    )
  )

  ; main
  (check_password password new_puzhash amount)
)
```

You can see that the password hash is baked into the source of the puzzle.
This means every time that you want to lock up a coin with a new password, you have to recreate the file that contains the source of the code.
It would be much nicer if we fully generalized it:

```chialisp
(mod (PASSWORD_HASH password new_puzhash amount)
  (defconstant CREATE_COIN 51)

  (defun check_password (PASSWORD_HASH password new_puzhash amount)

    (if (= (sha256 password) PASSWORD_HASH)
      (list (list CREATE_COIN new_puzhash amount))
      (x)
    )
  )

  ; main
  (check_password PASSWORD_HASH password new_puzhash amount)
)
```

However, now we have the problem that anyone can pass in whatever password/hash combo that they please and unlock this coin.
When we create this coin we need the password hash to be committed to. Before determining the puzzle hash of the coin we're going to create, we need to curry in the hash with something like this:

```chialisp
; curry_password_coin.clvm
(mod (password_hash password_coin_mod)
  (include "curry.clvm") ; From above

  (curry password_coin_mod (list password_hash))
)
```

If we compile this function and pass it parameters like this:

```
brun <curry password coin mod> '((q . 0xcafef00d) (q . <password coin mod>))'
```

we will receive a puzzle that looks very similar to our password coin module, but has been expanded to include the hash we passed in.
You can now run the currying mod above with a different password hash and it will output a new puzzle every time.
We can then hash that puzzle and create a coin with the returned puzzle hash.

Note that this required that we run the currying module using `brun` in our own environment off chain in order to create the puzzle we would lock up our coin with.
A lot of the time this currying will happen in python or whatever wrapper language is being used by the software creating the coins.
However, there are some use cases in which we would want to use currying within the scope of a puzzle.
Let's look at one now.

</details>

## 外部谜语和内部谜语

一种常见的设计模式，也是 Chialisp 最强大的功能之一，是能够拥有一个“包裹”内部拼图的外部拼图。 这个概念非常方便，因为它允许硬币在内部谜语内保留它的所有标准功能和可编程性，但由外部谜语绑定到一组额外的规则。

在这个例子中，我们将继续密码锁定，但这次我们将要求每次花费硬币时，都需要设置一个新密码。 让我们看一下所有代码，然后我们将其分解：

```chialisp
(mod (
    MOD_HASH        ;; curried in
    PASSWORD_HASH   ;; curried in
    INNER_PUZZLE    ;; curried in
    inner_solution
    password
    new_password_hash
  )

  (include "condition_codes.clvm")
  (include "sha256tree.clvm")
  (include "curry-and-treehash.clvm")

  (defun pw-puzzle-hash (MOD_HASH mod_hash_hash new_password_hash_hash inner_puzzle_hash)
     (puzzle-hash-of-curried-function
       MOD_HASH
       inner_puzzle_hash new_password_hash_hash mod_hash_hash ; parameters must be passed in reverse order
     )
   )

  ;; tweak `CREATE_COIN` condition by wrapping the puzzle hash, forcing it to be a password locked coin
  (defun-inline morph-condition (condition new_password_hash MOD_HASH)
   (if (= (f condition) CREATE_COIN)
     (list CREATE_COIN
       (pw-puzzle-hash MOD_HASH (sha256tree MOD_HASH) (sha256tree new_password_hash) (f (r condition)))
       (f (r (r condition)))
     )
     condition
   )
  )

  ;; tweak all `CREATE_COIN` conditions, enforcing created coins to be locked by passwords
  (defun morph-conditions (conditions new_password_hash MOD_HASH)
   (if conditions
     (c
       (morph-condition (f conditions) new_password_hash MOD_HASH)
       (morph-conditions (r conditions) new_password_hash MOD_HASH)
     )
     ()
   )
  )

  ; main
  (if (= (sha256 password) PASSWORD_HASH)
    (morph-conditions (a INNER_PUZZLE inner_solution) new_password_hash MOD_HASH)
    (x "wrong password")
  )

)
```

您可能会注意到我们导入了一个名为 `curry-and-treehash` 的新库。 我们将在几个步骤中讨论这个问题。

首先，让我们谈谈论点。 当您第一次创建这个谜语时，您需要输入 3 项内容：`MOD_HASH` ，它是此代码的树形哈希，不包含任何 currying 参数；`PASSWORD_HASH` ，它是解锁此硬币的密码的哈希，以及 `INNER_PUZZLE` 这是一个完全独立的谜题，对于如何使用硬币有自己的规则。

Chialisp 谜语倾向于自下而上阅读，所以让我们从这一块开始：

```chialisp
; main
(if (= (sha256 password) PASSWORD_HASH)
  (morph-conditions (a INNER_PUZZLE inner_solution) new_password_hash MOD_HASH)
  (x "wrong password")
)
```

这里发生的所有事情是我们要确保密码正确，如果正确，我们将在 `INNER_PUZZLE` 中运行柯里化并传入 `inner_solution`。 这将返回我们将传递给下一个函数的条件列表以及新密码哈希和`MOD_HASH`。

```chialisp
;; tweak all `CREATE_COIN` conditions, enforcing created coins to be locked by passwords
(defun morph-conditions (conditions new_password_hash MOD_HASH)
 (if conditions
   (c
     (morph-condition (f conditions) new_password_hash MOD_HASH)
     (morph-conditions (r conditions) new_password_hash MOD_HASH)
   )
   ()
 )
)
```

递归是 Chialisp 的基础，编写它时经常会出现这样的函数。 为了遍历条件列表，我们首先检查是否还有剩余的项目（请记住，空列表 `()` 或 **nil** 的计算结果为 false）。 然后，我们对第一个条件进行变形并将其与列表其余部分的递归输出连接起来。 最后，我们将以相同的顺序拥有相同的项目列表，但它们都将通过 `morph-condition`。

```chialisp
;; tweak `CREATE_COIN` condition by wrapping the puzzle hash, forcing it to be a password locked coin
(defun-inline morph-condition (condition new_password_hash MOD_HASH)
 (if (= (f condition) CREATE_COIN)
   (list CREATE_COIN
     (pw-puzzle-hash MOD_HASH (sha256tree MOD_HASH) (sha256tree new_password_hash) (f (r condition)))
     (f (r (r condition)))
   )
   condition
 )
)
```

这个功能也很简单。 我们首先检查操作码（列表中的第一项）是否为 CREATE_COIN。 如果不是，只需像往常一样返回条件。 如果是，则返回一个几乎完全相同的条件，除了我们将拼图哈希传递到将修改它的函数中：

```chialisp
(defun pw-puzzle-hash (MOD_HASH mod_hash_hash new_password_hash_hash inner_puzzle_hash
   (puzzle-hash-of-curried-function
     MOD_HASH
     inner_puzzle_hash new_password_hash_hash mod_hash_hash ; parameters must be passed in reverse order
   )
)
```

这就是令人兴奋的事情发生的地方。由于我们不知道内部拼图，只知道它的哈希值，因此无法将其直接柯里化到我们要创建的下一个谜语中。此外，如果我们不想在每次使用当前模块时都传入它的整个源代码，那么我们也没有什么难题可以将其放入其中。

然而，我们所关心的只是为下一个谜语生成正确的*谜语哈希*，我们确实有这个模块和内部谜语的树哈希。 我们可以使用 `puzzle-hash-of-curried-function`，它允许我们创建给定函数的谜语哈希：a) 该函数的谜语哈希和 b) 其所有参数的谜语哈希以相反的顺序*好像它们是树哈希*的一部分。 这意味着原子和数字的参数应该采用树哈希形式，带有 1 前缀，如 `(sha256 (q . 1) my-argument-value)` 并且 `sha256tree` 的输出适用于任何涉及 cons 细胞。 如果`puzzle-hash-of-curried-function` 本身采用参数值，则有可能猜测这些，但这可能需要重新计算昂贵的哈希值。

这个库的其他实现细节在本教程的这一部分中有点多，但本质上，它允许我们*恢复*我们已经完成的树哈希，除了最后一步。

就是这样！创建此币时，它只能通过散列到 PASSWORD\_HASH 中的咖 currying 的密码使用。内部拼图可以是您想要的任何东西，包括其他具有自己内部拼图的外部谜语。由于该内部谜语而产生的任何硬币都将被同一个外部拼图“包裹”，确保该硬币的每个子代都被密码*永远*锁定。

我们创建了一个简单的硬币，但您可以看到它的潜力。您不仅可以对您锁定的硬币执行一组规则，还可以对*每个*后代硬币执行一组规则。不仅如此，这些规则还可以*在其他智能硬币之上*执行。在 Chialisp 生态系统中，除非堆栈中的某个谜题另有说明，否则所有智能币都可以相互操作。可能性是无限的，代表了 Chialisp 为硬币提供的巨大可编程性。

在下一节中，我们将讨论 Chia 网络上的标准交易格式。

<details>
<summary>原文参考</summary>

- ## Outer and Inner puzzles

A common design pattern, and one of the most powerful features of Chialisp, is the ability to have an outer puzzle that "wraps" an inner puzzle.
This concept is extremely handy because it allows a coin to retain all of it's standard functionality and programmability within the inner puzzle, but be bound to an extra set of rules by the outer puzzle.

For this example, we're going to continue with our password locking, but this time we're going to require that every time the coin is spent, it requires a new password to be set.
Let's look at all the code and then we'll break it down:

```chialisp
(mod (
    MOD_HASH        ;; curried in
    PASSWORD_HASH   ;; curried in
    INNER_PUZZLE    ;; curried in
    inner_solution
    password
    new_password_hash
  )

  (include "condition_codes.clvm")
  (include "sha256tree.clvm")
  (include "curry-and-treehash.clvm")

  (defun pw-puzzle-hash (MOD_HASH mod_hash_hash new_password_hash_hash inner_puzzle_hash)
     (puzzle-hash-of-curried-function
       MOD_HASH
       inner_puzzle_hash new_password_hash_hash mod_hash_hash ; parameters must be passed in reverse order
     )
   )

  ;; tweak `CREATE_COIN` condition by wrapping the puzzle hash, forcing it to be a password locked coin
  (defun-inline morph-condition (condition new_password_hash MOD_HASH)
   (if (= (f condition) CREATE_COIN)
     (list CREATE_COIN
       (pw-puzzle-hash MOD_HASH (sha256tree MOD_HASH) (sha256tree new_password_hash) (f (r condition)))
       (f (r (r condition)))
     )
     condition
   )
  )

  ;; tweak all `CREATE_COIN` conditions, enforcing created coins to be locked by passwords
  (defun morph-conditions (conditions new_password_hash MOD_HASH)
   (if conditions
     (c
       (morph-condition (f conditions) new_password_hash MOD_HASH)
       (morph-conditions (r conditions) new_password_hash MOD_HASH)
     )
     ()
   )
  )

  ; main
  (if (= (sha256 password) PASSWORD_HASH)
    (morph-conditions (a INNER_PUZZLE inner_solution) new_password_hash MOD_HASH)
    (x "wrong password")
  )

)
```

You may notice that we imported a new library called `curry-and-treehash`.
We'll talk about that in a few steps.

First, let's talk about the arguments.
When you create this puzzle for the first time you need to curry in 3 things: `MOD_HASH` which is the tree hash of this code with no curried arguments, `PASSWORD_HASH` which is the hash of the password that will unlock this coin, and `INNER_PUZZLE` which is a completely separate puzzle that will have its own rules about how the coin can be spent.

Chialisp puzzles have the tendency to be read from the bottom up, so lets start with this chunk:

```chialisp
; main
(if (= (sha256 password) PASSWORD_HASH)
  (morph-conditions (a INNER_PUZZLE inner_solution) new_password_hash MOD_HASH)
  (x "wrong password")
)
```

All that's happening here is that we're making sure the password is correct and, if it is, we're going to run the curried in `INNER_PUZZLE` with the passed in `inner_solution`.
This will return a list of conditions that we will pass to the next function along with the new password hash and `MOD_HASH`.

```chialisp
;; tweak all `CREATE_COIN` conditions, enforcing created coins to be locked by passwords
(defun morph-conditions (conditions new_password_hash MOD_HASH)
 (if conditions
   (c
     (morph-condition (f conditions) new_password_hash MOD_HASH)
     (morph-conditions (r conditions) new_password_hash MOD_HASH)
   )
   ()
 )
)
```

Recursion is the foundation of Chialisp and functions like these very commonly show up when writing it.
In order to iterate through the list of conditions, we first check if there are still items left (remember that an empty list `()` or **nil** evaluates to false). Then, we morph the first condition and concatenate it with the recursive output of the rest of the list.
In the end, we will have the same list of items in the same order, but all of them will have passed thru `morph-condition`.

```chialisp
;; tweak `CREATE_COIN` condition by wrapping the puzzle hash, forcing it to be a password locked coin
(defun-inline morph-condition (condition new_password_hash MOD_HASH)
 (if (= (f condition) CREATE_COIN)
   (list CREATE_COIN
     (pw-puzzle-hash MOD_HASH (sha256tree MOD_HASH) (sha256tree new_password_hash) (f (r condition)))
     (f (r (r condition)))
   )
   condition
 )
)
```

This function is also pretty simple. We're first checking if the opcode (first item in the list) is CREATE_COIN.
If it's not, just return the condition as usual.
If it is, return a condition that is almost exactly the same, except we're passing the puzzle hash into a function that will modify it:

```chialisp
(defun pw-puzzle-hash (MOD_HASH mod_hash_hash new_password_hash_hash inner_puzzle_hash
   (puzzle-hash-of-curried-function
     MOD_HASH
     inner_puzzle_hash new_password_hash_hash mod_hash_hash ; parameters must be passed in reverse order
   )
)
```

This is where the exciting stuff happens.
Since we don't know the inner puzzle, only it's hash, it's impossible to curry it directly into the next puzzle we want to create.
Furthermore, if we don't want to pass in the whole source of this current module every time that we spend it, we don't have a puzzle to curry things into either.

However, all we care about is generating the correct *puzzle hash* for the next puzzle, and we do have the tree hashes for both this module and the inner puzzle.
We can use `puzzle-hash-of-curried-function` which allows us to create the puzzle hash of a function given: a) the puzzle hash of that function and b) the puzzle hashes of all of its arguments in reverse order _as though they were a part of a tree hash_.  This means that arguments that are atoms and numbers are expected to be in tree hash form, with a 1 prefix like ```(sha256 (q . 1) my-argument-value)``` and the output of `sha256tree` is suitable for anything involving cons cells.  It would be possible for `puzzle-hash-of-curried-function` to guess these if it took the parameter values themselves but that might require recomputation of expensive hashes.

Other implementation details of this library are a bit much to go into in this part of the tutorial but, in essence, it allows us to *resume* a tree hash that we have completed except for the last step.

And that's it!  When this coin is created, it can only be spent by a password that hashes to the curried in PASSWORD_HASH.
The inner puzzle can be anything that you want including other outer puzzles that have their own inner puzzles.
Whatever coins get created as a result of that inner puzzle will be "wrapped" by this same outer puzzle ensuring that every child of this coin is locked by a password *forever*.

We created a simple coin, but you can see the potential of this. You can enforce a set of rules not only on a coin that you lock up, but on *every* descendant coin.
Not only that, the rules can be enforced *on top of other smart coins*.
In the Chialisp ecosystem, all smart coins are interoperable with each other unless otherwise specified by one of the puzzles in the stack. The possibilities are endless and represent the vast programmability that Chialisp enables for coins.

In the next section, we'll talk about the standard transaction format on the Chia network.

</details>
