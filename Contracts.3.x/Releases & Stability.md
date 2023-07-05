# New Releases and API Stability
开发智能合约很困难，有时候会倾向于保守地处理依赖关系。然而，及时了解新版本的发布非常重要：这些版本可能包括错误修复，或者废弃旧模式以支持更新和更好的实践方法。

下面我们将描述你作为OpenZeppelin Contracts用户应该期待何时发布新版本，以及这对你的影响。

## 发布计划
OpenZeppelin Contracts遵循*语义化版本控制*。

### 次要版本
OpenZeppelin Contracts的目标是每1到2个月发布一个新的次要版本。

在发布周期开始时，我们决定要优先处理哪些问题，并将它们分配给[GitHub上的一个里程碑](https://github.com/OpenZeppelin/openzeppelin-contracts/milestones)。在接下来的五个星期内，我们会对这些问题进行修复和处理。

一旦里程碑完成，我们会发布一个功能冻结的发布候选版本。发布候选版本的目的是让社区在实际发布之前有一个期间来审查新代码。如果发现重要问题，可能需要发布更多的候选版本。在发布候选版本不再有更改的一周之后，新版本就会发布。

### 主要版本
几个月或一年后可能会发布一个新的主要版本。这些版本没有计划，但会根据需要发布破坏性变更，如库的核心功能重新设计（例如3.0中的[访问控制](https://github.com/OpenZeppelin/openzeppelin-contracts/pulls/2112)）。由于我们重视稳定性，我们希望这些变更不经常发生（主要版本之间的时间间隔至少为六个月）。然而，如果Solidity语言发生重大变化，我们可能被迫发布一个主要版本。

## API稳定性
在[OpenZeppelin 2.0版本](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v2.0.0)发布时，我们承诺保持稳定的API。我们希望更准确地定义我们对稳定性和API的理解，以便库的用户能够了解这些保证，并对其项目不会意外中断感到自信。

简而言之，API的稳定性意味着如果你的项目今天可以工作，它将继续工作。新的合约和功能将在次要版本中添加，但只会以向后兼容的方式添加。

### 版本控制方案
我们遵循语义化版本控制，这意味着在主要版本之间可能会出现API的破坏性变更（这种情况并不经常发生）。

### Solidity函数
虽然函数的内部实现可能会发生变化，但它们的语义和签名将保持不变。它们的参数领域不会变得更加宽松（例如，如果不允许转移0值，那么它将继续不被允许），一般状态限制也不会被解除（例如，当Paused修饰符存在时）。

如果向合约添加新的函数，将以向后兼容的方式进行：它们的使用不是强制性的，并且它们不会以可预见的方式扩展功能，从而可能破坏应用程序（例如，[可以添加一个内部方法来更轻松地检索已经可用的信息](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1512)）。

### internal
这不仅适用于外部和公共函数，也适用于内部函数：许多合约是通过继承来使用的（例如，Pausable，PullPayment，不同的Roles合约），因此通过调用这些函数来使用它们。同样，由于所有OpenZeppelin Contracts状态变量都是私有的，因此只能通过这种方式访问它们（例如，要创建新的ERC20代币，而不是手动修改totalSupply和balances，应该调用_mint）。

私有函数在行为、使用和存在方面没有任何保证。

最后，有时候语言的限制会迫使我们将本应该是私有的函数标记为internal，以便按照我们希望的方式实现功能。这些情况将有很好的文档说明，并且正常的稳定性保证将不适用。

### 库
我们的一些Solidity库使用结构体处理不应直接访问的内部数据（例如Roles）。在Solidity存储库中有一个[未解决的问题](https://github.com/ethereum/solidity/issues/4637)，请求一种语言功能来防止这种访问，但看起来它在短期内不会被实现。因此，我们将使用前导下划线并标记这些结构体，以清楚地告诉用户其内容和布局不是API的一部分。

### 事件
不会删除任何事件，并且它们的参数不会以任何方式更改。新的事件可能会在以后的版本中添加，并且现有的事件可能会在新的合理情况下被触发（例如，[从2.1开始，ERC20还会在transferFrom调用时触发Approval事件](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/707)）。

### 燃气成本
尽管通常会尝试降低使用OpenZeppelin Contracts的燃气成本，但对此没有任何保证。特别是，用户不应该假设升级库版本时燃气成本不会增加。

### 错误修复
为了修复错误，可能需要打破API稳定性的保证，我们将这样做。然而，这个决定不会轻易做出，我们将探索所有选项，以使变更尽可能不会造成中断。在必要时，可能会将可能导致不安全行为的合约或函数废弃而不是删除（例如[＃1543](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1543)和[＃1550](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1550)）。

### Solidity编译器版本
从版本0.5.0开始，Solidity团队切换到了更快的发布周期，每几周发布一个次要版本（v0.5.0发布于2018年11月，v0.5.5发布于2019年3月），每几个月发布一个主要的、破坏性的版本（v0.6.0发布于2019年12月，v0.7.0发布于2020年7月）。将Solidity编译器版本包含在OpenZeppelin Contracts的稳定性保证中将迫使库要么坚持使用旧的编译器，要么频繁发布主要更新以跟上较新的Solidity发布。

因此，最低要求的Solidity编译器版本不是稳定性保证的一部分，用户在使用较新版本的Contracts时可能需要升级其编译器。错误修复仍将被回溯到旧版本库中，以便所有当前使用的版本都能收到这些更新。

你可以在[这里](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1498#issuecomment-449191611)阅读更多关于这一决策背后的原理、我们考虑的其他选项以及为什么选择这条路的原因。