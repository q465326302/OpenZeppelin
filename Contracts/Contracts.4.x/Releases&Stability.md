# New Releases and API Stability
开发智能合约很难，对采取保守的方法有时更受青睐。然而，及时了解新版本的发布也非常重要：这些版本可能包括错误修复，或废弃旧模式以支持更新和更好的实践。

在这里，我们描述了何时应该预期新版本发布，以及这对你作为OpenZeppelin Contracts的用户的影响。

## 发布计划
OpenZeppelin Contracts遵循[语义化版本控制方案](#版本控制方案)。

我们的目标是每1到2个月发布一个新的次要版本。

### 发布候选版
在每次发布之前，我们会发布一个功能冻结的候选版。发布候选版的目的是在实际发布之前有一个社区审核新代码的时间。如果发现重要问题，则可能需要发布更多的候选版。在发布候选版没有更多变化的一周后，新版本将发布。

### 主要版本
几个月或一年后，可能会发布一个新的主要版本。这些版本没有计划，但将基于需要发布破坏性更改（例如3.0版本中的[核心功能重新设计](https://github.com/OpenZeppelin/openzeppelin-contracts/pulls/2112)，如访问控制）。由于我们重视稳定性，我们希望这些版本发布不频繁（主要版本之间的间隔时间不少于六个月）。然而，当Solidity语言发生重大变化时，我们可能不得不发布一个主要版本。

## API稳定性
在[OpenZeppelin Contracts 2.0版本](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v2.0.0)发布时，我们承诺保持稳定的API。我们的目标是更精确地定义我们对稳定性和API的理解，以便库的用户可以理解这些保证，并且可以确信他们的项目不会意外中断。

简而言之，API稳定意味着如果你的项目今天正常工作，那么在次要升级后它仍将继续正常工作。新合约和功能将在次要版本中添加，但只会以向后兼容的方式添加。

### 版本控制方案
我们遵循[SemVer](https://semver.org/)，这意味着API在主要版本之间可能会发生破坏性变化（这种情况[不会经常发生](#主要版本)）。

### Solidity函数
虽然函数的内部实现可能会发生变化，但它们的语义和签名将保持不变。它们的参数域不会变得更宽松（例如，如果不允许转移值为0，变化后仍然不允许），也不会解除一般状态限制（例如，在暂停时修改器）。

如果向合约添加新函数，它将以向后兼容的方式添加：它们的使用不是强制性的，并且它们不会以可能预见地破坏应用程序的方式扩展功能（例如，可以添加内部方法以使检索已经可用的信息更容易）。

#### internal
这不仅适用于外部和公共函数，还适用于内部函数：许多合约是用于继承它们的（例如，Pausable，PullPayment，AccessControl），因此通过调用这些函数使用它们。同样，由于所有OpenZeppelin Contracts状态变量都是私有的，因此只能通过这种方式访问它们（例如，要[创建新的ERC20代币，而不是手动修改totalSupply和balances，应该调用_mint](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1512)）。

私有函数对其行为、使用或存在没有任何保证。

最后，有时语言限制将迫使我们将本应为私有的函数成为内部函数，以便按照我们想要的方式实现功能。这些情况将有很好的文档记录，并且正常的稳定性将不适用。

### 库
我们的一些Solidity库使用结构体处理不应直接访问的内部数据（例如，Counter）。在Solidity存储库中有一个[开放问题](https://github.com/ethereum/solidity/issues/4637)，请求一种防止访问的语言特性，但看起来它不会很快实现。因此，我们将使用前导下划线并标记该结构体，以使用户清楚其内容和布局不是API的一部分。

### 事件
不会删除任何事件，并且它们的参数不会以任何方式更改。可以在后续版本中添加新事件，并且现有事件可以在新的合理情况下发出（例如，[在2.1版本开始，ERC20还会在transferFrom调用中发出Approval](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/707)）。

### 草案
一些合约实现了仍处于草案状态的EIP，这些合约的文件名以draft-开头，例如utils/cryptography/draft-EIP712.sol。由于它们作为草案的性质，这些合约的细节可能会发生变化，我们无法保证它们的稳定性。 OpenZeppelin Contracts的次要版本可能会包含标记为草案的合约的破坏性更改，这将在[更改日志](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/CHANGELOG.md)中充分的公布。包含的EIP被生产项目使用，这可能使它们不太可能发生重大变化。

### gas成本
虽然通常会尝试降低使用OpenZeppelin Contracts的gas成本，但不能保证这一点。特别是，用户不应假设升级库版本时gas成本不会增加。

### 错误修复
为了修复错误，可能需要打破API稳定性保证，我们会这样做。但是，这个决定不会轻易做出，我们将尽一切可能采取措施使更改尽可能不会造成中断。当足够时，可能导致不安全行为的合约或函数将被弃用而不是删除（例如[＃1543](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1543)和[＃1550](https://github.com/OpenZeppelin/openzeppelin-contracts/pull/1550)）。

### Solidity编译器版本
从版本0.5.0开始，Solidity团队开始更快的发布周期，每隔几周发布次要版本（v0.5.0于2018年11月发布，v0.5.5于2019年3月发布），每隔几个月发布主要的破坏性版本（v0.6.0于2019年12月发布，v0.7.0于2020年7月发布）。将Solidity编译器版本包含在OpenZeppelin Contracts的稳定性保证中将迫使库要么坚持使用旧的编译器，要么频繁发布主要更新以跟上较新的Solidity版本。

因此，**最低要求的Solidity编译器版本不是稳定性保证的一部分**，当用户在使用较新版本的Contracts时可能需要升级编译器。错误修复仍将被回溯到过去的主要版本，以便正在使用的所有版本都可以接收这些更新。

你可以在[这里](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1498#issuecomment-449191611)阅读更多关于这一决策背后的原因、我们考虑的其他选项以及为什么选择这条道路的信息。