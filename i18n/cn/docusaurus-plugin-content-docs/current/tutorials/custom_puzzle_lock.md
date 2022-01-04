---
id: custom_puzzle_lock
title: 如何使用自定义谜语锁定硬币
sidebar_label: 使用自定义谜语锁定硬币
---

> How to lock a coin with a custom puzzle

本教程教您如何使用定制的谜语锁定硬币。

## 创建一个谜语

为您的硬币创建任何您想要的谜语。我们将使用密码锁定的硬币作为本教程的示例。

```chialisp
(mod (password new_puzhash amount)
    (defconstant CREATE_COIN 51)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
        (list (list CREATE_COIN new_puzhash amount))
        (x)
    )
)
```

`0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824` 是单词 `hello` 的sha256哈希。

**示例有效谜底：** `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)`。

如果密码正确，谜语将创建一个带有 2 mojo 的新硬币，并使用新的谜语哈希 `0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675` 锁定它。

**任何剩余的零钱都作为费用支付给农民**。

<details>
<summary>原文参考</summary>

This tutorial teaches you how to lock a coin with a custom-made puzzle.

- ## Create a puzzle

Create whatever puzzle you want for your coin. We'll use a password-locked coin as an example for this tutorial.

```chialisp
(mod (password new_puzhash amount)
    (defconstant CREATE_COIN 51)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
        (list (list CREATE_COIN new_puzhash amount))
        (x)
    )
)
```
`0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824` is the sha256 hash of the word `hello`.

**Example valid solution:** `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)`.

If password is correct, the puzzle will create a new coin with 2 mojos and lock it using a new puzzle hash `0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675`.

**Any remaining change goes to a farmer as a fee**.

</details>

## 获取自定义谜语的编译版本

有多种方法可以获得自定义谜语的编译版本。

### 使用 [clvm_tools](https://github.com/Chia-Network/clvm_tools)（官方）

您可以使用 clvm_tools 存储库（包含在官方 Chia 存储库中）从终端编译自定义拼图。您需要做的就是使用 `run` 命令，并将您的谜题作为参数提供。

```bash
run '(mod (password new_puzhash amount)
    (defconstant CREATE_COIN 51)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
        (list (list CREATE_COIN new_puzhash amount))
        (x)
    )
)'
```

结果在 `(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))`.

### 使用 [Chialisp 网络工具](https://clisp.surrealdev.com/)（非官方）

将您的自定义谜语粘贴到文本区域并点击 **Compile**。编译后的版本将出现在 **Run Output** 部分。

<details>
<summary>原文参考</summary>

- ## Get compiled version of the custom puzzle

There are multiple ways how you can get compiled version of your custom puzzle.

- ### Using [clvm_tools](https://github.com/Chia-Network/clvm_tools) (official)

You can compile your custom puzzle from the terminal using the clvm_tools repository (included in the official Chia repository). All you need to do is to use the `run` command with your puzzle provided as an argument.

```bash
run '(mod (password new_puzhash amount)
    (defconstant CREATE_COIN 51)

    (if (= (sha256 password) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824))
        (list (list CREATE_COIN new_puzhash amount))
        (x)
    )
)'
```
results in `(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))`.

- ### Using [Chialisp web tool](https://clisp.surrealdev.com/) (unofficial)

Paste your custom puzzle into the text area and hit **Compile**. The compiled version will appear in the **Run Output** section.

</details>

## 从谜语中获取谜语哈希

### 使用 [clvm_工具](https://github.com/Chia-Network/clvm_tools)

您可以使用官方 clvm_tools 存储库中包含的 `opc -H <compiled_puzzle>` 命令获取 Chialisp 谜语的哈希值。

```bash
opc -H '(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))'
```

这个命令的响应将是一个谜语哈希**在第一行**和你的谜语的序列化版本在第二行。

**示例响应：**

```
4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71
ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080
```

正如您从该响应的第一行中看到的，我们自定义谜语的谜语哈希是`0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71`

### 使用 Chialisp

您还可以使用 Chialisp 本身来获取 Chialisp 谜语的哈希值。获得谜语哈希的谜语如下：

```chialisp
(mod (puzzle)
    (defconstant TREE 1)

    (defun sha256tree1 (TREE)
       (if (l TREE)
           (sha256 2 (sha256tree1 (f TREE)) (sha256tree1 (r TREE)))
           (sha256 1 TREE)
       )
    )

    (sha256tree1 puzzle)
)
```

