# Meta Transactions

## Core

### [ERC2771Context](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/metatx/ERC2771Context.sol)
```
import "@openzeppelin/contracts/metatx/ERC2771Context.sol";
```

支持ERC2771的上下文变体。

> WARNING
在依赖于特定calldata长度的合约中避免使用这种模式，因为它们会受到任何根据ERC2771规范将from地址附加到msg.data的转发器的影响，将地址大小（20字节）添加到calldata大小中。一个意想不到的行为例子可能是在试图只调用如果msg.data.length == 0才能访问的receive函数时，无意中调用了fallback（或其他）函数。

**FUNCTIONS**
constructor(trustedForwarder_)

trustedForwarder()

isTrustedForwarder(forwarder)

_msgSender()

_msgData()

#### constructor(address trustedForwarder_)
internal#

#### trustedForwarder() → address
public#
返回可信转发器的地址。

#### isTrustedForwarder(address forwarder) → bool
public#
指示任何特定地址是否为可信转发器。

#### _msgSender() → address sender
internal#
覆盖msg.sender。当呼叫不是由受信任的转发器执行或者调用数据长度小于20字节（一个地址长度）时，默认为原始的msg.sender。

#### _msgData() → bytes
internal#
覆盖msg.data。当不是由受信任的转发器进行调用或者调用数据长度小于20字节（一个地址长度）时，默认为原始的msg.data。

## Utils

### [ERC2771Forwarder](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/metatx/ERC2771Forwarder.sol)
```
import "@openzeppelin/contracts/metatx/ERC2771Forwarder.sol";
```

这是一个与ERC2771合约兼容的转发器。请参阅*ERC2771Context*。

此转发器对包含以下内容的转发请求进行操作：

* from：代表某个地址进行操作。它必须等于请求签名者。

* to：应被调用的地址。

* value：附加到请求调用的本地代币的数量。

* gas：将与请求调用一起转发的gas限制的数量。

* nonce：一个独特的交易排序标识符，用于避免重放和请求无效。

* deadline：请求在此时间戳之后不再可执行。

* data：与请求调用一起发送的编码msg.data。

如果中继器正在处理大量请求，它们可以提交批次。在高吞吐量的情况下，中继器可能会遇到链的限制，例如mempool中的交易数量限制。在这些情况下，建议将负载分布在多个账户之间。

> NOTE
批量请求包括对未使用的msg.value的可选退款，这是通过执行带有空calldata的调用来实现的。虽然这在ERC-2771合规性的范围内，但如果退款接收者认为转发器是一个可信的转发器，它必须正确处理msg.data.length == 0。在OpenZeppelin Contracts 4.9.3版本之前的ERC2771Context没有正确处理这个问题。

### 安全考虑
如果中继器提交一个转发请求，它应该愿意支付请求中指定的gas数量的100%。这个合约没有实施任何形式的报复性gas，假设有一个频道的激励机制让中继器代表签名者支付执行费用。通常，中继器由一个项目操作，该项目将其视为用户获取成本。

通过提供支付gas，中继器有可能被攻击者利用这些gas用于与预期的频道激励不一致的其他目的。如果你操作一个中继器，考虑将目标合约和函数选择器列入白名单。当特别转发ERC-721或ERC-1155转移时，考虑拒绝使用数据字段，因为它可以用来执行任意代码。

FUNCTIONS
constructor(name)

verify(request)

execute(request)

executeBatch(requests, refundReceiver)

_validate(request)

_recoverForwardRequestSigner(request)

_execute(request, requireValidRequest)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

EVENTS
ExecutedForwardRequest(signer, nonce, success)

IERC5267
EIP712DomainChanged()

ERRORS
ERC2771ForwarderInvalidSigner(signer, from)

ERC2771ForwarderMismatchedValue(requestedValue, msgValue)

ERC2771ForwarderExpiredRequest(deadline)

ERC2771UntrustfulTarget(target, forwarder)

NONCES
InvalidAccountNonce(account, currentNonce)

#### constructor(string name)
public#
请查阅 *EIP712.constructor*.

