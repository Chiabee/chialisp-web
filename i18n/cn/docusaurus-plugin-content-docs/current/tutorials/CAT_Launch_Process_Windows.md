# CAT 创建教程 (Windows)

> CAT creation tutorial (Windows)

本教程用于在 Windows 上创建 Chia Asset Tokens (CATS)。我们还提供了一个[适用于 Linux 和 MacOS 用户教程](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Linux_MacOS "Chia Asset Token tutorial for Linux and MacOS users")的版本。

目录：

* [介绍](#介绍)
* [CAT 发放粒度](#cat-发放粒度)
* [设置 Chia 环境](#设置-chia-环境)
* [创建单一铸造的 CAT](#创建单一铸造的-cat)
* [创建多铸造的 CAT](#创建多铸造的-cat)
* [准备主网](#准备主网)
* [生成安全密钥对](#生成安全密钥对)
* [潜在的未来添加](#潜在的未来添加)

<details>
<summary>原文参考</summary>

This tutorial is for creating Chia Asset Tokens (CATS) on Windows. We also have made available a version of [this tutorial for Linux and MacOS users](https://www.chialisp.com/docs/tutorials/CAT_Launch_Process_Linux_MacOS "Chia Asset Token tutorial for Linux and MacOS users").

</details>

-----

## 介绍

欢迎来到 CAT 的世界！我们很高兴您能来到这里，我们迫不及待地想看到您提出的创意。

本教程将帮助您直接进入并开始发布您自己的 CAT。但是，在我们开始之前，您应该了解一些事项。

[CAT1 标准](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation")目前处于**草案**形式。这意味着如果我们向最终标准添加任何重大更改，您在此处所做的任何事情都可能在以后失效。 **继续需要您自担风险**。

由于这仍然是标准草案，并非所有边缘情况都经过彻底测试。为了最大程度地降低遇到意外结果的风险，我们建议您执行以下操作（本教程后面将详细讨论每一项）：

* 在同一台计算机上运行您的全节点和轻钱包。
* 请勿使用可执行安装程序安装本教程的轻钱包。相反，从源代码安装，因为本文档将向您展示如何操作。这是因为可执行安装程序将覆盖您的完整节点安装。
* 为您发出的每个 CAT 生成一个新的公钥/私钥对。这个密钥对应该用于发布一个特定的 CAT，**没有别的**。在发出 CAT 时，它也应该是您计算机上唯一的密钥对。
* 请勿尝试在活跃的农业机器上测试您的 CAT 创作。
* 在将您的 CAT 发布到主网之前，在测试网上彻底测试。

有关本教程的任何问题，请前往我们 [Keybase](https://keybase.io/team/chia_network.public "Chia's Keybase forum") 论坛上的 #chialisp 频道，那里有很多友好的人可以帮你。

<details>
<summary>原文参考</summary>

- ## Introduction

Welcome to the world of CATs! We're excited to have you here, and we can't wait to see the creative ideas you come up with.

This tutorial will help you jump right in and get started with issuing your own CATs. However, there are a few things you should know before we begin.

The [CAT1 standard](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation") is currently in **draft** form. This means that anything you do here could potentially be invalidated later if we add any breaking changes to the final standard. **Proceed at your own risk**.

As this still a draft standard, not all edge cases have been thoroughly tested. To minimize your risk of running into unexpected results, we recommend that you do following (each of these will be discussed in more detail later in the tutorial):

* Run your full node and light wallet on the same computer.
* Do not use the executable installer to install the light wallet for this tutorial. Instead, install from source, as this document will show you how to do. The reason for this is because the executable installer will overwrite your full node installation.
* Generate a new public/private key pair for each CAT you issue. This key pair should be used for issuing one specific CAT **and nothing else**. It should also be the only key pair on your computer while issuing the CAT.
* Do not attempt to test your CAT's creation on an active farming machine.
* Test thoroughly on testnet before issuing your CAT to mainnet.

For any questions regarding this tutorial, head over to the #chialisp channel on our [Keybase](https://keybase.io/team/chia_network.public "Chia's Keybase forum") forum, where there are lots of friendly folks who can help you.

</details>

## CAT 发放粒度

CAT 面额，以及铸造和熔化背后的规则，可能需要一些时间来适应。在您签发 CAT 之前，请记住以下几点：

* 大多数 Chia 钱包选择在 XCH 中显示其价值。然而，这纯粹是一种装饰性的选择，因为 Chia 的区块链只知道 mojos。一个 XCH 等于一万亿（1,000,000,000,000）个魔力。
* 同样，默认决定将 1 CAT 映射到 1000 XCH mojo。默认情况下，该比率对于所有 CAT 都是相同的。
* 可以将特定 CAT 的 CAT:mojo 比率设置为 1:1000 以外的值，但这样做可能会对令牌之间的互操作性产生负面影响。我们建议您使用默认设置，除非您有充分的理由不这样做。
* 因此，单个代币的默认熔炼值为 1000 mojo。无论代币的面值或流通量如何，这都是正确的。
* 代币的面值与其熔值不一定相关，更不用说匹配了。

使用一个 XCH，您可以铸造 10 亿个 CAT。这些代币的面值可以是零，也可以是多个 XCH，或者介于两者之间。这个价值是由市场决定的——无论有人愿意为它付出什么，它都是值得的。除了它们的 1000-mojo 熔化值外，代币的价值与基础 XCH 无关。

这些概念在我们的 [CAT1 标准](https://chialisp.com/docs/puzzles/cats#cat-denominations-value-and-retirement-rules "CAT1 standard documentation")中有更详细的讨论。

<details>
<summary>原文参考</summary>

- ## CAT issuance granularity

CAT denominations, as well as the rules behind minting and melting, can take some getting used to. Here are a few things to keep in mind before you issue your CATs:

* Most Chia wallets choose to display their value in XCH. However, this is a purely cosmetic choice because Chia's blockchain only knows about mojos. One XCH is equal to one trillion (1,000,000,000,000) mojos.
* In a similar vein, a default decision was made to map 1 CAT to 1000 XCH mojos. By default, this ratio will be the same for all CATs.
* It is possible to set the CAT:mojo ratio to something other than 1:1000 for a specific CAT, but doing so could negatively affect interoperability between tokens. We recommend that you use the default setting unless you have a good reason to do otherwise.
* Therefore, the default melt value of a single token is 1000 mojos. This remains true regardless of the token's face value or its circulating supply.
* A token's face value and its melt value are not necessarily correlated, let alone matched.

With one XCH, you can mint 1 billion CATs. The face value of these tokens could be zero, or multiple XCH, or anywhere in between. This value is decided by the market -- it's worth whatever someone is willing to pay for it. The value of the tokens has nothing to do with the underlying XCH, other than their 1000-mojo melt value.

These concepts are discussed in greater detail in our [CAT1 standard](https://chialisp.com/docs/puzzles/cats#cat-denominations-value-and-retirement-rules "CAT1 standard documentation").

</details>

## 设置 Chia 环境

首先，关于 Chia 源代码当前结构的一些说明，因为它与 CAT 相关：

> 目前，轻钱包是唯一能够发行新 CAT 的钱包。它目前位于一个名为 `protocol_and_cats_rebased` 的 GitHub 分支中。该分支最终将与 `main` 分支合并，但虽然 CAT1 标准仍处于草案形式，但两个分支将保持分开。
>
> 发行 CAT 有两个阶段：在测试网上测试您的发行和在主网上实际发行。在这两个阶段，你都需要一个同步的全节点和一个同步的轻钱包。
>
>为了测试您的发行，您需要使用 testnet10。该测试网仅位于 `protocol_and_cats_rebased` 分支上。因此，您必须将此分支用于您的完整节点和轻钱包。
>
>为了在主网上发行，你的轻钱包必须来自 `protocol_and_cats_rebased`，但你的完整节点可以基于 `main` 或 `protocol_and_cats_rebased`。

为了开始本教程，我们假设您当前正在测试您的发行，因此将在 `protocol_and_cats_rebased` 分支上运行。在本教程的后面，我们将向您展示如何切换到主网。

好吧，是时候开始了！

1. 从GitHub克隆 `protocol_and_cats_rebased` 分支并安装轻钱包：

    a. 安装运行正常 Chia 构建所需的先决条件，例如 Git、Python 和 Rust。请注意，您的 Python 版本必须介于 3.7 和 3.9 之间。你可以通过运行`python version`来获取版本。

    b. 以管理员身份打开一个新的 PowerShell 窗口，创建一个名为 `protocol_and_cats_rebased` 的新文件夹，然后 cd 到它。

    > 如果您错过了上述警告，请不要运行可执行安装程序来安装轻钱包。如果您安装了一个，它将覆盖您的完整节点。这将在未来的版本中修复。现在，运行下面列出的 `git` 命令以避免覆盖你的完整节点。
  
    c. 运行 `git clone https://github.com/Chia-Network/chia-blockchain.git -b protocol_and_cats_rebased --recurse-submodules` 来克隆 Chia 的 CAT 分支。
 
    d. 运行 `cd chia-blockchain`。
    
    e. 运行 `.\Install.ps1` 来安装轻钱包。
    
    f. 运行 `.\venv\Scripts\Activate.ps1` 来激活一个虚拟环境。
    
    g. 运行 `chia init` 来初始化你的环境。

    h. 如果您收到此消息：“警告：未受保护的 SSL 文件！”然后运行 `chia init --fix-ssl-permissions`。

    i. 运行 `chia configure -t true` 切换到 testnet10。
    
    j. 运行 `.\Install-gui.ps1` 来安装轻钱包 GUI。

    >如果此命令失败，您可能会在运行 `.\install-gui.sh` 时获得更好的运气。

2. 运行并同步轻钱包GUI：

    a. 在做任何其他事情之前，最好将 log_level 设置为 INFO。为此，请运行 `chia configure -log-level INFO`。
    
    b. 运行 `cd chia-blockchain-gui`。
    
    c. 运行 `npm run electron` 将轻钱包 GUI 作为守护进程运行。

    d. 如果您已经有一个“带有公共指纹的私钥”，请在 GUI 加载时选择它。否则，选择“创建新的私钥”。

    e. “状态：正在同步”应该出现在 GUI 的右上角。几分钟后，这应该会更改为“状态：已同步”。这个过程不需要很长时间，因为轻钱包只请求和下载该特定钱包所需的块。

    f. 如果您的总余额为 0，您可以从 [我们的水龙头](https://testnet10-faucet.chia.net "testnet10 TXCH faucet") 获取一些 testnet10 TXCH。

3. 同步你的 testnet10 full_node：

    因为您在测试网上运行，所以您可以下载一个数据库来加速全节点的同步。
    
    > **警告：不要在主网上尝试这个。**

    a. 打开一个新的浏览器窗口并转到[我们的下载站点](https://download.chia.net/?prefix=testnet10/ "Testnet10 database download")。
    
    b. 点击文件 “blockchain_v1_testnet10.sqlite” 下载。根据您的连接速度，这可能需要几分钟时间。
    
    c. 将 .sqlite 文件移动到 db 文件夹，该文件夹位于 `C:\Users\<User>\.chia\standalone_wallet\wallet\db\`。
    
    d. 切换到安装轻钱包的 “chia-blockchain” 目录。

    e. 如果 “(venv)” 没有出现在你的命令行左侧，运行 `.\venv\Scripts\Activate.ps1` 来激活你的虚拟环境。
    
    f. 运行 `chia start node`。您应该会收到一条消息，说明 chia_full_node 已启动。如果您将 .sqlite 文件复制到您的 db 文件夹，您的节点应该会在几分钟内同步。您可以在 `C:\Users\<User>\.chia\standalone_wallet\log\debug.log` 中监控同步进度。

    g. 当您的完整节点正在同步时，您可以继续下一步。


4. 设置 CAT 管理工具，它将帮助您签发 CAT：

    a. 运行 `git clone https://github.com/Chia-Network/CAT-admin-tool.git -b main --recurse-submodules` 来克隆 CAT 管理工具。

    b. 运行 `cd CAT-admin-tool-main`。
    
    c. 运行 `python -m venv venv` 来创建一个虚拟环境。

    d. 运行 `.\venv\Scripts\Activate.ps1` 来激活虚拟环境。

    e. 运行 `pip install .`。这将需要几分钟。
    
    f. 运行 `pip install chia-dev-tools --no-deps`。
    
    g. 运行 `pip install pytest`。您可以放心地忽略有关缺少需求的错误。

5. 你的环境应该已经设置好了，但让我们确保：

    a. 运行 `cats --help`。你应该得到一个使用说明。
    
    b. 运行 `cdv --help`。你应该得到另一个使用声明。
    
    c. 运行 `chia show -s`。您应该会收到以下消息：“当前区块链状态：完整节点已同步”，以及最新区块高度的列表。

    d. 验证“状态：已同步”是否显示在独立 GUI 的右上角。

    e. 确保你的钱包里有一些 TXCH。

您的环境现已设置完毕，您可以开始发布 CAT。


<details>
<summary>原文参考</summary>

- ## Setting up your Chia environment

First, a few notes on the current structure of Chia's source code as it pertains to CATs:

>For now, the light wallet is the only wallet capable of issuing new CATs. It is currently located in a GitHub branch called `protocol_and_cats_rebased`. This branch will eventually be merged with the `main` branch, but while the CAT1 standard is still in draft form, the two branches will be kept separate.
>
>There are two phases of issuing a CAT: testing your issuance on testnet and actually issuing on mainnet. In both of these phases, you'll need a synced full node and a synced light wallet.
>
>For testing your issuance, you'll need to use testnet10. This testnet is only located on the `protocol_and_cats_rebased` branch. Therefore, you must use this branch for both your full node and the light wallet.
>
>For issuing on mainnet, your light wallet must come from `protocol_and_cats_rebased`, but your full node can be based off of either `main` or `protocol_and_cats_rebased`.

To start this tutorial, we'll assume you are currently testing your issuance, and therefore will be running on the `protocol_and_cats_rebased` branch. Later in the tutorial, we'll show you how to make the switch to mainnet.

All right, time to get started!

1. Clone the `protocol_and_cats_rebased` branch from GitHub and install the light wallet:

    a. Install the necessary prerequisites to run a normal Chia build, such as Git, Python, and Rust. Note that your Python version must be between 3.7 and 3.9. You can obtain the version by running `python version`.

    a. Open a new PowerShell window as Administrator, create a new folder called `protocol_and_cats_rebased` and cd to it.

    > In case you missed the above warning, please do not run the executable installer to install the light wallet. It will overwrite your full node if you have one installed. This will be fixed in a future release. For now, run the `git` command listed below to avoid overwriting your full node.
  
    b. Run `git clone https://github.com/Chia-Network/chia-blockchain.git -b protocol_and_cats_rebased --recurse-submodules` to clone Chia's CAT branch.
 
    c. Run `cd chia-blockchain`.
    
    d. Run `.\Install.ps1` to install the light wallet.
    
    e. Run `.\venv\Scripts\Activate.ps1` to activate a virtual environment.
    
    f. Run `chia init` to initialize your environment.

    g. If you receive this message: "WARNING: UNPROTECTED SSL FILE!" then run `chia init --fix-ssl-permissions`.

    h. Run `chia configure -t true` to switch to testnet10.
    
    i. Run `.\Install-gui.ps1` to install the light wallet GUI.

    >If this command fails, you may have better luck running `.\install-gui.sh`.

2. Run and sync the light wallet GUI:

    a. Before doing anything else, it’s a good idea to set your log_level to INFO. To do this, run `chia configure -log-level INFO`.
    
    b. Run `cd chia-blockchain-gui`.
    
    c. Run `npm run electron` to run the light wallet GUI as a daemon.

    d. If you already have a "Private key with public fingerprint", select it when the GUI loads. Otherwise, select "CREATE A NEW PRIVATE KEY".

    e. "Status: Syncing" should appear in the upper right corner of the GUI. Within a few minutes, this should change to "Status: Synced". This process doesn’t take long because the light wallet only requests and downloads the blocks that are required for that specific wallet.

    f. If your Total Balance is 0, you can get some testnet10 TXCH from [our faucet](https://testnet10-faucet.chia.net "testnet10 TXCH faucet").

3. Sync your testnet10 full_node:

    Because your are running on the testnet, you can download a database to speed up the syncing of your full node.
    
    > **WARNING: Do not attempt this on mainnet.** 

    a. Open a new browser window and go to [our download site](https://download.chia.net/?prefix=testnet10/ "Testnet10 database download").
    
    b. Click the file "blockchain_v1_testnet10.sqlite" to download it. Depending on your connection speed, this could take several minutes.
    
    c. Move the .sqlite file to the db folder, which is located in `C:\Users\<User>\.chia\standalone_wallet\wallet\db\`.
    
    d. Change to the "chia-blockchain" directory where you installed the light wallet.

    e. If "(venv)" doesn’t appear on the left side of your command line, run `.\venv\Scripts\Activate.ps1` to activate your virtual environment.
    
    f. Run `chia start node`. You should receive a message that the chia_full_node has been started. If you copied the .sqlite file to your db folder, your node should be synced within a few minutes. You can monitor the syncing progress in `C:\Users\<User>\.chia\standalone_wallet\log\debug.log`.

    g. While your full node is syncing, you may proceed to the next step.


4. Set up the CAT admin tool, which will help you to issue your CATs:

    a. Run `git clone https://github.com/Chia-Network/CAT-admin-tool.git -b main --recurse-submodules` to clone the CAT admin tool.

    b. Run `cd CAT-admin-tool-main`.
    
    c. Run `python -m venv venv` to create a virtual environment.

    d. Run `.\venv\Scripts\Activate.ps1` to activate the virtual environment.

    e. Run `pip install .`. This will take a few minutes.
    
    f. Run `pip install chia-dev-tools --no-deps`.
    
    g. Run `pip install pytest`. You can safely ignore the errors about missing requirements.

5. Your environment should be all set, but let's make sure:

    a. Run `cats --help`. You should get a usage statement.
    
    b. Run `cdv --help`. You should get another usage statement.
    
    c. Run `chia show -s`. You should get this message: "Current Blockchain Status: Full Node Synced", along with a listing of the latest block heights.

    d. Verify that "Status: Synced" is showing in the upper right side of the standalone GUI.

    e. Make sure you have some TXCH in your wallet.

Your environment is now set up and you are ready to start issuing CATs.

</details>

## 创建单一铸造的 CAT

如果您是视觉学习者，请参阅我们的 [创建单一铸造 CAT 视频教程](https://chialisp.com/docs/tutorials/single_issuance_CAT "Single-mint CAT video tutorial")。

> 注意：本节将讨论代币资产发行限制器 (TAIL)，以及 CAT 的一些技术细节。要复习 CAT 和 TAIL，请查看我们的 [CAT1 标准](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation")。

首先，您将创建一个单一的 CAT。这是发出 CAT 的默认方式。这也是最简单的。它包含一个只能用于特定 XCH 硬币的 TAIL。在 Chia 中，代币只能使用一次，因此在这种情况下，CAT 只能铸造一次代币。

带有单一铸币 TAIL 的 CAT 对任何想要创建具有保证固定供应量的代币的人都非常有用。

你可以在[这里](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/genesis-by-coin-id-with-0.clvm "Single-mint TAIL")。

1. 找一个要铸造的硬币，然后创建并推送一个新的支出包：

    a. 如果您不在那里，请更改为 “CAT-admin-tool-main” 文件夹。

    b. 弄清楚您要使用多少个 XCH mojo 来签发 CAT。默认情况下，每个 CAT 令牌将包含 1000 个 mojo，因此您应该将要铸造的令牌数量乘以 1000。例如，如果您想铸造 100 万个令牌，则需要 10 亿个 XCH mojo（的 1/1000 XCH）。

    c. 在独立 GUI 中记下您的接收地址。下一步将需要它。

    d. 运行 `cats --tail .\reference_tails\genesis_by_coin_id.clsp.hex --send-to <your receive address> --amount <XCH mojos> --as-bytes --select-coin`

    --select-coin 标志将从您的钱包中选择一个硬币来铸造您的代币。输出的最后一行将是“Name: &lt;Coin ID&gt;”。您将使用“&lt;Coin ID&gt;”在下一步中。

    e. 再次运行相同的命令，这次删除 --select-coin 标志并添加一个新标志，“--curry 0x&lt;Coin ID&gt;”。为&lt;Coin ID&gt;做序是非常重要的。此处使用 0x，以使 CLVM 将值解释为字节而不是字符串。这是要运行的完整命令：

    `cats --tail .\reference_tails\genesis_by_coin_id.clsp.hex --send-to <your receive address> --amount <XCH mojos> --as-bytes --curry 0x<Coin ID>`

    该命令将输出两个值，&lt;Asset ID&gt;和&lt;花费捆绑&gt;。 &lt;资产 ID&gt;将是此 CAT 的 ID，因此保存此值以备后用很重要。 &lt;花费捆绑包&gt;是一大堵文字墙。这是您将以字节格式推送到区块链的实际交易。

    f. 复制&lt;Spend Bundle&gt;的值并运行 `cdv rpc pushtx <Spend Bundle>`。您应该会收到消息 "status": "SUCCESS", "success": true。

    恭喜！您已经签发了第一张 CAT。不过，你仍然需要告诉你的钱包。

2. 为您的新 CAT 添加一个钱包 ID：

    一个。切换到您的轻钱包 GUI。在几分钟内，您的余额应该会减少您刚刚铸造的 mojo 数量。但是，它不会出现在您的交易中。该功能尚未实现。

    湾现在您可以为您的新 CAT 添加钱包 ID。在左上角，单击“+ ADD TOKEN”，然后单击“+自定义”。在名称字段中输入您的 CAT 的名称（可以是任何名称）。对于令牌和资产发行限制字段，粘贴&lt;资产 ID&gt;你从几步前保存。单击添加。

    C。您现在将被带到您的新 CAT 钱包。余额应显示您选择使用的 XCH mojo 数量除以 1000。这是因为 CAT mojo 默认情况下是 CAT 的千分之一。
    
    > 注意：目前您可能会遇到两个外观错误，将在接下来的两个步骤中介绍。
    
    d.如果您看到总余额为 0，则需要刷新您的钱包。运行 `chia start wallet-only -r`。您现在应该看到正确的余额。这将在未来的版本中修复。

    e.您的猫的名字可能不会自动显示在您的钱包中。在这种情况下，单击“状态：已同步”旁边的三个点，然后单击“重命名钱包”。您可以将其重命名为其正确的名称。这将在未来的版本中自动完成。

您现在可以在 GUI 中访问您的 CAT。您可以像使用常规 XCH 一样发送和接收新代币。

<details>
<summary>原文参考</summary>

- ## Creating a single-mint CAT

If you're a visual learner, please see our [video tutorial for creating a single-mint CAT](https://chialisp.com/docs/tutorials/single_issuance_CAT "Single-mint CAT video tutorial").

> NOTE: This section will discuss Token Asset Issuance Limiters (TAILs), as well some technical details of CATs. For a refresher on CATs and TAILs, check out our [CAT1 standard](https://chialisp.com/docs/puzzles/cats "CAT1 standard documentation").

To get started, you will create a single-mint CAT. This is the default way to issue a CAT. It's also the simplest. It contains a TAIL that only can be used on a specific XCH coin. In Chia the coins can only be spent once, so in this case, the CAT can only mint tokens once.

A CAT with a single-mint TAIL will be useful for anyone who wants to create a token with a guaranteed fixed supply.

You can find the TAIL we'll use for this example [here](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/genesis-by-coin-id-with-0.clvm "Single-mint TAIL").

1. Find a coin to mint, and create and push a new spendbundle:

    a. Change to the "CAT-admin-tool-main" folder if you're not already there.

    b. Figure out how many XCH mojos you want to use to issue your CAT. By default each CAT token will contain 1000 mojos, so you should multiply the number of tokens you want to mint by 1000. For example, if you want to mint 1 million tokens, you'll need 1 billion XCH mojos (1/1000 of an XCH).

    c. Take note of your Receive Address in the standalone GUI. You'll need it for the next step.

    d. Run `cats --tail .\reference_tails\genesis_by_coin_id.clsp.hex --send-to <your receive address> --amount <XCH mojos> --as-bytes --select-coin`

    The --select-coin flag will choose a coin from your wallet for minting your tokens. The final line of the output will be "Name: &lt;Coin ID&gt;". You’ll use "&lt;Coin ID&gt;" in the next step.

    e. Run the same command again, this time removing the --select-coin flag and adding a new flag, "--curry 0x&lt;Coin ID&gt;". It’s very important to preface the &lt;Coin ID&gt; with 0x here, to make CLVM interpret the value as bytes and not a string. Here’s the full command to run:

    `cats --tail .\reference_tails\genesis_by_coin_id.clsp.hex --send-to <your receive address> --amount <XCH mojos> --as-bytes --curry 0x<Coin ID>`

    The command will output two values, &lt;Asset ID&gt; and &lt;Spend Bundle&gt;. &lt;Asset ID&gt; will be the ID of this CAT, so it’s important to save this value for later. &lt;Spend Bundle&gt; is a large wall of text. This is the actual transaction that you will push to the blockchain, in byte format.

    f. Copy the value of &lt;Spend Bundle&gt; and run `cdv rpc pushtx <Spend Bundle>`. You should receive the message "status": "SUCCESS", "success": true.

    Congratulations! You have issued your first CAT. You still need to tell your wallet about it, though.

2. Add a wallet ID for your new CAT:

    a. Switch to your light wallet GUI. Within a few minutes, your balance should decrease by the number of mojos you just minted. It won’t show up in your transactions, though. That feature has not yet been implemented.

    b. Now you can add a wallet ID for your new CAT. In the upper left corner, click "+ ADD TOKEN", then click "+ Custom". Enter the name of your CAT (it can be anything) in the Name field. For the Token and Asset Issuance Limitations field, paste the &lt;Asset ID&gt; you saved from a few steps ago. Click ADD.

    c. You will now be taken to your new CAT wallet. The balance should show the number of XCH mojos you chose to use, divided by 1000. This is because CAT mojos by default are one-thousandth of a CAT.
    
    > NOTE: There are currently two cosmetic bugs you might run into, covered in the next two steps.
    
    d. If you see a Total Balance of 0, you need to refresh your wallet. Run `chia start wallet-only -r`. You should now see the correct balance. This will be fixed in a future release.

    e. Your cat's name might not show up in your wallet automatically. In this case, click the three dots next to "Status: Synced" and click "Rename Wallet". You can rename it to its proper name. This will be done automatically in a future release.

You now have access to your CAT in the GUI. You can send and receive your new tokens just like you would with regular XCH.

</details>

## 创建多铸造的 CAT

如果您是视觉学习者，请参阅我们的 [创建多铸造 CAT 视频教程](https://chialisp.com/docs/tutorials/multiple_issuance_CAT "Multiple mint CAT video tutorial")。

接下来，我们将创建一个能够多次铸造代币的 CAT。这个 CAT 使用了一个委托的 TAIL，它比前一个灵活得多。只要您签署了您指定的谜语哈希，您就可以使用您想要的任何 TAIL 来铸造新令牌。这允许诸如回扣优惠和分布式铸造和停用代币等功能。

您可以在[此处](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/delegated_genesis_checker.clvm "Delegated TAIL")中找到我们将用于此示例的 TAIL .

我们将设置这个 CAT 来委托我们之前设置的相同的 TAIL。这意味着在您允许之前，没有其他人可以铸造新的代币。请记住，这只是委托 TAIL 的许多可能实现中的一种。

1. 找一个要铸造的硬币，然后创建并推送一个新的支出包：

    a. 如果您不在那里，请更改为 “CAT-admin-tool-main” 文件夹。

    b. 弄清楚您要使用多少个 XCH mojo 来签发 CAT。默认情况下，每个 CAT 令牌将包含 1000 个 mojo，因此您应该将要铸造的令牌数量乘以 1000。例如，如果您想铸造 100 万个令牌，则需要 10 亿个 XCH mojo（的 1/1000 XCH）。

    c. 在独立 GUI 中记下您的接收地址。

    d. 运行 `chia keys show`。记下您的&lt;指纹&gt;和&lt;主公钥&gt;。

    e. 运行 `cats --tail .\reference_tails\delegated_tail.clsp.hex --curry 0x<Master public key> --send-to <wallet address> -a <XCH mojos> --as-bytes --select-coin`

    --select-coin 标志将从您的钱包中选择一个硬币来发行 CAT。输出的最后一行将是“Name: &lt;Coin ID&gt;”。您将使用“&lt;Coin ID&gt;”在下一步中。

    现在你有了一个硬币，你可以创建一个完整的委托 TAIL。在我们的例子中，它委托的 TAIL 将是单一铸造品种。
    
    f. 运行 `cdv clsp curry .\reference_tails\genesis_by_coin_id.clsp.hex -a 0x<Coin ID>`。（请记住，&lt;Coin ID&gt; 之前的 0x 是必需的。）此命令的结果将是一个 <委派谜题>，您将把它作为解决方案的一部分传递给您的主 TAIL。

    g. 使用附加的 --treehash 标志再次运行相同的命令。这会给你&lt;treehash&gt;您刚刚创建的拼图：
    
    `cdv clsp curry .\reference_tails\genesis_by_coin_id.clsp.hex -a 0x<Coin ID> --treehash`

    h. 使用 &lt;Fingerprint&gt; 对 treehash 进行签名（此处不需要 0x）。您通过运行此命令在上面注意到：
    
    `chia keys sign -d <treehash> -f <Fingerprint> -t m -b`
    
    最后两个标志用于路径和字节。确保生成的公钥对应于 &lt;Fingerprint&gt;你刚用。复制&lt;签名&gt;用于下一步。

    i. 运行与上面相同的“cats”命令，但删除--select-coin 标志并添加--solution 标志，传入&lt;委托谜语&gt;你刚刚计算。这必须用引号和括号括起来，并且必须包含一个解决方案，我们将其留空。还要添加 --signature 标志，因此命令如下所示：

     `cats --tail .\reference_tails\delegated_tail.clsp.hex --curry 0x<Master public key> --send-to <wallet address> -a <amount in mojos to issue> --as-bytes --solution "(<delegated puzzle> ())" --signature <Signature>`

    该命令将输出两个值，&lt;Asset ID&gt;和&lt;花费捆绑&gt;。 &lt;资产 ID&gt;将是此 CAT 的 ID，因此保存此值以备后用很重要。 &lt;花费组合&gt;是一大堵文字墙。这是您将以字节格式推送到区块链的实际交易。

    j. 运行`cdv rpc pushtx <Spend Bundle>`。您应该会收到消息 "status": "SUCCESS", "success": true。

2. 为您的新 CAT 添加一个钱包 ID：

    a. 切换到您的轻钱包 GUI。在几分钟内，您的余额应该会减少您刚刚铸造的 mojo 数量。但是，它不会出现在您的交易中。该功能尚未实现。

    b. 现在您可以为您的新 CAT 添加钱包 ID。在左上角，单击“+ ADD TOKEN”，然后单击“+自定义”。在第一个文本字段中输入 CAT 的名称（可以是任何名称）。对于第二个文本字段，粘贴 &lt;Asset ID&gt;你从几步前保存。单击添加。

    c. 您现在将被带到您的新 CAT 钱包。余额应显示您选择使用的 XCH mojo 数量除以 1000。这是因为 CAT mojo 默认情况下是 CAT 的千分之一。
    
    > 注意：目前您可能会遇到两个外观错误，将在接下来的两个步骤中介绍。
    
    d. 如果您看到总余额为 0，则需要刷新您的钱包。运行`chia start wallet-only -r`。您现在应该看到正确的余额。这将在未来版本中修复。

    e. 您的猫的名字可能不会自动显示在您的钱包中。 在这种情况下，单击“状态：已同步”旁边的三个点，然后单击“重命名钱包”。 您可以将其重命名为其正确的名称。 这将在未来的版本中自动完成。

     就像前面的示例一样，您现在可以在 GUI 中访问您的 CAT。

3. 由于此 CAT 使用委托的 TAIL，您可以通过重新执行本节中的步骤 1 来发行新的铸币。 运行“cdv rpc pushtx”命令后，您的CAT钱包中的余额将根据新的铸币数量增加。

<details>
<summary>原文参考</summary>

- ## Creating a multiple mint CAT

If you're a visual learner, please see our [video tutorial for creating a multiple mint CAT](https://chialisp.com/docs/tutorials/multiple_issuance_CAT "Multiple mint CAT video tutorial").

Next we’ll create a CAT capable of minting tokens multiple times. This CAT uses a delegated TAIL, which is much more flexible than the previous one. As long as you sign a puzzlehash that you specify, you can mint new tokens using whatever TAIL you want. This allows for features such as rebate offers and distributed minting and retiring of tokens.

You can find the TAIL we'll use for this example [here](https://github.com/Chia-Network/chia-blockchain/blob/protocol_and_cats_rebased/chia/wallet/puzzles/delegated_genesis_checker.clvm "Delegated TAIL").

We’ll set up this CAT to delegate the same TAIL we set up previously. What this means is that nobody else can mint new tokens until you allow it. Keep in mind that this is only one of many possible implementations of a delegated TAIL.

1. Find a coin to mint, and create and push a new spendbundle:

    a. Change to the "CAT-admin-tool-main" folder if you're not already there.

    b. Figure out how many XCH mojos you want to use to issue your CAT. By default each CAT token will contain 1000 mojos, so you should multiply the number of tokens you want to mint by 1000. For example, if you want to mint 1 million tokens, you'll need 1 billion XCH mojos (1/1000 of an XCH).

    c. Take note of your Receive Address in the standalone GUI.

    d. Run `chia keys show`. Take note of your &lt;Fingerprint&gt; and &lt;Master public key&gt;.

    e. Run `cats --tail .\reference_tails\delegated_tail.clsp.hex --curry 0x<Master public key> --send-to <wallet address> -a <XCH mojos> --as-bytes --select-coin`

    The --select-coin flag will choose a coin from your wallet to issue the CAT from. The final line of the output will be "Name: &lt;Coin ID&gt;". You’ll use "&lt;Coin ID&gt;" in the next step.

    Now that you have a coin, you can create a full delegated TAIL. In our case, the TAIL it delegates will be of the single-mint variety.
    
    f. Run `cdv clsp curry .\reference_tails\genesis_by_coin_id.clsp.hex -a 0x<Coin ID>`. (Keep in mind the 0x before &lt;Coin ID&gt; is necessary.) The result of this command will be a &lt;delegated puzzle&gt;, which you’ll pass in as part of the solution to your main TAIL.

    g. Run the same command again, with the additional --treehash flag. This will give you the &lt;treehash&gt; of the puzzle you just created:
    
    `cdv clsp curry .\reference_tails\genesis_by_coin_id.clsp.hex -a 0x<Coin ID> --treehash`

    h. Sign the treehash (you do not need 0x here) with the &lt;Fingerprint&gt; you noted above by running this command:
    
    `chia keys sign -d <treehash> -f <Fingerprint> -t m -b`
    
    The last two flags are for the path and bytes. Make sure the resulting Public Key corresponds to the &lt;Fingerprint&gt; you just used. Copy the &lt;Signature&gt; to use in the next step.

    i. Run the same "cats" command as above, but remove the --select-coin flag and add the --solution flag, passing in the &lt;delegated puzzle&gt; you just calculated. This must be surrounded by quotes and parenthesis, and it must contain a solution, which we'll leave empty. Add the --signature flag as well, so the command looks like this:

    `cats --tail .\reference_tails\delegated_tail.clsp.hex --curry 0x<Master public key> --send-to <wallet address> -a <amount in mojos to issue> --as-bytes --solution "(<delegated puzzle> ())" --signature <Signature>`

    This command will output two values, &lt;Asset ID&gt; and &lt;Spend Bundle&gt;. &lt;Asset ID&gt; will be the ID of this CAT, so it’s important to save this value for later. &lt;Spend Bundle&gt; is a large wall of text. This is the actual transaction that you will push to the blockchain, in byte format.

    j. Run `cdv rpc pushtx <Spend Bundle>`. You should receive the message "status": "SUCCESS", "success": true.

2. Add a wallet ID for your new CAT:

    a. Switch to your light wallet GUI. Within a few minutes, your balance should decrease by the number of mojos you just minted. It won’t show up in your transactions, though. That feature has not yet been implemented.

    b. Now you can add a wallet ID for your new CAT. In the upper left corner, click "+ ADD TOKEN", then click "+ Custom". Enter the name of your CAT (it can be anything) in the first text field. For the second text field, paste the &lt;Asset ID&gt; you saved from a few steps ago. Click ADD.

    c. You will now be taken to your new CAT wallet. The balance should show the number of XCH mojos you chose to use, divided by 1000. This is because CAT mojos by default are one-thousandth of a CAT.
    
    > NOTE: There are currently two cosmetic bugs you might run into, covered in the next two steps.
    
    d. If you see a Total Balance of 0, you need to refresh your wallet. Run `chia start wallet-only -r`. You should now see the correct balance. This will be fixed in a future release.

    e. Your cat's name might not show up in your wallet automatically. In this case, click the three dots next to "Status: Synced" and click "Rename Wallet". You can rename it to its proper name. This will be done automatically in a future release.

    Just like the previous example, you now have access to your CAT in the GUI.

3. Because this CAT uses a delegated TAIL, you can issue new mintings by re-doing step 1 from this section. After you run the “cdv rpc pushtx” command, the balance in your CAT wallet will increase according to the new minting.

</details>

## 准备主网

在您对在测试网上发布您的 CAT 感到满意后，您可能希望转移到主网上。请记住，在公共区块链上发布代码存在额外的风险。如果您的 CAT 和/或 TAIL 未安全创建，您的资金可能会被冻结或被盗。 **谨慎行事。**

也就是说，向主网发布 CAT 与向测试网发布 CAT 并没有太大区别。你仍然需要一个同步的全节点和一个轻钱包。

一个区别是，对于主网发行，您的完整节点可能基于“主”代码分支。这将节省您同步完整节点的时间。你的轻钱包仍然需要在 `protocol_and_cats_rebased` 分支上运行。

当您准备好将 CAT 发布到主网时，第一步是运行 `chia configure -t false`，这将指示 Chia 将您的配置切换到主网。

接下来，您需要以安全的方式生成和保护您的密钥，我们将在下一节中讨论。

> 注意：您可能想知道如何在我们 GUI 的默认选项中列出您的 CAT。我们将在 2022 年 1 月左右正式确定此流程，但尚未完成。但是不要让这阻止您创建自己的 CAT。即使它们未在我们的 GUI 中列出，它们也将具有完整的功能。

<details>
<summary>原文参考</summary>

- ## Preparing for mainnet

After you are comfortable with issuing your CAT on testnet, you may wish to move to mainnet. Please keep in mind that there are extra risks inherent to publishing code on a public blockchain. If your CAT and/or TAIL have not been created securely, your funds could potentially be bricked or stolen. **Proceed with caution.**

That said, issuing a CAT to mainnet isn't very different from issuing one to testnet. You'll still need a synced full node and a light wallet.

One difference is that for mainnet issuance, your full node could be based off of the `main` code branch. This will save you time in syncing your full node. Your light wallet will still need to be running on the `protocol_and_cats_rebased` branch.

When you are ready to issue your CAT to mainnet, the first step is to run `chia configure -t false`, which will instruct Chia to switch your configuration to mainnet.

Next, you'll need to generate and protect your keys in a secure manner, which we'll discuss in the following section.

> NOTE: You may be wondering how to get your CAT listed among the default options in our GUI. We will formalize this process around January 2022, but it has not yet been completed. But don't let that discourage you from creating your own CATs. They will be fully functional, even if they are not listed in our GUI.

</details>

## 生成安全密钥对

在我们引导您安全地生成和保存新的公钥/私钥对的过程之前，请阅读此重要消息。

> **警告：** 您将要使用的密钥对将控制这些令牌的铸造和淘汰 ** 永远**。如果私钥被泄露，攻击者可以铸造新令牌，以及融化任何他们拥有进入常规 XCH。
>
> 使攻击无效的唯一方法是跟踪非法造币厂（幸运的是，所有这些都在公共分类账上完全可见），发布新的 CAT，然后为新的 CAT 类型提供合法的旧 CAT 交换。
>
> 这将是一个复杂且耗时的过程，可能会导致人们在某个时候被出售假冒 CAT。保密您的私钥非常重要。

以下是生成用于发布新 CAT 的安全公钥/私钥对的方法：

1. 打开您选择的命令提示符并切换到 `chia-blockchain` 目录。

2. 运行 `.\venv\Scripts\Activate.ps1` 来激活你的虚拟环境。

3. 运行 `chia keys show`。

    a. 如果您收到此消息：`There are no saved private keys`，请继续执行步骤 4。

    b. 如果您的机器上存储了任何密钥，它们现在会显示出来。请注意每个密钥的指纹，因为您_不会_使用这些密钥来创建您的 CAT，并且您不想将它们与您将要创建的密钥混淆。更好的是，如果这对您来说是一个选择，那就是删除您现有的密钥，以避免将它们与新密钥混淆。在删除密钥之前，请务必将助记词种子保存在安全的地方。

4. 运行 `chia init`。如果尚未设置，这将初始化您的环境。
    
5. 运行 `chia keys generate`。这将生成一个新的公钥/私钥对。

6. 运行 `chia keys show --show-mnemonic-seed`。这将显示您的公钥和私钥，以及您的助记符种子。

7. 将新密钥对的 `助记种子（24 个秘密词）` 复制到安全的离线位置。这 24 个词是您将来恢复钱包所需的全部内容。

<details>
<summary>原文参考</summary>

- ## Generating a secure key pair

Before we walk you through the process of securely generating and saving a new public/private key pair, please read this important message.

>**WARNING:** The key pair you are about to use will control the minting and retirement of these tokens **forever.** If the private key were ever compromised, an attacker could mint new tokens, as well as melt any that they owned into regular XCH.
>
>The only way to nullify an attack would be to keep track of illegitimate mints (luckily all of this is fully visible on the public ledger), issue a new CAT, and then offer an exchange of legitimate old CATs for the new CAT type.
>
>This would be a complex and time-consuming process that would likely result in people being sold counterfeit CATs at some point. It’s very important to keep your private key secret.

Here's how to generate a secure public/private key pair for issuing your new CAT:

1. Open your choice of command prompt and change to the `chia-blockchain` directory.

2. Run `.\venv\Scripts\Activate.ps1` to activate your virtual environment.

3. Run `chia keys show`.

    a. If you receive this message: `There are no saved private keys`, then proceed to step 4.

    b. If you have any keys stored on your machine, they'll be shown now. Note the Fingerprint for each key, as you will _not_ be using these keys to create your CAT and you don't want to confuse them for the key you are about to create. Even better, if this is an option for you, would be to delete your existing keys to avoid confusing them for the new ones. Just be sure to save your mnemonic seed somewhere secure before you delete your keys.

4. Run `chia init`. This will initialize your environment if it has not yet been set up.
    
5. Run `chia keys generate`. This will generate a new public/private key pair.

6. Run `chia keys show --show-mnemonic-seed`. This will show your public and private keys, as well as your Mnemonic seed.

7. Copy your new key pair's `Mnemonic seed (24 secret words)` to a secure offline location. These 24 words are all you'll need to restore your wallet in the future.

</details>

## 潜在的未来添加

由于我们的 Cat1 标准仍处于草案形式，因此本文档可能需要多次更新。将来，我们可能会添加用于发布具有不同 TAIL 的 CAT 的部分，并且我们将在 `protocol_and_cats_rebased` 分支合并到 `main` 分支时进行重大修订。

请务必回来查看未来的更新。祝你好运，铸币快乐！

<details>
<summary>原文参考</summary>

- ## Potential future additions

As our Cat1 standard is still in draft form, this document will likely require multiple updates. In the future, we may add sections for issuing CATs with different TAILs, and we'll do a major revision when the `protocol_and_cats_rebased` branch is merged into the `main` branch.

Be sure to check back for future updates. Good luck and happy minting!

</details>
