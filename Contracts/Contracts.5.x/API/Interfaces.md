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
```
import "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";
```

ERC3156 FlashBorrower接口的定义，如在[ERC-3156](https://eips.ethereum.org/EIPS/eip-3156)中定义。

**FUNCTIONS**
onFlashLoan(initiator, token, amount, fee, data)

#### onFlashLoan(address initiator, address token, uint256 amount, uint256 fee, bytes data) → bytes32
external#
接收闪电贷款。

#### [IERC4626](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC4626.sol)
```
import "@openzeppelin/contracts/interfaces/IERC4626.sol";
```

ERC4626 "代币化保险库标准"的接口，如[ERC-4626](https://eips.ethereum.org/EIPS/eip-4626)中定义。

**FUNCTIONS**
asset()

totalAssets()

convertToShares(assets)

convertToAssets(shares)

maxDeposit(receiver)

previewDeposit(assets)

deposit(assets, receiver)

maxMint(receiver)

previewMint(shares)

mint(shares, receiver)

maxWithdraw(owner)

previewWithdraw(assets)

withdraw(assets, receiver, owner)

maxRedeem(owner)

previewRedeem(shares)

redeem(shares, receiver, owner)

IERC20METADATA
name()

symbol()

decimals()

IERC20
totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

**EVENTS**
Deposit(sender, owner, assets, shares)

Withdraw(sender, receiver, owner, assets, shares)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### asset() → address assetTokenAddress
external#
返回用于Vault账户、存款和提款的底层代币的地址。

* 必须是ERC-20代币合约。

* 不得出现回滚。

#### totalAssets() → uint256 totalManagedAssets
external#
返回由Vault“管理”的基础资产的总额。

* 应包括来自收益的任何复利。

* 必须包括对Vault中资产收取的任何费用。

* 不得撤销。

#### convertToShares(uint256 assets) → uint256 shares
external#
返回保险库在理想情况下，满足所有条件时，会为提供的资产兑换的股份数量。

* 不得包含在保险库中对资产收取的任何费用。

* 不得根据调用者显示任何变化。

* 在实际进行交换时，不得反映滑点或其他链上条件。

* 不得撤销。

> NOTE
这个计算可能不反映“每用户”的每股价格，而应反映“平均用户”的每股价格，即平均用户在兑换时应期望看到的价格。

#### convertToAssets(uint256 shares) → uint256 assets
external#
返回保险库将为提供的股份交换的资产数量，在理想情况下，所有条件都满足的情况下。

* 不得包含对保险库中资产收取的任何费用。

* 不得显示取决于调用者的任何变化。

* 在执行实际交换时，不得反映滑点或其他链上条件。

* 不得撤销。

> NOTE
这个计算可能不反映“每用户”的每股价格，而应反映“平均用户”的每股价格，即平均用户在交换时应期望看到的价格。

#### maxDeposit(address receiver) → uint256 maxAssets
external#
返回接收者可以通过存款调用存入保险库的基础资产的最大金额。

* 如果接收者受到某些存款限制，必须返回一个限制值。

* 如果对可以存入的资产的最大金额没有限制，必须返回2 ** 256 - 1。

* 不得撤销。

#### previewDeposit(uint256 assets) → uint256 shares
external#
允许链上或链下用户在当前区块模拟他们的存款效果，考虑当前链上条件。

* 必须返回尽可能接近且不超过在同一交易中存款调用中会铸造的保险库份额的确切数量。即，如果在同一交易中调用，存款应返回与previewDeposit相同或更多的份额。

* 不必考虑maxDeposit返回的存款限制，应始终假设存款会被接受，无论用户是否有足够的代币批准等。

* 必须包含存款费用。集成商应注意存款费用的存在。

* 不得撤销。

> NOTE
convertToShares和previewDeposit之间的任何不利差异应被视为份额价格的滑点或其他类型的条件，这意味着存款人将通过存款损失资产。

#### deposit(uint256 assets, address receiver) → uint256 shares
external#
通过存入准确数量的基础代币，铸币厂将Vault股份分享给接收者。

* 必须发出存款事件。

* 可能支持额外的流程，其中底层代币在存款执行前由金库合约拥有，并在存款期间进行记账。

* 如果所有资产无法存入（由于达到存款限额，滑点，用户未向金库合约批准足够的底层代币等），则必须撤销。

> NOTE
大多数实现将需要预先批准金库与金库的底层资产代币。

#### maxMint(address receiver) → uint256 maxShares
external#
返回通过mint调用，可以为接收者铸造的Vault份额的最大数量。- 如果接收者受到某种铸币限制，必须返回一个有限的值。- 如果对可以铸造的份额数量没有限制，必须返回2 ** 256 - 1。- 不得还原。

#### previewMint(uint256 shares) → uint256 assets
external#
允许链上或链下用户在当前区块模拟他们的铸币效果，根据当前链上条件。

* 必须返回与铸币调用在同一交易中存入的资产数量尽可能接近且不少于确切的数量。也就是说，如果在同一交易中调用，mint应返回与previewMint相同或更少的资产。

* 不必考虑像maxMint返回的铸币限制，应始终行事，就像铸币会被接受一样，无论用户是否有足够的已批准的代币等。

* 必须包含存款费用。集成商应注意存款费用的存在。

* 不得撤销。

> NOTE
convertToAssets和previewMint之间的任何不利差异应被视为份额价格的滑点或其他类型的条件，意味着存款人将通过铸币损失资产。

#### mint(uint256 shares, address receiver) → uint256 assets
external#
通过存入底层代币，Mints会将Vault份额完全分享给接收者。

* 必须触发存款事件。

* 可能支持另一种流程，在这种流程中，底层代币在执行铸币之前由Vault合约拥有，并在铸币过程中进行记账。

* 如果所有的份额无法被铸造（由于达到存款限额，滑点，用户未向Vault合约批准足够的底层代币等），则必须回滚。

> NOTE
大多数实现将需要预先批准Vault与Vault的底层资产代币。

#### maxWithdraw(address owner) → uint256 maxAssets
external#
返回可以通过提款调用从Vault中的所有者余额中提取的基础资产的最大金额。

* 如果所有者受到某些提款限制或时间锁定的影响，必须返回一个限制值。

* 不得撤销。

#### previewWithdraw(uint256 assets) → uint256 shares
external#
允许链上或链下用户在当前区块模拟他们的提款效果，考虑当前链上条件。

* 必须返回与在同一交易中调用提款时燃烧的保险库份额尽可能接近且不少于确切数量。即，如果在同一交易中调用，提款应返回相同或更少的份额作为预览提款。

* 不必考虑像maxWithdraw返回的提款限制，应始终假设提款会被接受，无论用户是否有足够的份额等。

* 必须包含提款费用。集成者应注意提款费用的存在。

* 不得撤销。

> NOTE
convertToShares和previewWithdraw之间的任何不利差异应被视为份额价格的滑点或其他类型的条件，这意味着存款人将通过存款损失资产。

#### withdraw(uint256 assets, address receiver, address owner) → uint256 shares
external#
从所有者那里销毁股份，并将准确的底层代币资产发送给接收者。

* 必须发出提款事件。

* 可能支持另一种流程，即在执行提款之前，底层代币由Vault合约拥有，并在提款过程中进行记账。

* 如果所有资产都无法提取（由于达到提款限额、滑点、所有者没有足够的股份等），则必须撤销。

请注意，某些实现将需要在执行提款之前先向Vault发出请求。这些方法应该分别执行。

#### maxRedeem(address owner) → uint256 maxShares
external#
返回Vault中可以通过赎回调用从所有者余额中赎回的最大Vault份额。

* 如果所有者受到某些提款限制或时间锁定，必须返回一个限制值。

* 如果所有者不受任何提款限制或时间锁定的影响，必须返回balanceOf(owner)。

* 不得撤销。

#### previewRedeem(uint256 shares) → uint256 assets
external#
允许链上或链下用户在当前区块模拟他们的赎回效果，考虑到当前的链上条件。

* 必须返回尽可能接近且不超过在同一交易中赎回调用所提取的资产的确切数量。也就是说，如果在同一交易中调用，赎回应返回与previewRedeem相同或更多的资产。

* 不必考虑像maxRedeem返回的赎回限制，应始终假设赎回会被接受，无论用户是否拥有足够的份额等。
https://github.com/OpenZeppelin/openzeppelin-contracts/issues
* 必须包含提现费用。集成商应注意提现费用的存在。

* 不得撤销。

> NOTE
convertToAssets和previewRedeem之间的任何不利差异应被视为份额价格的滑点或其他类型的条件，这意味着赎回者将通过赎回而损失资产。

#### redeem(uint256 shares, address receiver, address owner) → uint256 assets
external#
精确地从所有者那里销毁股份，并将底层代币的资产发送给接收者。

* 必须发出提款事件。

* 可能支持一个额外的流程，在这个流程中，底层代币在赎回执行之前由Vault合约拥有，并在赎回过程中进行记账。

* 如果所有的股份无法赎回（由于达到提款限额、滑点、所有者没有足够的股份等），则必须回滚。

> NOTE
一些实现将需要在执行提款之前先向Vault发出预请求。这些方法应该分别执行。

#### Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares)
event#

#### Withdraw(address indexed sender, address indexed receiver, address indexed owner, uint256 assets, uint256 shares)
event#

### [IERC5313](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC5313.sol)
```
import "@openzeppelin/contracts/interfaces/IERC5313.sol";
```

为轻型合同所有权标准的接口。

一个标准化的最小接口，用于识别控制合同的账户。

**FUNCTIONS**
owner()

#### owner() → address
external
获取所有者的地址。

### [IERC5267](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC5267.sol)
```
import "@openzeppelin/contracts/interfaces/IERC5267.sol";
```

**FUNCTIONS**
eip712Domain()

#### eip712Domain() → bytes1 fields, string name, string version, uint256 chainId, address verifyingContract, bytes32 salt, uint256[] extensions
external#
返回用于描述此合约用于EIP-712签名的域分隔符的字段和值。

#### EIP712DomainChanged()
event#
可能会发出信号，表示域可能已经改变。

### [IERC5805](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC5805.sol)
```
import "@openzeppelin/contracts/interfaces/IERC5805.sol";
```

**FUNCTIONS**
IVOTES
getVotes(account)

getPastVotes(account, timepoint)

getPastTotalSupply(timepoint)

delegates(account)

delegate(delegatee)

delegateBySig(delegatee, nonce, expiry, v, r, s)

IERC6372
clock()

CLOCK_MODE()

**EVENTS**
IVOTES
DelegateChanged(delegator, fromDelegate, toDelegate)

DelegateVotesChanged(delegate, previousVotes, newVotes)

**ERRORS**
IVOTES
VotesExpiredSignature(expiry)

### [IERC6372](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/interfaces/IERC6372.sol)
```
import "@openzeppelin/contracts/interfaces/IERC6372.sol";
```

FUNCTIONS
clock()

CLOCK_MODE()

#### clock() → uint48
external#
用于标记检查点的时钟。可以被覆盖以实现基于时间戳的检查点（和投票）。

#### CLOCK_MODE() → string
external#
时钟的描述