然后，这个谜语的谜底需要在第一个位置包含我们想要散列的已编译谜语（用括号将编译后的谜题包裹起来，并将其作为谜底提供）。

**密码锁定硬币的示例谜底：**
`((a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1)))`

此示例谜语的结果哈希再次为 `0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71`。

<details>
<summary>原文参考</summary>

- ## Get puzzle hash from a puzzle

- ### Using [clvm_tools](https://github.com/Chia-Network/clvm_tools)

You can get the hash of your Chialisp puzzle with the `opc -H <compiled_puzzle>` command included in the official clvm_tools repository.
```bash
opc -H '(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))'
```
The response of this command will be a puzzle hash **on the first line** and a serialized version of your puzzle on the second line.

**Example response:**
```
4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71
ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080
```

As you can see from the first line of this response, puzzle hash for our custom puzzle is `0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71`

- ### Using Chialisp

You can also use Chialisp itself to get the hash of the Chialisp puzzle. The puzzle to get a puzzle hash is following:

```chialisp
(mod (puzzle)
    (defconstant TREE 1)

    (defun sha256tree1 (TREE)
       (if (l TREE)
           (sha256 2 (sha256tree1 (f TREE)) (sha256tree1 (r TREE)))
           (sha256 1 TREE)
       )
    )

    (sha256tree1 puzzle)
)
```

A solution to this puzzle then needs to contain the compiled puzzle we want to hash in the first position (wrap the compiled puzzle with parentheses and provide it as a solution).

**Example solution for the password-locked coin:**
`((a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1)))`

The resulting hash for this example puzzle is again `0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71`.

</details>

## 将谜语哈希转换为接收地址

您可以将谜语哈希转换为接收地址，反之亦然。 地址只是一个编码的谜语哈希。 由于谜语哈希匹配特定谜语，这也意味着接收地址匹配特定谜语。

您可以使用 [Chia explorer 在线工具](https://www.chiaexplorer.com/tools/address-puzzlehash-converter) 进行谜语哈希与接收地址的转换。谜语哈希被编码为带有 xch 前缀的 bech32m 格式以形成接收地址。 谜语哈希 `0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71`的接收地址是 `xch1fppus6dm5hm94g0gqmxnwtdw2e5v5wmfvsxsvl5xsd72j6ejfecsdnkf2e`。

<details>
<summary>原文参考</summary>

- ## Convert puzzle hash to a receive address

You can convert a puzzle hash to a receive address and vice-versa. An address is just an encoded puzzle hash. And since a puzzle hash matches specific puzzle, it also means that a receive address matches a specific puzzle.

You can use [Chia explorer's online tool](https://www.chiaexplorer.com/tools/address-puzzlehash-converter) for converting between puzzle hash to receive address. The puzzle hash is encoded to bech32m format with xch prefix to form a receive address. The receive address for the puzzle hash `0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71` is `xch1fppus6dm5hm94g0gqmxnwtdw2e5v5wmfvsxsvl5xsd72j6ejfecsdnkf2e`.

</details>

## 将 Chia 发送到接收地址

使用 Chia GUI 或 CLI 发送交易，就像您通常使用所需的金额一样。作为接收地址，设置上一步中的地址（密码锁定硬币示例：`xch1fppus6dm5hm94g0gqmxnwtdw2e5v5wmfvsxsvl5xsd72j6ejfecsdnkf2e`）。这将用一个新的谜语锁定你的硬币。

---

如果您还有其他问题，请加入 [Chia Network 的公共 Keybase 团队](https://keybase.io/team/chia_network.public) 并在 *#chialisp* 频道中提问。

<details>
<summary>原文参考</summary>

- ## Send Chia to the receive address

Use the Chia GUI or CLI to send a transaction as you would typically do with the amount you want. As a receive address, set the address from the previous step (password-locked coin example: `xch1fppus6dm5hm94g0gqmxnwtdw2e5v5wmfvsxsvl5xsd72j6ejfecsdnkf2e`). That will lock your coin with a new puzzle.

---

If you have further questions, join [Chia Network's public Keybase team](https://keybase.io/team/chia_network.public) and ask in the *#chialisp* channel.

</details>
