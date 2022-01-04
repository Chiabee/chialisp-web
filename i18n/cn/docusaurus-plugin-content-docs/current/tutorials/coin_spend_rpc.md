---
id: coin_spend_rpc
title: 如何使用 RPC 调用花费硬币
sidebar_label: 使用 RPC 花费一个硬币
---

> How to spend a coin using an RPC call

本教程教您如何使用 RPC 调用在任何谜题上花费硬币。我们将使用[如何使用自定义拼图锁定硬币](custom_puzzle_lock) 中的密码锁定硬币谜语作为示例。

## 获取您的硬币信息（金额、谜语哈希和父信息）

花费硬币的 RPC 调用要求您指定要花费的硬币。对于唯一标识，您需要硬币的金额、谜语哈希和父信息。这三个信息也足以计算硬币的 ID。

### 使用 Chia explorer（通过谜语哈希/接收地址）

如果您知道您正在寻找的硬币的谜语哈希或接收地址，您可以[使用 Chia explorer 搜索](https://www.chiaexplorer.com/blockchain/search)。 Chia explorer无法使用拼图hash搜索，所以如果你有谜语哈希，首先需要使用 [Chia explorer 的工具](https://www.chiaexplorer.com/tools/address-puzzlehash-converter)转换为接收地址。
请记住，接收地址只是编码的谜语哈希，并且仍会引用您正在寻找的谜语。

当您搜索接收地址时，您会看到所有被相应谜语锁定的硬币。选择您要花费的那一项。这将为您提供硬币的金额、谜语哈希和父信息。

<details>
<summary>原文参考</summary>

This tutorial teaches you how to spend a coin with any puzzle using RPC calls. We will be using the password-locked coin puzzle from [How to lock coin with a custom puzzle](custom_puzzle_lock) as an example.

- ## Get your coin's info (amount, puzzle hash & parent info)

RPC call for spending a coin requires you to specify which coin you are spending. For unique identification, you need the coin's amount, puzzle hash, and parent info. Those three pieces of information are also enough to calculate the coin's ID.

- ### Using Chia explorer (by puzzle hash/receive address)

If you know the puzzle hash or receive address of the coin you are looking for, you can [search for it using Chia explorer](https://www.chiaexplorer.com/blockchain/search). Chia explorer cannot search using puzzle hash, so if you have a puzzle hash, you first need to convert it to receive address using [Chia explorer's tool](https://www.chiaexplorer.com/tools/address-puzzlehash-converter).
Remember that receive addresses are just encoded puzzle hashes and will still refer to the puzzle you are looking for.

When you search for a receive address, you'll see all coins locked by the corresponding puzzle. Select the one you want to spend. That will get you the coin's amount, puzzle hash, and parent info.

</details>

## 获取序列化谜语和谜底

花费硬币的下一件事是硬币的谜语和谜底。谜语和谜底以序列化格式提供，因此我们需要为每个谜语和谜底提供。谜语必须编译为低级 Chialisp 并正常序列化。

要序列化谜底，您需要稍微修改谜底方式，使其成为有效的 Chialisp 程序。为此，您需要引用您的谜底。例如，在谜底为 `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)` 的情况下，你需要将其更改为 `(q . (hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2))` 这使得可以编译它有效 Chialisp 程序。

**密码锁定硬币示例：**

**谜语：** `(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))`

**序列化的谜语：** `0xff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080`

**谜底：** `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)` (as valid Chialisp program `(q . (hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2))`)

**序列化的谜底：** `0xff8568656c6c6fffa05f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675ff0280`

_警告：您必须更改此解决方案并将目标谜语哈希替换为您自己的哈希才能取回您的硬币_

### 使用 [clvm_tools](https://github.com/Chia-Network/clvm_tools) 进行序列化

可以使用我们在[如何使用自定义谜语锁定硬币](custom_puzzle_lock#get-puzzle-hash-from-a-puzzle) (`opc -H <compiled_puzzle>`) 中用于获取谜语哈希的相同命令获取我们的谜语和谜底的序列化版本。序列化版本将包含在响应中**在第二行**。

```bash
opc -H '(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))'
```

**示例响应：**

```
4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71
ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080
```

### 使用 [Quexington 的 Chialisp 开发工具](https://github.com/Quexington/chialisp_dev_utility) 进行序列化 

按照存储库的 README 设置新项目并序列化谜语。

简而言之：将您编译的谜语/谜底粘贴到您的工作文件中并调用“chialisp build”。这将生成带有您的谜语/谜底的序列化版本的 `.hex` 文件（取决于您的工作文件）。

### 使用 [Chialisp Web 工具](https://clisp.surrealdev.com/) 进行序列化

将您的谜语粘贴到文本区域，然后单击 **Compile**。序列化的结果将显示在 **Serialized** 部分。

<details>
<summary>原文参考</summary>

- ## Get serialized puzzle and solution

The next thing you need to know to spend the coin is the coin's puzzle and solution. Puzzles and solutions are provided in a serialized format, so we need to get that for each. The puzzle has to be compiled to low-level Chialisp and is serialized as normal.

To serialize the solution, you need to slightly modify the solution format to make it valid Chialisp program. For that, you need to quote your solution. For example, in case of the solution `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)` you need to change it to `(q . (hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2))` which makes it valid Chialisp program that can be compiled.

**Example for the password-locked coin:**

**Puzzle:** `(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))`

**Serialized puzzle:** `0xff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080`

**Solution:** `(hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2)` (as valid Chialisp program `(q . (hello 0x5f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675 2))`)

**Serialized solution:** `0xff8568656c6c6fffa05f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675ff0280`

_WARNING: You have to change this solution and replace the target puzzle hash with your own to get your coins back_

- ### Serialization using [clvm_tools](https://github.com/Chia-Network/clvm_tools)

The same command that we used for getting the puzzle hash in [How to lock coin with a custom puzzle](custom_puzzle_lock#get-puzzle-hash-from-a-puzzle) (`opc -H <compiled_puzzle>`) can be used for getting the serialized version of our puzzle and solution as well. The serialized version will be included in the response **on the second line**.

```bash
opc -H '(a (q 2 (i (= (sha256 5) (q . 0x2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824)) (q 4 (c 2 (c 11 (c 23 ()))) ()) (q 8)) 1) (c (q . 51) 1))'
```

**Example response:**
```
4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71
ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080
```

- ### Serialization using [Quexington's Chialisp Dev Utility](https://github.com/Quexington/chialisp_dev_utility)

Follow repository's README to set up a new project and serialize puzzle.

In short: paste your compiled puzzle/solution to your work file and call `chialisp build`. That will generate `.hex` files with a serialized version of your puzzle/solution (depending on your work file).

- ### Serialization using [Chialisp web tool](https://clisp.surrealdev.com/)

Paste your puzzle in the text area and click **Compile**. The serialized result will be displayed in the **Serialized** section.

</details>

## 用 RPC 调用花一个硬币

要花费您的硬币，您只需要使用特定于您的花费的值调用 RPC（广播交易示例）。

```bash
curl --insecure --cert ~/.chia/mainnet/config/ssl/full_node/private_full_node.crt --key ~/.chia/mainnet/config/ssl/full_node/private_full_node.key -d '{        "spend_bundle": {
            "aggregated_signature": "0xc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
            "coin_solutions": [
                {
                    "coin": {
                        "amount": 1,
                        "parent_coin_info": "0xccd5bb71183532bff220ba46c268991a00000000000000000000000000004082",
                        "puzzle_hash": "0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71"
                    },
                    "puzzle_reveal": "ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080 ",
                    "solution": "ff8568656c6c6fffa05f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675ff0280"
                }
            ]
        }}' -H "Content-Type: application/json" -X POST https://localhost:8555/push_tx
```

`spend_bundle` 对象包含一个 `aggregated_signature`，我们稍后可以在谜语中宣称，以及 `coin_solutions`：我们花费的所有硬币的对象列表。 如果您的谜语不需要 `aggregated_signature`，请使用 0xc 后跟 191 个零（如上例所示）。 但是，值得注意的是，不使用签名的谜语通常是不安全的，应仅用于测试目的。

`coin_solution` 包含有关它正在花费的 `coin` 的信息（`amount`、`parent_coin_info` 和 `puzzle_hash`）。 它还包括一个序列化的谜语作为 `puzzle_reveal` 和序列化的“解决方案”。

如果您正确填写所有信息并发送此请求，您的硬币将根据其提供的谜底案花费，并且应从 RPC 返回响应 `{"status": "SUCCESS", "success": true}` 称呼。

如果您的谜语需要聚合签名，请继续关注更多教程。

---

如果您还有其他问题，请加入 [Chia Network 的公共 Keybase 团队](https://keybase.io/team/chia_network.public) 并在 *#chialisp* 频道中提问。

<details>
<summary>原文参考</summary>

- ## Spend a coin with RPC call

To spend your coin, you only need to call RPC (broadcast transaction example) with values specific to your spend.

```bash
curl --insecure --cert ~/.chia/mainnet/config/ssl/full_node/private_full_node.crt --key ~/.chia/mainnet/config/ssl/full_node/private_full_node.key -d '{        "spend_bundle": {
            "aggregated_signature": "0xc00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
            "coin_solutions": [
                {
                    "coin": {
                        "amount": 1,
                        "parent_coin_info": "0xccd5bb71183532bff220ba46c268991a00000000000000000000000000004082",
                        "puzzle_hash": "0x4843c869bba5f65aa1e806cd372dae5668ca3b69640d067e86837ca96b324e71"
                    },
                    "puzzle_reveal": "ff02ffff01ff02ffff03ffff09ffff0bff0580ffff01a02cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b982480ffff01ff04ffff04ff02ffff04ff0bffff04ff17ff80808080ff8080ffff01ff088080ff0180ffff04ffff0133ff018080 ",
                    "solution": "ff8568656c6c6fffa05f5767744f91c1c326d927a63d9b34fa7035c10e3eb838c44e3afe127c1b7675ff0280"
                }
            ]
        }}' -H "Content-Type: application/json" -X POST https://localhost:8555/push_tx
```

The `spend_bundle` object contains an `aggregated_signature`, which we can later assert in the puzzle, and `coin_solutions`: a list of objects for all of the coins we are spending. If `aggregated_signature` is not necessary for your puzzle, use 0xc followed by 191 zeros (as in the example above). However, it's worth noting that a puzzle that doesn't use a signature is usually unsafe and should be used only for testing purposes.

The `coin_solution` contains information about the `coin` it is spending (`amount`, `parent_coin_info`, and  `puzzle_hash`). It also includes a serialized puzzle as a `puzzle_reveal` and serialized `solution`.

If you fill in all your information correctly and send this request, your coin will be spent according to its provided solution, and the response `{"status": "SUCCESS", "success": true}` should be returned from the RPC call.

If your puzzle requires an aggregated signature, stay tuned for more tutorials.

---

If you have further questions, join [Chia Network's public Keybase team](https://keybase.io/team/chia_network.public) and ask in the *#chialisp* channel.

</details>
