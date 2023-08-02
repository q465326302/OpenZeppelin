# Cross Chain Awareness
这个目录提供了构建块来提高智能合约的跨链意识。
* *CrossChainEnabled*是一个抽象层，包含访问器和修饰符，用于控制接收跨链消息时的执行流程。

## CrossChainEnabled的特殊化

*CrossChainEnabled*的以下特殊化为特定桥接的*CrossChainEnabled*抽象提供了实现。这可用于构建复杂的跨链感知组件，如*AccessControlCrossChain*。

### CrossChainEnabledAMB
```
import "@openzeppelin/contracts/crosschain/amb/CrossChainEnabledAMB.sol";
```
[AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge)专业化或*CrossChainEnabled*抽象化。
截至2020年2月，以下链之间可用AMB桥接：
* ETH ⇌ xDai
* ETH ⇌ qDai
* ETH ⇌ ETC
* ETH ⇌ BSC
* ETH ⇌ POA
* BSC ⇌ xDai
* POA ⇌ xDai
* Rinkeby ⇌ xDai
* Kovan ⇌ Sokol
*自v4.6版本起可用。*

**FUNCTIONS**
constructor(bridge)
_isCrossChain()
_crossChainSender()

#### constructor(address bridge)
公开# 

#### _isCrossChain() → bool
内部#
查看 *CrossChainEnabled._isCrossChain* 函数

#### _crossChainSender() → address
内部#
请查看 *CrossChainEnabled._crossChainSender*。

