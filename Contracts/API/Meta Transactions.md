# Meta Transactions
 
## 核心

### ERC2771Context
```
import "@openzeppelin/contracts/metatx/ERC2771Context.sol";
```
支持ERC2771的上下文变体。

支持ERC2771的上下文变体。

**FUNCTIONS**
constructor(trustedForwarder)
isTrustedForwarder(forwarder)
_msgSender()
_msgData()

#### constructor(address trustedForwarder)
内部#

#### isTrustedForwarder(address forwarder) → bool
公开#

#### _msgSender() → address sender
内部#

#### _msgData() → bytes
内部#

## 实用程序

### MinimalForwarder
```
import "@openzeppelin/contracts/metatx/MinimalForwarder.sol";
```
简单的最小转发器，可与ERC2771兼容的合约一起使用。请参阅*ERC2771Context*。

MinimalForwarder主要用于测试，因为它缺少成为一个良好的生产就绪转发器所需的功能。该合约不打算具备用于稳定的转发系统所需的全部属性。一个具备良好特性的完全功能的转发系统需要更多的复杂性。我们建议您查看其他项目，如GSN，他们的目标是构建一个这样的系统。

**FUNCTIONS**

constructor()
getNonce(from)
verify(req, signature)
execute(req, signature)

EIP712
_domainSeparatorV4()
_hashTypedDataV4(structHash)
eip712Domain()

**EVENTS**

IERC5267
EIP712DomainChanged()

#### constructor()
公开#

#### getNonce(address from) → uint256
公开#

#### verify(struct MinimalForwarder.ForwardRequest req, bytes signature) → bool
公开#

#### execute(struct MinimalForwarder.ForwardRequest req, bytes signature) → bool, bytes
公开#
