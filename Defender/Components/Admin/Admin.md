# 管理员
Defender Admin 服务充当通过安全的多重签名合约或时间锁管理您的智能合约项目的接口。 Defender Admin 完全不控制您的系统，该系统完全由签名者的密钥控制。

##  使用案例
在需要安全管理智能合约的链上操作时，请使用Defender Admin。任何管理操作都不应由单个所有者单方面控制。相反，设置一个多签名钱包，并依靠Defender Admin通过它运行以下任何操作：

* 将合约升级到新实现

* 调整影响协议行为的数值参数

* 管理受限操作的访问控制列表

* 在紧急情况下暂停合约

## 合同和提案
要开始在Defender Admin中管理您的合约，第一步是注册它。这是一个一次性的过程，您需要指定网络和地址，并为您的合约分配一个名称。如果合约已经被验证，合约的ABI将自动从etherscan或sourcify中提取，否则您需要手动输入它。

>NOTE
Defender Admin会自动尝试检测您的合约的一些特征。今天，它将检测它是否是一个[EIP1967兼容](https://eips.ethereum.org/EIPS/eip-1967)或传统的zOS代理（在这种情况下加载实现的ABI），以及它是否由*ProxyAdmin*合约管理。如果您已经使用[Sourcify](https://sourcify.dev/)验证了您的合约，您的NatSpec注释将在管理面板中可用。

添加合同后，您可以为其创建新的提案。每个提案都是您要在合同上执行的操作，该操作通过多重签名合同执行，并需要达成一定数量的管理员同意。创建后，其他管理员可以审查和批准提案，直到其达到批准阈值并被执行。

或者，如果该函数未限制只能通过多重签名调用，则还可以选择直接在管理员上执行合同上的操作。
![admin-1.png](img/admin-1.png)

>NOTE
在创建新提案时，Defender Admin将首先模拟它，并且如果该操作回退，它将拒绝创建它，并显示合同返回的回退原因。
![admin-2.png](img/admin-2.png)

这是一个视频演示，展示了使用EOA和多重签名来提出和批准管理操作的过程：
https://youtu.be/XJ3UUNYlbxg

## API访问
您可以通过*Defender Admin API*以编程方式向您的Admin仪表板添加合约并创建新的提案。请查看[defender-admin-client npm](https://www.npmjs.com/package/defender-admin-client)包以获取更多信息。

## 提案类型
Defender Admin目前支持三种提案类型：升级、暂停和自定义操作。未来将会添加更多的提案类型。

### 升级
升级操作只能在支持 EIP1967 或遗留的 zOS 可升级代理上执行，这些代理公开了 upgradeTo(address) 函数。Defender Admin 将直接处理由多重签名钱包直接拥有的代理或由 ProxyAdmin 管理的代理，而 ProxyAdmin 又由钱包拥有。升级操作只需要选择新的实现地址，Defender Admin 就会处理其余的事情。

>WARNING
Defender Admin目前不会验证实现的存储布局兼容性。因此，我们强烈建议使用openzeppelin-upgrades库来*部署目标实现*。可以使用*Truffle*中的prepareUpgrade函数或*Hardhat*中的defender.proposeUpgrade函数来完成此操作。

### 暂停/恢复
可以在ABI公开了pause()函数的合约上执行暂停操作。如果ABI还公开了unpause()函数，Defender Admin也会让您执行unpause操作。暂停和取消暂停操作仅需要您指定应通过哪个Admin账户执行。

此外，如果您的合约ABI公开了paused()或isPaused()函数并返回布尔结果，Defender将查询它，并在合约的仪表板页面上显示其状态，如下图所示。
![admin-3.png](img/admin-3.png)

### 访问控制管理
访问控制操作是指*符合OpenZeppelin访问*控制的grantRole和revokeRole接口的调用，通过其ABI实现。

Defender提供了创建此类提案的功能，当ABI包含grantRole和revokeRole函数时，您可以使用合约已经索引的角色来填充它们（通过从[TheGraph](https://thegraph.com/)消耗的事件，如果合约的网络受支持，否则通过getLogs JSON-RPC调用索引）。

如果您想提供一个未在您的合约中索引的角色，您可以通过给它相应的bytes32来为您的合约提供新的角色。
![admin-4.png](img/admin-4.png)

此外，如果检测到合约具有访问控制接口，我们会提供一个小部件，以便轻松地浏览合约角色、它们的管理员和成员。
![admin-5.png](img/admin-5.png)

### 定制操作
自定义操作是对托管合约中的任何函数的调用。Defender管理员将处理事务数据的编码，并通过选择的多重签名钱包将其提交为新提案。

如果未指定多重签名钱包，则Defender将直接将交易发送到合约。

自定义操作也可以重复，这将向您呈现预填充的表单，以便您在批准之前审查和调整操作。

>WARNING
目前还不支持某些 ABI 类型，例如嵌套结构体。如果您需要调用当前不支持的函数，请联系我们！

### 批次
批处理提案是从多个合约（或仅一个）调用多个函数的呼叫。它允许您在单个交易中原子地执行多个操作。

>NOTE
目前仅支持使用多重签名（例如Gnosis Safe）进行批量交易，方法是通过将DELEGATECALL编码到[Gnosis MultiSendCallOnly](https://github.com/safe-global/safe-contracts/blob/v1.3.0/contracts/libraries/MultiSendCallOnly.sol)模块中。

>NOTE
批量交易也需要在同一网络上进行。

Defender通过在主仪表板上点击“添加提案”选项来提供此功能，通过管理员UI实现。
![admin-6.png](img/admin-6.png)

通过使用此提案创建工作流程，您可以在定义第一步之后添加新步骤，然后再创建提案。例如，您可以添加“修改访问控制”步骤，然后添加另一个步骤来调用自定义操作。这将显示在提案创建审核页面中，并在创建之前作为整体进行验证。
![admin-7.png](img/admin-7.png)

>NOTE
目前仅支持访问控制管理和自定义操作。

## 多重签名钱包
Defender Admin支持两种多签钱包：[Gnosis Safe](https://gnosis-safe.io/)和[Gnosis MultisigWallet](https://github.com/gnosis/MultiSigWallet)。如果您使用的多签实现不受支持，请告诉我们！

### Gnosis Safe
Gnosis Safe钱包收集每个管理员的离线签名，然后提交一个包含所有签名的单个交易来执行操作。为了分享签名，它依赖于由Gnosis托管的[Safe Transaction Service](https://safe-transaction.gnosis.io/)。

>NOTE
安全交易服务仅在Mainnet、xDai、BSC、Polygon、Avalanche、Aurora、Optimism、Arbitrum、Goerli和Sepolia上可用。但是，您可以在任何网络上使用Defender Admin；如果交易服务不可用，它将跳过与其同步。

使用Gnosis Safe时，Defender Admin将同步所有签名到和从Safe Transaction Service。这样，您团队中使用[Safe UI](https://gnosis-safe.io/app)的任何管理员仍将能够签署Defender Admin提议。

>NOTE
Safe合约要求所有提案必须按顺序执行。如果你已经收集了所有提案的签名，但仍无法执行它，请确保没有先前的提案等待执行。

Gnosis Safe钱包还允许执行DELEGATECALL到其他合约中，以在多签上下文中执行任意代码。您可以使用“defender-admin-client”通过API创建管理操作提案来发出委托调用。

### 发送资金
Gnosis Safe钱包上的发送资金功能可以让您将网络原生资产从您的Gnosis Safe转移到您选择的接收者。

要使用此功能，您需要将Gnosis Safe钱包添加到Defender Admin合同集合中。在Admin中转到您的Gnosis Safe页面，点击新建提案→发送资金。然后选择收件人地址和要转移的资金金额。从那时起，它就像任何其他管理提案一样运作：您需要从足够的Gnosis Safe所有者那里收集批准才能执行交易。Defender会指导您完成该过程。

>NOTE
如果您最初从Defender部署了您的Gnosis Safe，它已经在您的Defender Admin合同列表中的Multisigs部分中。

我们正在努力扩展此功能，使其能够发送ERC20代币资金，请保持关注。

### Gnosis多签钱包

Gnosis MultisigWallet要求每个管理员提交新的交易以获得他们的批准，因此无需单独的服务来协调。

除了普通的MultisigWallet外，Defender Admin还支持dYdX开发的[PartiallyDelayedMultisig变体](https://gist.github.com/spalladino/1e853ce79254b9aea70c8b49fd7d9ab3#file-partiallydelayedmultisig-sol)。在这个钱包中，一旦提案获得批准，就需要等待一段时间锁定期才能执行。Defender Admin会从合约中加载这些信息并在界面上显示。

### 从Defender Admin管理您的多重签名

使用Defender创建Gnosis Safe多重签名
https://youtu.be/IOescPDrF7Y
您可以直接从Defender创建和部署新的Gnosis Safe多重签名钱包。这在官方Gnosis Safe UI尚未可用的网络中特别方便。要创建新的Gnosis Safe，请转到“管理”并单击“合同”，然后单击“创建Gnosis Safe”。您将进入一个简单的表单，要求您提供多重签名的初始所有者列表和门槛。就是这样！

### 从Defender修改您的多重签名设置
您可以通过创建自定义操作提案来修改您的多重签名设置，以执行管理函数addOwner或changeThreshold，就像您导入Defender中的任何其他合约一样。
![admin-8.png](img/admin-8.png)

## 时间锁
从Defender创建一个时间锁控制器
https://youtu.be/Yi9Y0bcj-Zc
您可以直接从Defender创建和部署新的Timelock Controller。要创建新的Timelock，请转到“管理”并单击“合同”，然后单击“创建Timelock”。您将进入一个简单的表格，在那里您将被要求提供最初的提议者和执行者列表，以及提议执行的最小延迟时间。

为了在etherscan上验证合同，您可以在下面找到源代码和编译器设置：

部署使用OpenZeppelin Contracts库提供的[TimelockController合同v4.8.1的原始实例](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v4.8.1/contracts/governance/TimelockController.sol)。

部署合同的编译器设置：
```
solidity: {
    version: "0.8.4",
    settings: {
        optimizer: {
            enabled: true,
            runs: 200
        }
    }
}
```

### 将合同所有权转移给TimelockController
为了利用时间锁执行函数，需要将其分配访问权限到相应的智能合约，可以通过角色或所有权来实现。可以通过调用transferOwnership或相*关角色*分配函数来完成。
https://youtu.be/cXDp2n5al7U

### 创建时间锁定提案
Defender Admin支持通过OpenZeppelin Contracts库提供的*TimelockController合约进行时间锁定的管理提案*。
https://youtu.be/59p98wGqdVo
要执行一个时间锁定的提案，您需要：

1. 一个多重签名（或EOA），作为TimelockController中的提案人。

2. 一个具有您想在合约上运行的操作权限的TimelockController。

一旦权限得到适当的设置，只需像平常一样创建一个提案，在执行策略部分勾选Timelock复选框。然后输入您的timelock地址，并选择批准提案和执行之间的最小延迟时间。
![admin-9.png](img/admin-9.png)

请注意，无论是通过多重签名还是EOA（外部账户）批准，您都可以创建一个时间锁定的提案。只要正确的链上权限结构得到设置，任何批准政策都应该可以运作。
![admin-10.png](img/admin-10.png)

### 管理时间锁定的提案
一旦您创建了一个时间锁定的提案，Defender将指导您和您的合作者完成它。假设您选择通过Gnosis Safe批准提案，从提案创建到基础管理操作执行的步骤如下：

1. 收集足够的多签所有者批准（根据多签的当前配置）。

2. 安排行动，具有指定的延迟期。请注意，使用的多签需要是TimelockController合同中的提议者。在此阅读更多信息。

3. 在指定的延迟期结束后执行操作。值得注意的是，在TimelockController合同中执行此操作的EOA需要是执行者。

>NOTE
目前Defender不支持定时锁定的升级提案。这项能力正在开发中，我们计划很快发布它。

https://youtu.be/z6EP6JTj7ME

### 治理
您还可以将Admin提案的控制委托给Governor合约。要创建Governor提案，只需将执行策略设置为Governor，并输入有效的Governor合约地址。Defender将执行基本检查以验证合约是否实际符合Governor接口，然后才会让您继续进行。

Defender Admin支持在OpenZeppelin的Governor合约以及Compound的Alpha和Bravo方言上创建提案。
![admin-11.png](img/admin-11.png)

