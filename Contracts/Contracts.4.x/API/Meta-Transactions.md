# Meta Transactions
 
## 核心

### ERC2771Context
```
import "@openzeppelin/contracts/metatx/ERC2771Context.sol";
```

支持ERC2771的上下文变体。

支持ERC2771的上下文变体。

**FUNCTIONS**
[constructor(trustedForwarder)](#constructoraddress-trustedforwarder)
[isTrustedForwarder(forwarder)](#istrustedforwarderaddress-forwarder-→-bool)
[_msgSender()](#_msgsender-→-address-sender)
[_msgData()](#_msgdata-→-bytes)

#### constructor(address trustedForwarder)
internal#

#### isTrustedForwarder(address forwarder) → bool
公开#

#### _msgSender() → address sender
internal#

#### _msgData() → bytes
internal#

## 实用程序

### MinimalForwarder
```
import "@openzeppelin/contracts/metatx/MinimalForwarder.sol";
```

简单的最小转发器，可与ERC2771兼容的合约一起使用。请参阅[ERC2771Context](#erc2771context)。

MinimalForwarder主要用于测试，因为它缺少成为一个良好的生产就绪转发器所需的功能。该合约不打算具备用于稳定的转发系统所需的全部属性。一个具备良好特性的完全功能的转发系统需要更多的复杂性。我们建议你查看其他项目，如GSN，他们的目标是构建一个这样的系统。

**FUNCTIONS**

[constructor()](#constructor)
[getNonce(from)](#getnonceaddress-from-→-uint256)
[verify(req, signature)](#verifystruct-minimalforwarderforwardrequest-req-bytes-signature-→-bool)
[execute(req, signature)](#executestruct-minimalforwarderforwardrequest-req-bytes-signature-→-bool-bytes)

EIP712
[_domainSeparatorV4()](./Utils.md#_domainseparatorv4-→-bytes32)
[_hashTypedDataV4(structHash)](./Utils.md#_hashtypeddatav4bytes32-structhash-→-bytes32)
[eip712Domain()](./Utils.md#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**

IERC5267
[EIP712DomainChanged()](./Interfaces.md#eip712domainchanged)

#### constructor()
公开#

#### getNonce(address from) → uint256
公开#

#### verify(struct MinimalForwarder.ForwardRequest req, bytes signature) → bool
公开#

#### execute(struct MinimalForwarder.ForwardRequest req, bytes signature) → bool, bytes
公开#