### CrossChainEnabledArbitrumL1
```
import "@openzeppelin/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL1.sol";
```
[Arbitrum](https://arbitrum.io/)专业化或*CrossChainEnabled*抽象L1侧（主网）。

此版本仅应部署在L1上，以处理源自L2的跨链消息。对于另一侧，请使用*CrossChainEnabledArbitrumL2*。

桥接合约由Arbitrum团队提供和维护。您可以在[Arbitrum的开发人员文档](https://developer.offchainlabs.com/docs/useful_addresses)中找到此合约在rinkeby测试网上的地址。

*自v4.6以来可用。*

**FUNCTIONS**
constructor(bridge)
_isCrossChain()
_crossChainSender()

#### constructor(address bridge)
内部#

#### _isCrossChain() → bool
内部#
查看*CrossChainEnabled._isCrossChain*方法

#### _crossChainSender() → address
内部#
查看 *CrossChainEnabled._crossChainSender*。

### CrossChainEnabledArbitrumL2
```
import "@openzeppelin/contracts/crosschain/arbitrum/CrossChainEnabledArbitrumL2.sol";
```
[Arbitrum](https://arbitrum.io/)专业化或*CrossChainEnabled*抽象层适用于L2（Arbitrum）。

此版本仅应部署在L2上，以处理源自L1的跨链消息。对于另一侧，请使用*CrossChainEnabledArbitrumL1*。

Arbitrum L2包括固定地址的ArbSys合约。因此，此*CrossChainEnabled*的专业化不包括构造函数。

*自v4.6起可用。*

> WARNING
目前在Arbitrum中存在一个错误，导致当该合约部署在代理后面时无法检测跨链调用。这将在网络升级到Arbitrum Nitro时修复，该升级计划于2022年8月31日进行。

**FUNCTIONS**
_isCrossChain()
_crossChainSender()

#### _isCrossChain() → bool
内部#
查看 *CrossChainEnabled._isCrossChain* 

#### _crossChainSender() → address
内部#
查看 *CrossChainEnabled._crossChainSender*。

### CrossChainEnabledOptimism
```
import "@openzeppelin/contracts/crosschain/optimism/CrossChainEnabledOptimism.sol";
```
[Optimism](https://www.optimism.io/)专业化或*CrossChainEnabled*抽象。

CrossDomainMessenger合约由Optimism团队提供和维护。您可以在[Optimism monorepo的部署部分](https://github.com/ethereum-optimism/optimism/tree/develop/packages/contracts/deployments)找到该合约在mainnet和kovan上的地址。

*自v4.6以来可用。*

**FUNCTIONS**
constructor(messenger)
_isCrossChain()
_crossChainSender()

#### constructor(address messenger)
内部#

#### _isCrossChain() → bool
内部#
查看 *CrossChainEnabled._isCrossChain*

#### _crossChainSender() → address
内部#
查看 *CrossChainEnabled._crossChainSender*

### CrossChainEnabledPolygonChild
```
import "@openzeppelin/contracts/crosschain/polygon/CrossChainEnabledPolygonChild.sol";
```
[Polygon](https://polygon.technology/)专业化或*CrossChainEnabled*抽象化的子侧（polygon/mumbai）。

此版本仅应部署在子链上，以处理源自父链的跨链消息。

fxChild合约由多边形团队提供和维护。您可以在[Polygon的Fx-Portal文档](https://docs.polygon.technology/docs/develop/l1-l2-communication/fx-portal/#contract-addresses)中找到此合约的地址，包括polygon和mumbai。

*自v4.6以来可用。*

**FUNCTIONS**
constructor(fxChild)
_isCrossChain()
_crossChainSender()
processMessageFromRoot(, rootMessageSender, data)

REENTRANCYGUARD
_reentrancyGuardEntered()

#### constructor(address fxChild)
内部#

#### _isCrossChain() → bool
内部#
查看 *CrossChainEnabled._isCrossChain*

#### _crossChainSender() → address
内部#
查看 *CrossChainEnabled._crossChainSender*

#### processMessageFromRoot(uint256, address rootMessageSender, bytes data)
外部#
外部入口点用于接收和转发来自fxChild的消息。

非可重入性非常重要，以避免跨链调用能够通过仅使用用户定义的参数循环执行来模拟任何人。

注意：如果_fxChild调用任何其他执行委托调用的函数，则安全性可能会受到影响。

## 跨链库
除了 *CrossChainEnabled* 抽象之外，跨链意识还可以通过库来实现。这些库可以用于构建复杂的设计，例如具有与多个桥接器交互能力的合约。

### LibAMB
```
import "@openzeppelin/contracts/crosschain/amb/LibAMB.sol";
```
使用[AMB](https://docs.tokenbridge.net/amb-bridge/about-amb-bridge)桥接家族的跨链感知合约的原始代码。

**FUNCTIONS**
isCrossChain(bridge)
crossChainSender(bridge)

#### isCrossChain(address bridge) → bool
内部#
返回当前函数调用是否是通过桥Relayer传递的跨链消息的结果。

#### crossChainSender(address bridge) → address
内部#
返回通过桥触发当前跨链消息的发送者地址。

> NOTE
在尝试恢复发送者之前，应该检查*isCrossChain*，因为如果当前函数调用不是跨链消息的结果，它将返回NotCrossChainCall。

### LibArbitrumL1
```
import "@openzeppelin/contracts/crosschain/arbitrum/LibArbitrumL1.sol";
```
用于 [Arbitrum](https://arbitrum.io/) 跨链感知合约的原语。

此版本仅应在 L1 上使用，以处理来自 L2 的跨链消息。对于另一侧，请使用 *LibArbitrumL2*。

**FUNCTIONS**
isCrossChain(bridge)
crossChainSender(bridge)

#### isCrossChain(address bridge) → bool
内部#
返回当前函数调用是否是通过桥Relayer的跨链消息的结果。

#### crossChainSender(address bridge) → address
内部#
返回通过桥梁触发当前跨链消息的发送者地址。

> NOTE
在尝试恢复发送者之前，应该检查*isCrossChain*，因为如果当前函数调用不是跨链消息的结果，它将回滚并出现NotCrossChainCall。

### LibArbitrumL2
```
import "@openzeppelin/contracts/crosschain/arbitrum/LibArbitrumL2.sol";
```
用于 [Arbitrum](https://arbitrum.io/) 跨链智能合约的原语。

此版本仅用于 L2，用于处理来自 L1 的跨链消息。对于另一侧，请使用 *LibArbitrumL1*。

> WARNING
目前在Arbitrum中存在一个错误，当部署在代理后，该合约无法检测到跨链调用。这将在网络升级为Arbitrum Nitro时修复，目前计划于2022年8月31日进行。

**FUNCTIONS**
isCrossChain(arbsys)
crossChainSender(arbsys)

#### isCrossChain(address arbsys) → bool
内部#

#### crossChainSender(address arbsys) → address
内部#
返回通过arbsys触发当前跨链消息的发送者地址。

在尝试恢复发送者之前，应该先检查*isCrossChain*，如果当前函数调用不是跨链消息的结果，则会返回NotCrossChainCall。

### LibOptimism
```
import "@openzeppelin/contracts/crosschain/optimism/LibOptimism.sol";
```
面向[Optimism](https://www.optimism.io/)的跨链感知合约的原语。有关此处使用的[功能文档](https://community.optimism.io/docs/developers/bridge/messaging/#accessing-msg-sender)，请参阅文档。

**FUNCTIONS**
isCrossChain(messenger)
crossChainSender(messenger)

#### isCrossChain(address messenger) → bool
内部#
返回当前函数调用是否是由信使Relayer的跨链消息的结果。

#### crossChainSender(address messenger) → address
内部#
返回通过信使触发当前跨链消息的发送者地址。

> NOTE
在尝试恢复发送者之前，应先检查*isCrossChain*，因为如果当前函数调用不是跨链消息的结果，它将返回NotCrossChainCall。