#### verify(struct ERC2771Forwarder.ForwardRequestData request) → bool
public#
如果在当前区块时间戳下，请求对于提供的签名有效，则返回true。

当目标信任此转发器，请求尚未过期（未达到截止日期），并且签名者与签名请求的from参数匹配时，交易被认为是有效的。

> NOTE
请求可能在这里返回false，但如果提供了退款接收者，它不会导致*executeBatch*回滚。

#### execute(struct ERC2771Forwarder.ForwardRequestData request)
public#
根据ERC-2771协议，代表`签名`的签署者执行一个请求。提供给请求调用的gas可能不会完全等于请求的数量，但调用不会耗尽gas。如果请求无效或调用回滚，将会恢复，这种情况下，nonce不会被消耗。

要求：
* 请求值应等于提供的msg.value。

* 根据*verify*，请求应该是有效的。

#### executeBatch(struct ERC2771Forwarder.ForwardRequestData[] requests, address payable refundReceiver)
public#
批量执行版本，可选退款和原子*执行*。

如果批量请求中至少有一个无效请求（参见*验证*），则将跳过该请求，并在执行结束时将未使用的请求值退还给refundReceiver参数。这样做是为了防止当一个请求无效或已经提交时，整个批次都会回滚。

如果refundReceiver是address(0)，那么当至少有一个请求无效时，这个函数将回滚，而不是跳过它。这可能在需要原子执行批量请求（至少在顶层）时很有用。例如，如果中继器使用的服务避免包含已回滚的交易，可以选择不退款（从而不进行原子性）。

要求：
* 请求的值之和应等于提供的msg.value。

* 当refundReceiver是零地址时，所有请求都应有效（参见*验证*）。

> NOTE
设置零退款接收者只保证第一级转发调用的全或无请求执行。如果转发请求调用到另一个带有子调用的合约，第二级调用可能会回滚，而顶级调用不会回滚。

#### _validate(struct ERC2771Forwarder.ForwardRequestData request) → bool isTrustedForwarder, bool active, bool signerMatch, address signer
internal#
验证是否可以在当前区块时间戳下，以请求签名者的身份执行提供的请求，这取决于给定的请求签名。

#### _recoverForwardRequestSigner(struct ERC2771Forwarder.ForwardRequestData request) → bool, address
internal#
返回一个元组，其中包含恢复的EIP712转发请求消息哈希的签名者和一个布尔值，该布尔值表示签名是否有效。

> NOTE
如果*ECDSA.tryRecover*表示没有恢复错误，那么签名就被认为是有效的。

#### _execute(struct ERC2771Forwarder.ForwardRequestData request, bool requireValidRequest) → bool success
internal#
返回一个元组，其中包含恢复的EIP712转发请求消息哈希的签名者和一个布尔值，该布尔值表示签名是否有效。

#### _execute(struct ERC2771Forwarder.ForwardRequestData request, bool requireValidRequest) → bool success
internal#
验证并执行一个签名请求，返回请求调用的成功值。

这是一个内部函数，不验证msg.value。

要求：
* 调用者必须提供足够的gas来进行调用。

* 如果requireValidRequest为真，请求必须是有效的（参见*verify*）。

触发一个*ExecutedForwardRequest*事件。

> IMPORTANT
使用这个函数不会检查所有的msg.value是否已经发送，可能会导致值被困在合约中。

#### ExecutedForwardRequest(address indexed signer, uint256 nonce, bool success)
event#
当执行ForwardRequest时发出。

> NOTE
不成功的转发请求可能是由于无效的签名、过期的截止日期，或者简单地在请求的调用中回滚。该合约保证中继器无法强制请求的调用耗尽gas。

#### ERC2771ForwarderInvalidSigner(address signer, address from)
error#
来自的请求与恢复的签名者不匹配。

#### ERC2771ForwarderMismatchedValue(uint256 requestedValue, uint256 msgValue)
error#
请求的值与可用的消息值不匹配。

#### ERC2771ForwarderExpiredRequest(uint48 deadline)
error#
请求截止日期已过期。

#### ERC2771UntrustfulTarget(address target, address forwarder)
error#
请求目标不信任转发者。