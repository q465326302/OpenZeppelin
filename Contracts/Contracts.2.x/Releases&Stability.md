# New Releases and API Stability
开发智能合约很困难，有时候会倾向于保守地对待依赖关系。然而，及时了解新版本的发布非常重要：这些新版本可能包含错误修复，或者废弃旧的模式以支持更新和更好的实践。

下面我们将介绍你应该什么时候期望新版本发布，并且这如何影响你作为OpenZeppelin Contracts的用户。

## 发布计划
OpenZeppelin Contracts遵循[语义版本控制方案](#版本控制方案)。

### 次要版本
OpenZeppelin Contracts**每5周**发布一个新版本。

在发布周期开始时，我们会决定要优先解决哪些问题，并将它们分配给[GitHub上的里程碑](https://github.com/OpenZeppelin/openzeppelin-contracts/milestones)。在接下来的5周内，这些问题会被修复和处理。

一旦里程碑完成，我们会发布一个功能冻结的发布候选版本。发布候选版本的目的是在实际发布之前，给社区提供一个审核新代码的时期。如果发现重要问题，可能需要发布更多的候选版本。在发布候选版本没有更多变化的一个星期后，新版本会发布。

### 主要版本
每隔几个月可能会发布一个新的主要版本。这些版本没有计划，但会基于需要发布破坏性变更，比如库的核心特性的重新设计（例如2.0中的[roles](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1146)）。由于我们重视稳定性，我们希望这些变更不经常发生（至少间隔6个月）。然而，如果Solidity语言发生了重大变化，我们可能被迫发布一个主要版本。

## API稳定性
在[OpenZeppelin 2.0](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v2.0.0)版本中，我们承诺保持稳定的API。我们的目标是更精确地定义我们所理解的稳定和API，并让库的用户理解这些保证，并对他们的项目不会出现意外中断充满信心。

简而言之，API的稳定意味着如果你的项目今天工作，它将继续工作。新的合约和功能将在次要版本中添加，但只会以向后兼容的方式添加。这个规则的例外是*Drafts*类别中的合约，这些合约应该被视为不稳定的。

### 版本控制方案
我们遵循[语义化](https://semver.org/)版本控制，这意味着在主要版本之间可能发生API破坏（这种情况并[不经常发生](#发布计划)）。

### Solidity函数
虽然函数的内部实现可能会改变，但它们的语义和签名将保持不变。它们的参数领域不会更加宽松（例如，如果不允许转移值为0，它将仍然不允许），也不会解除一般的状态限制（例如，当Paused修饰符存在时）。

如果向合约添加了新的函数，它将以向后兼容的方式进行添加：使用它们不是强制性的，并且它们不会以可能破坏应用程序的方式扩展功能（例如，[可以添加一个内部方法以更轻松地检索已经可用的信息](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1512)）。

#### internal
这不仅适用于外部和公共函数，也适用于内部函数：许多合约是通过继承来使用的（例如Pausable、PullPayment和不同的Roles合约），因此通过调用这些函数来使用它们。同样，由于所有OpenZeppelin Contracts状态变量都是私有的，所以只能通过这种方式来访问它们（例如，创建新的ERC20代币时，应该调用_mint，而不是手动修改totalSupply和balances）。

私有函数对其行为、使用和存在没有任何保证。

最后，有时候语言限制会迫使我们将本应该是私有的函数设置为internal，以便按照我们希望的方式实现功能。这些情况将被很好地记录，并且正常的稳定性保证将不适用。

### Libraries
我们的一些Solidity库使用struct来处理不应直接访问的内部数据（例如Roles）。在Solidity存储库中有一个[开放的问题](https://github.com/ethereum/solidity/issues/4637)，请求一种语言特性来防止此类访问，但看起来它不会很快实现。因此，我们将使用前导下划线和标记该struct，以明确告知用户它的内容和布局不是API的一部分。

### 事件
不会删除任何事件，它们的参数也不会以任何方式改变。新的事件可能会在以后的版本中添加，现有的事件可能会在新的合理情况下被触发（例如，[从2.1版本开始，ERC20还会在transferFrom调用中触发Approval事件](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/707)）。

### gas成本
尽管通常会尝试降低使用OpenZeppelin Contracts的gas成本，但不能保证这一点。特别是，用户不应该假设在升级库版本时gas成本不会增加。

### 错误修复
为了修复错误，可能需要打破API稳定性的保证，我们将这样做。然而，我们不会轻易做出这个决定，并且会探索所有选项，使更改尽可能不会造成破坏。如果有足够的理由，可能会废弃可能导致不安全行为的合约或函数，而不是删除它们（例如，[＃1543](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1543)和[＃1550](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1550)）。

### Solidity编译器版本
从版本0.5.0开始，Solidity团队采用了更快的发布周期，每隔几周发布次要版本（v0.5.0于2018年11月发布，v0.5.5于2019年3月发布），每隔几个月发布一次主要的、破坏性变更的版本（v0.6.0计划于2019年3月底发布）。将Solidity编译器版本包含在OpenZeppelin Contracts的稳定性保证中，将迫使库要么坚持使用旧的编译器，要么频繁发布主要更新以跟上较新的Solidity版本。

因此，**最低要求的Solidity编译器版本不是稳定性保证的一部分**，当使用较新版本的Contracts时，用户可能需要升级其编译器。错误修复仍将被反向移植到较旧的库版本，以便所有正在使用的版本都能获得这些更新。

你可以在[这里](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1498#issuecomment-449191611)阅读更多关于这一决策的理由、我们考虑的其他选项以及为什么选择这条道路的内容。