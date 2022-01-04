---
id: structure_of_a_chia_application
title: Chia 应用程序的结构
sidebar_label: Chia 应用程序的结构
---

> Structure of a Chia Application

# 使用 chia 的应用程序的基本结构

> Basic structure of an app using chia

```

        CHIA                        Your Code
                 |   +--------------------+       +----------+
    Node RPC <---|-> | Specialized Wallet |<----->| Database |
       ^         |   +--------------------+       +----------+
       |         |       ^         ^
       |         |       |         |
       |         |       |         v
       v         |       |    +------------------+
    Wallet RPC --|-------+    | State management |
                 |            +------------------+
                 |                    ^
                                      |
                                      v
                                     User
```

### 开发 chia 应用程序的顾虑

- 该应用程序可能至少目前需要连接到 chia RPC。

- 需要一个“钱包”类型的系统来跟踪区块链流量，以便应用程序的当前状态概念可以从区块链中表示的相同信息中恢复，并通过节点 API 保持呈现给最终用户的状态一致以及一组精心设计的 chialisp 代码参数。通过全节点的 get_additions_and_removals 和通过 get_puzzle_and_solution 的解决方案可以获得区块中触及的硬币。通过检查每个块，可以找到特殊格式的硬币解决方案并跟踪它们。

- 应用程序至少需要允许用户以易于理解的方式执行操作，这意味着结合使用钱包和节点的 RPC API 来

   1. 通过 get_transactions、get_coin_record_by_name 和 get_private_key 以及 master_sk_to_wallet_sk 函数确定在与区块链上的硬币交互时要使用的公钥和私钥。

   2. 通过 push_tx 将交易发送到区块链。

- 应用程序可能无法保证在其部署的代码的整个持续时间内保持运行，因此从区块链获取的任何状态都应存储在本地缓存数据库中并从中检索。

- 如果不止一个方在相关硬币上进行合作，则需要将识别相关硬币的标识符发送到带外，或者需要将接收者可识别的内容嵌入硬币解决方案中。

<details>
<summary>原文参考</summary>

- ### Concerns for developing chia apps

- The app likely needs a connection to the chia RPC at least for now.

- A "wallet" type system is needed to track blockchain traffic so that the app's
  concept of the current state can be recovered from the same information that's
  represented in the blockchain and in order to keep the state presented to the
  end user consistent via the node API and a well designed set of arguments to
  the chialisp code.  Coins touched in a block are available via the full node's
  get_additions_and_removals and the solutions via get_puzzle_and_solution.  By
  checking out each block, it'll be possible to find specially formatted
  coin solutions and track them.

- The app needs at a minimum to allow the user to take actions in a comprehensible
  way, which means using a combination of the wallet and node's RPC API to

    1. Establish which public and private keys to use when interacting with coins
       on the blockchain via get_transactions, get_coin_record_by_name and
       get_private_key and the master_sk_to_wallet_sk function.

    2. Send transactions to the blockchain via push_tx.

- It's likely that the app won't be able to guarantee that it remains running for
  the full duration of the purpose of the code it deploys, therefore any state
  picked up from the blockchain should be stored in and retrieved from a local
  cache database.

- If more than one party is cooperating over the coin in question, then either an
  identifier picking out the coin in question needs to be sent out of band or
  something identifiable by the recipient needs to be embedded in the coin
  solution.

</details>

