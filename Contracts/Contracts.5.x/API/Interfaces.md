# Interfaces

## 标准化接口列表

这些接口以.sol文件的形式提供，也可以通过npm包作为编译器.json ABI文件提供。这些对于与实现它们的第三方合约进行交互非常有用。

* IERC20

* IERC20Errors

* IERC20Metadata

* IERC165

* IERC721

* IERC721Receiver

* IERC721Enumerable

* IERC721Metadata

* IERC721Errors

* IERC777

* IERC777Recipient

* IERC777Sender

* IERC1155

* IERC1155Receiver

* IERC1155MetadataURI

* IERC1155Errors

* IERC1271

* IERC1363

* IERC1363Receiver

* IERC1363Spender

* IERC1820Implementer

* IERC1820Registry

* IERC1822Proxiable

* IERC2612

* IERC2981

* IERC3156FlashLender

* IERC3156FlashBorrower

* IERC4626

* IERC4906

* IERC5267

* IERC5313

* IERC5805

* IERC6372

## 详细的ABI

### [IERC20Errors](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/draft-IERC6093.sol)
```
import "@openzeppelin/contracts/interfaces/draft-IERC6093.sol";
```

标准ERC20错误接口是[ERC-6093](https://eips.ethereum.org/EIPS/eip-6093)的自定义错误，用于ERC20代币。

**ERRORS**
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### ERC20InsufficientBalance(address sender, uint256 balance, uint256 needed)
error#
表示与发送者当前余额相关的错误。用于转账。

#### ERC20InvalidSender(address sender)
error#
表示令牌发送者出现故障。用于转账。

#### ERC20InvalidReceiver(address receiver)
error#
表示令牌接收器出现故障。用于转账。

#### ERC20InsufficientAllowance(address spender, uint256 allowance, uint256 needed)
error#
表示花费者的津贴出现故障。用于转账。

#### ERC20InvalidApprover(address approver)
error#
表示批准令牌的批准者出现故障。用于审批。

#### ERC20InvalidSpender(address spender)
error#
表示批准的支出者出现故障。用于批准。

### [IERC721Errors](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/draft-IERC6093.sol)
```
import "@openzeppelin/contracts/interfaces/draft-IERC6093.sol";
```

标准ERC721错误接口的[ERC-6093](https://eips.ethereum.org/EIPS/eip-6093)自定义错误，用于ERC721代币。

**ERRORS**
ERC721InvalidOwner(owner)

ERC721NonexistentToken(tokenId)

ERC721IncorrectOwner(sender, tokenId, owner)

ERC721InvalidSender(sender)

ERC721InvalidReceiver(receiver)

ERC721InsufficientApproval(operator, tokenId)

ERC721InvalidApprover(approver)

ERC721InvalidOperator(operator)

#### ERC721InvalidOwner(address owner)
error#
表示一个地址不能是所有者。例如，在EIP-20中，address(0)是被禁止的所有者。在余额查询中使用。

#### ERC721NonexistentToken(uint256 tokenId)
error#
表示其所有者为零地址的tokenId。

#### ERC721IncorrectOwner(address sender, uint256 tokenId, address owner)
error#
表示与特定令牌的所有权相关的错误。用于转账。

#### ERC721InvalidSender(address sender)
error#
表示令牌发送者出现故障。用于转账。

#### ERC721InvalidReceiver(address receiver)
error#
表示令牌接收器出现故障。用于转账。

#### ERC721InsufficientApproval(address operator, uint256 tokenId)
error#
表示操作员批准的失败。用于转账。

#### ERC721InvalidApprover(address approver)
error#
表示批准令牌的批准者出现故障。用于批准中。

#### ERC721InvalidOperator(address operator)
error#
表示操作员未获批准的失败。用于批准中。

### [IERC1155Errors](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/draft-IERC6093.sol)
```
import "@openzeppelin/contracts/interfaces/draft-IERC6093.sol";
```

标准ERC1155错误接口是[ERC-6093](https://eips.ethereum.org/EIPS/eip-6093)为ERC1155代币定制的错误。

**ERRORS**
ERC1155InsufficientBalance(sender, balance, needed, tokenId)

ERC1155InvalidSender(sender)

ERC1155InvalidReceiver(receiver)

ERC1155MissingApprovalForAll(operator, owner)

ERC1155InvalidApprover(approver)

ERC1155InvalidOperator(operator)

ERC1155InvalidArrayLength(idsLength, valuesLength)

#### ERC1155InsufficientBalance(address sender, uint256 balance, uint256 needed, uint256 tokenId)
error#
表示与发送者当前余额相关的错误。用于转账。

#### ERC1155InvalidSender(address sender)
error#
表示令牌发送者出现故障。用于转账。

#### ERC1155InvalidReceiver(address receiver)
error#
表示令牌接收器出现故障。用于转账。

#### ERC1155MissingApprovalForAll(address operator, address owner)
error#
表示操作员批准的失败。用于转账。

#### ERC1155InvalidApprover(address approver)
error#
表示批准令牌的批准者出现故障。用于批准。

#### ERC1155InvalidOperator(address operator)
error#
表示操作员未获批准的失败。用于批准。

#### ERC1155InvalidArrayLength(uint256 idsLength, uint256 valuesLength)
error#
表示在safeBatchTransferFrom操作中，ids和values之间的数组长度不匹配。用于批量转移。

### [IERC1271](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1271.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1271.sol";
```

ERC1271标准签名验证方法的合约接口，如在[ERC-1271](https://eips.ethereum.org/EIPS/eip-1271)中定义。

**FUNCTIONS**
isValidSignature(hash, signature)

#### isValidSignature(bytes32 hash, bytes signature) → bytes4 magicValue
external#
应返回提供的签名是否适用于提供的数据

### [IERC1363](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1363.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1363.sol";
```
定义了一个ERC20代币的接口，该接口支持在transfer或transferFrom后执行接收者代码，或在approve后执行支出者代码。

### [IERC1363](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1363.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1363.sol";
```

符合ERC1363协议的接口，在[EIP](https://eips.ethereum.org/EIPS/eip-1363)中定义。
定义了一个ERC20代币的接口，该接口支持在transfer或transferFrom后执行接收者代码，或在approve后执行花费者代码。

**FUNCTIONS**
transferAndCall(to, amount)

transferAndCall(to, amount, data)

transferFromAndCall(from, to, amount)

transferFromAndCall(from, to, amount, data)

approveAndCall(spender, amount)

approveAndCall(spender, amount, data)

IERC20
totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

IERC165
supportsInterface(interfaceId)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### transferAndCall(address to, uint256 amount) → bool
external#
将代币从msg.sender转移到另一个地址，然后在接收者上调用onTransferReceived

#### transferAndCall(address to, uint256 amount, bytes data) → bool
external#
将代币从msg.sender转移到另一个地址，然后在接收者上调用onTransferReceived

#### transferFromAndCall(address from, address to, uint256 amount) → bool
external#
将代币从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived

#### transferFromAndCall(address from, address to, uint256 amount, bytes data) → bool
external#
将代币从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived

#### approveAndCall(address spender, uint256 amount) → bool
external#
批准通过的地址代表msg.sender花费指定数量的代币，然后在spender上调用onApprovalReceived。

#### approveAndCall(address spender, uint256 amount, bytes data) → bool
external#
批准通过的地址代表msg.sender花费指定数量的代币，然后在spender上调用onApprovalReceived。

### [IERC1363Receiver](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1363Receiver.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1363Receiver.sol";
```

为任何希望支持来自{ERC1363}代币合约的*IERC1363.transferAndCall*或*IERC1363.transferFromAndCall*的合约提供的接口。

**FUNCTIONS**
onTransferReceived(operator, from, amount, data)

#### onTransferReceived(address operator, address from, uint256 amount, bytes data) → bytes4
external#
任何ERC1363智能合约在转账或转账后都会在接收者上调用此函数。此函数可能会抛出以撤销和拒绝转账。返回的值除了魔术值以外，必须导致交易被撤销。注意：代币合约地址总是消息的发送者。

#### [IERC1363Spender](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1363Spender.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1363Spender.sol";
```

用于支持来自{ERC1363}代币合约的*IERC1363.approveAndCall*的任何合约的接口。

**FUNCTIONS**
onApprovalReceived(owner, amount, data)

#### onApprovalReceived(address owner, uint256 amount, bytes data) → bytes4
external#
在获得批准后，任何ERC1363智能合约都会在接收者上调用此函数。此函数可能会抛出以撤销和拒绝批准。返回的值除了魔术值以外，必须导致交易被撤销。注意：代币合约地址始终是消息发送者。

### [IERC1820Implementer](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1820Implementer.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1820Implementer.sol";
```

用于ERC1820实现者的接口，根据[EIP](https://eips.ethereum.org/EIPS/eip-1820#interface-implementation-erc1820implementerinterface)定义。由将在*IERC1820Registry*中注册为实现者的合约使用。

**FUNCTIONS**
canImplementInterfaceForAddress(interfaceHash, account)

#### canImplementInterfaceForAddress(bytes32 interfaceHash, address account) → bytes32
external#
如果此合约为账户实现了interfaceHash，返回一个特殊值（ERC1820_ACCEPT_MAGIC）。

参见*IERC1820Registry.setInterfaceImplementer*。

### [IERC1820Registry](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC1820Registry.sol)
```
import "@openzeppelin/contracts/interfaces/IERC1820Registry.sol";
```

全球ERC1820注册表的接口，根据[EIP](https://eips.ethereum.org/EIPS/eip-1820)定义。账户可以在此注册表中注册接口的实施者，也可以查询支持。

实施者可以由多个账户共享，并且每个账户也可以实施多个接口。合同可以为自己实施接口，但是外部拥有的账户（EOA）必须将此委托给合同。

还可以通过注册表查询*IERC165*接口。

有关深入解释和源代码分析，请参阅EIP文本。

**FUNCTIONS**
setManager(account, newManager)

getManager(account)

setInterfaceImplementer(account, _interfaceHash, implementer)

getInterfaceImplementer(account, _interfaceHash)

interfaceHash(interfaceName)

updateERC165Cache(account, interfaceId)

implementsERC165Interface(account, interfaceId)

implementsERC165InterfaceNoCache(account, interfaceId)

**EVENTS**
InterfaceImplementerSet(account, interfaceHash, implementer)

ManagerChanged(account, newManager)

#### setManager(address account, address newManager)
external#
将newManager设置为账户的管理者。账户的管理者能够为其设置接口实现者。

默认情况下，每个账户都是其自身的管理者。在newManager中传递一个0x0的值将会重置管理者到这个初始状态。

触发一个*ManagerChanged*事件。

要求：
* 调用者必须是账户的当前管理者。

#### getManager(address account) → address
external#
返回账户的管理员。

参见*setManager*。

#### setInterfaceImplementer(address account, bytes32 _interfaceHash, address implementer)
external#
将实现者合约设置为账户的接口哈希的实现者。

零地址的账户被视为调用者的地址。零地址也可以在实现者中使用，以删除旧的实现者。

参见*接口哈希*以了解如何创建这些。

触发一个*InterfaceImplementerSet*事件。

要求：
* 调用者必须是账户的当前管理者。

* 接口哈希不能是*IERC165*接口id（即，它不能以28个零结尾）。

* 实现者必须实现*IERC1820Implementer*，并在查询支持时返回true，除非实现者是调用者。参见*IERC1820Implementer.canImplementInterfaceForAddress*。

#### getInterfaceImplementer(address account, bytes32 _interfaceHash) → address
external#
返回账户的interfaceHash的实现者。如果没有注册此类实现者，则返回零地址。

如果interfaceHash是一个*IERC165*接口id（即，它以28个零结尾），则会查询账户是否支持它。

账户是零地址是调用者地址的别名。

#### interfaceHash(string interfaceName) → bytes32
external#
返回一个接口名的接口哈希，如[EIP相应部分所定义](https://eips.ethereum.org/EIPS/eip-1820#interface-name)。

#### updateERC165Cache(address account, bytes4 interfaceId)
external#

#### implementsERC165Interface(address account, bytes4 interfaceId) → bool
external#

#### implementsERC165InterfaceNoCache(address account, bytes4 interfaceId) → bool
external#

#### InterfaceImplementerSet(address indexed account, bytes32 indexed interfaceHash, address indexed implementer)
event#

#### ManagerChanged(address indexed account, address indexed newManager)
event#

### [IERC1822Proxiable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/draft-IERC1822.sol)
```
import "@openzeppelin/contracts/interfaces/draft-IERC1822.sol";
```

ERC1822：通用可升级代理标准（UUPS）记录了一种通过简化代理进行可升级性的方法，其升级完全由当前实现控制。

**FUNCTIONS**
proxiableUUID()

#### proxiableUUID() → bytes32
external#
返回可代理合约假定用于存储实现地址的存储槽。

> IMPORTANT
指向可代理合约的代理本身不应被视为可代理，因为这有可能使升级到它的代理变砖，通过代理到自身直到耗尽gas。因此，如果通过代理调用此函数，它必须回滚。

### [IERC2612](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC2612.sol)
```
import "@openzeppelin/contracts/interfaces/IERC2612.sol";
```

**FUNCTIONS**
IERC20PERMIT
permit(owner, spender, value, deadline, v, r, s)

nonces(owner)

DOMAIN_SEPARATOR()

### [IERC2981](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC2981.sol)
```
import "@openzeppelin/contracts/interfaces/IERC2981.sol";
```

NFT版税标准的接口。

这是一种标准化的方式，用于获取非同质化代币（NFT）的版税支付信息，以实现所有NFT市场和生态系统参与者的版税支付的通用支持。

**FUNCTIONS**
royaltyInfo(tokenId, salePrice)

IERC165
supportsInterface(interfaceId)

#### royaltyInfo(uint256 tokenId, uint256 salePrice) → address receiver, uint256 royaltyAmount
external#
返回应支付多少版税以及给谁，基于可能以任何交易单位计价的销售价格。版税金额以该交易单位计价，并应以该交易单位支付。

### [IERC3156FlashLender](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC3156FlashLender.sol)
```
import "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
```

ERC3156 FlashLender的接口，如*ERC-3156*中定义。

**FUNCTIONS**
maxFlashLoan(token)

flashFee(token, amount)

flashLoan(receiver, token, amount, data)

#### maxFlashLoan(address token) → uint256
external#
可供借贷的货币数量。

#### flashFee(address token, uint256 amount) → uint256
external#
对给定贷款所需收取的费用。

#### flashLoan(contract IERC3156FlashBorrower receiver, address token, uint256 amount, bytes data) → bool
external#
启动闪电贷款。

### [IERC3156FlashBorrower](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC3156FlashBorrower.sol)