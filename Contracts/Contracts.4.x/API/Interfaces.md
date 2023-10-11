# Interfaces

## List of standardized interfaces
这些接口可作为.sol文件和编译器.json ABI文件（通过npm包）提供。它们对于与实现这些接口的第三方合约进行交互非常有用。
* [IERC20](./ERC20.md#ierc20)

* [IERC20Metadata](./ERC20.md#ierc20metadata)

* [IERC165](./Utils.md#ierc165)

* [IERC721](./ERC721.md#ierc721)

* [IERC721Receiver](./ERC721.md#ierc721receiver)

* [IERC721Enumerable](./ERC721.md#ierc721enumerable)

* [IERC721Metadata](./ERC721.md#ierc721metadata)

* [IERC777](./ERC777.md#ierc777)

* [IERC777Recipient](./ERC777.md#ierc777recipient)

* [IERC777Sender](./ERC777.md#ierc777sender)

* [IERC1155](./ERC1155.md#ierc1155)

* [IERC1155Receiver](./ERC1155.md#ierc1155receiver)

* [IERC1155MetadataURI](./ERC1155.md#ierc1155metadatauri)

* [IERC1271](./Interfaces.md#ierc1271)

* [IERC1363](./Interfaces.md#ierc1363)

* [IERC1363Receiver](./Interfaces.md#ierc1363receiver)

* [IERC1363Spender](./Interfaces.md#ierc1363spender)

* [IERC1820Implementer](./Utils.md#ierc1820implementer)

* [IERC1820Registry](./Utils.md#ierc1820registry)

* [IERC1822Proxiable](./Interfaces.md#ierc1822proxiable)

* [IERC2612](./Interfaces.md#ierc2612)

* [IERC2981](./Interfaces.md#ierc2981)

* [IERC3156FlashLender](./Interfaces.md#ierc3156flashlender)

* [IERC3156FlashBorrower](./Interfaces.md#ierc3156flashborrower)

* [IERC4626](./Interfaces.md#ierc4626)

* [IERC4906](./Interfaces.md)

* [IERC5267](./Interfaces.md#ierc5267)

* [IERC5313](./Interfaces.md#ierc5313)

* [IERC5805](./Interfaces.md#ierc5805)

* [IERC6372](./Interfaces.md#ierc6372)

## 详细说明 ABI

### IERC1271
```
import "@openzeppelin/contracts/interfaces/IERC1271.sol";
```

[ERC1271](https://eips.ethereum.org/EIPS/eip-1271)中定义的合约的标准签名验证方法的接口。

*自v4.1版本开始可用。*

### IERC1363
```
import "@openzeppelin/contracts/interfaces/IERC1363.sol";
```

ERC1363兼容合约的接口，如[EIP](https://eips.ethereum.org/EIPS/eip-1363)所定义的。

定义了一个支持在转账或transferFrom之后执行接收者代码，或在approve之后执行spender代码的ERC20代币接口。

**FUNCTIONS**
[transferAndCall(to, amount)](#transferandcalladdress-to-uint256-amount-→-bool)
[transferAndCall(to, amount, data)](#transferandcalladdress-to-uint256-amount-bytes-data-→-bool)
[transferFromAndCall(from, to, amount)](#transferfromandcalladdress-from-address-to-uint256-amount-→-bool)
[transferFromAndCall(from, to, amount, data)](#transferfromandcalladdress-from-address-to-uint256-amount-bytes-data-→-bool)
[approveAndCall(spender, amount)](#approveandcalladdress-spender-uint256-amount-→-bool)
[approveAndCall(spender, amount, data)](#approveandcalladdress-spender-uint256-amount-bytes-data-→-bool)

IERC20
[totalSupply()](./ERC20.md#totalsupply-→-uint256)
[balanceOf(account)](./ERC20.md#balanceofaddress-account-→-uint256)
[transfer(to, amount)](./ERC20.md#transferaddress-to-uint256-amount-→-bool)
[allowance(owner, spender)](./ERC20.md#allowanceaddress-owner-address-spender-→-uint256)
[approve(spender, amount)](./ERC20.md#approveaddress-spender-uint256-amount-→-bool)
[transferFrom(from, to, amount)](./ERC20.md#transferfromaddress-from-address-to-uint256-amount-→-bool)

IERC165
[supportsInterface(interfaceId)](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)

**EVENTS**

IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-indexed-from-address-indexed-to-uint256-value)
[Approval(owner, spender, value)](./ERC20.md#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)

#### transferAndCall(address to, uint256 amount) → bool
外部#
从msg.sender地址转移代币到另一个地址，然后调用接收方的onTransferReceived函数。

#### transferAndCall(address to, uint256 amount, bytes data) → bool
外部#
从msg.sender地址将代币转移到另一个地址，然后调用接收者的onTransferReceived方法。

#### transferFromAndCall(address from, address to, uint256 amount) → bool
外部#
将代币从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived函数。

#### transferFromAndCall(address from, address to, uint256 amount, bytes data) → bool
外部#
将代币从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived函数。

#### approveAndCall(address spender, uint256 amount) → bool
外部#
批准通过地址代表msg.sender花费指定数量的代币，并调用spender的onApprovalReceived函数。

#### approveAndCall(address spender, uint256 amount, bytes data) → bool
外部#
批准传递的地址代表msg.sender花费指定数量的代币，并在spender上调用onApprovalReceived。

### IERC1363Receiver
```
import "@openzeppelin/contracts/interfaces/IERC1363Receiver.sol";
```

任何希望支持 [IERC1363.transferAndCall](#transferandcalladdress-to-uint256-amount-bytes-data-→-bool) 或 [IERC1363.transferFromAndCall](#transferfromandcalladdress-from-address-to-uint256-amount-bytes-data-→-bool) 的合约的接口，来自 {ERC1363} 代币合约。

**FUNCTIONS**
[onTransferReceived(operator, from, amount, data)](#ontransferreceivedaddress-operator-address-from-uint256-amount-bytes-data-→-bytes4)

#### onTransferReceived(address operator, address from, uint256 amount, bytes data) → bytes4
外部#
在转账或从智能合约中转账后，任何ERC1363智能合约都会调用此函数。此函数可能会抛出错误并拒绝转账。返回值如果不是预定的特殊值，必须导致交易被撤销。注意：代币合约地址始终为消息发送者。

#### IERC1363Spender
```
import "@openzeppelin/contracts/interfaces/IERC1363Spender.sol";
```

任何合约希望支持来自{ERC1363}代币合约的[IERC1363.approveAndCall](#approveandcalladdress-spender-uint256-amount-bytes-data-→-bool)的接口。

**FUNCTIONS**
[onApprovalReceived(owner, amount, data)](#onapprovalreceivedaddress-owner-uint256-amount-bytes-data-→-bytes4)

#### onApprovalReceived(address owner, uint256 amount, bytes data) → bytes4
外部#
在approve之后，任何ERC1363智能合约都会调用此函数来操作接收者。该函数可能会抛出异常以回滚并拒绝授权。返回值若非特定的魔术值，必须导致交易被回滚。注意：代币合约地址始终为消息发送者。

### IERC1822Proxiable
```
import "@openzeppelin/contracts/interfaces/draft-IERC1822.sol";
```

ERC1822: 通用可升级代理标准（UUPS）记录了一种通过简化代理实现完全由当前实现控制的升级能力的方法。

**FUNCTIONS**
[proxiableUUID()](#proxiableuuid-→-bytes32)

#### proxiableUUID() → bytes32
外部#
返回可代理合约假定用于存储实现地址的存储槽。

> IMPORTANT
指向可代理合约的代理本身不应被视为可代理的，因为这会导致代理升级到自身并耗尽燃料而破坏代理。因此，如果通过代理调用此函数，关键是使其回滚。

### IERC2612
```
import "@openzeppelin/contracts/interfaces/IERC2612.sol";
```

**FUNCTIONS**

IERC20PERMIT
[permit(owner, spender, value, deadline, v, r, s)](./ERC20.md#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)
[nonces(owner)](./ERC20.md#noncesaddress-owner-→-uint256)
[DOMAIN_SEPARATOR()](./ERC20.md#domain_separator-→-bytes32)

### IERC2981
```
import "@openzeppelin/contracts/interfaces/IERC2981.sol";
```

NFT版税标准的接口。

这是一种标准化的方式，用于检索非同质化代币（NFT）的版税支付信息，以实现在所有NFT市场和生态系统参与者之间普遍支持版税支付。

*自版本4.5起可用。*

**FUNCTIONS**
[royaltyInfo(tokenId, salePrice)](#royaltyinfouint256-tokenid-uint256-saleprice-→-address-receiver-uint256-royaltyamount)

IERC165
[supportsInterface(interfaceId)](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)

### royaltyInfo(uint256 tokenId, uint256 salePrice) → address receiver, uint256 royaltyAmount
外部#
根据可能以任何货币单位表示的销售价格，返回应支付的版税金额以及版税支付对象。版税金额应以相同的货币单位支付。

### IERC3156FlashLender
```
import "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
```

[ERC-3156](https://eips.ethereum.org/EIPS/eip-3156)定义的ERC3156 FlashLender的接口。

*从v4.1版本开始可用。*

**FUNCTIONS**
[maxFlashLoan(token)](#maxflashloanaddress-token-→-uint256)
[flashFee(token, amount)](#flashfeeaddress-token-uint256-amount-→-uint256)
[flashLoan(receiver, token, amount, data)](#flashloancontract-ierc3156flashborrower-receiver-address-token-uint256-amount-bytes-data-→-bool)

#### maxFlashLoan(address token) → uint256
外部#
可供借出的货币数量。

#### flashFee(address token, uint256 amount) → uint256
外部#
给定贷款的费用。

#### flashLoan(contract IERC3156FlashBorrower receiver, address token, uint256 amount, bytes data) → bool
外部#
发起闪电贷。

### IERC3156FlashBorrower
```
import "@openzeppelin/contracts/interfaces/IERC3156FlashBorrower.sol";
```

[ERC-3156](https://eips.ethereum.org/EIPS/eip-3156)定义的ERC3156 FlashBorrower的接口。

*从v4.1版本开始可用。*

**FUNCTIONS**
[onFlashLoan(initiator, token, amount, fee, data)](#onflashloanaddress-initiator-address-token-uint256-amount-uint256-fee-bytes-data-→-bytes32)

#### onFlashLoan(address initiator, address token, uint256 amount, uint256 fee, bytes data) → bytes32
外部#
接收一笔闪电贷款。

### IERC4626
```
import "@openzeppelin/contracts/interfaces/IERC4626.sol";
```
ERC4626 "代币化保险库标准"的接口，如[ERC-4626](https://eips.ethereum.org/EIPS/eip-4626)中定义。

*自v4.7版本起可用。*

**FUNCTIONS**
[asset()](#asset-→-address-assettokenaddress)
[totalAssets()](#totalassets-→-uint256-totalmanagedassets)
[convertToShares(assets)](#converttosharesuint256-assets-→-uint256-shares)
[convertToAssets(shares)](#converttoassetsuint256-shares-→-uint256-assets)
[maxDeposit(receiver)](#maxdepositaddress-receiver-→-uint256-maxassets)
[previewDeposit(assets)](#previewdeposituint256-assets-→-uint256-shares)
[deposit(assets, receiver)](#deposituint256-assets-address-receiver-→-uint256-shares)
[maxMint(receiver)](#maxmintaddress-receiver-→-uint256-maxshares)
[previewMint(shares)](#previewmintuint256-shares-→-uint256-assets)
[mint(shares, receiver)](#mintuint256-shares-address-receiver-→-uint256-assets)
[maxWithdraw(owner)](#maxwithdrawaddress-owner-→-uint256-maxassets)
[previewWithdraw(assets)](#previewwithdrawuint256-assets-→-uint256-shares)
[withdraw(assets, receiver, owner)](#withdrawuint256-assets-address-receiver-address-owner-→-uint256-shares)
[maxRedeem(owner)](#maxredeemaddress-owner-→-uint256-maxshares)
[previewRedeem(shares)](#previewredeemuint256-shares-→-uint256-assets)
[redeem(shares, receiver, owner)](#redeemuint256-shares-address-receiver-address-owner-→-uint256-assets)

IERC20METADATA

[name()](./ERC20.md#name-→-string)
[symbol()](./ERC20.md#symbol-→-string)
[decimals()](./ERC20.md#decimals-→-uint8)

IERC20
[totalSupply()](./ERC20.md#totalsupply-→-uint256)
[balanceOf(account)](./ERC20.md#balanceofaddress-account-→-uint256)
[transfer(to, amount)](./ERC20.md#transferaddress-to-uint256-amount-→-bool)
[allowance(owner, spender)](./ERC20.md#allowanceaddress-owner-address-spender-→-uint256)
[approve(spender, amount)](./ERC20.md#approveaddress-spender-uint256-amount-→-bool)
[transferFrom(from, to, amount)](./ERC20.md#transferfromaddress-from-address-to-uint256-amount-→-bool)

**EVENTS**
[Deposit(sender, owner, assets, shares)](#depositaddress-indexed-sender-address-indexed-owner-uint256-assets-uint256-shares)
[Withdraw(sender, receiver, owner, assets, shares)](#withdrawaddress-indexed-sender-address-indexed-receiver-address-indexed-owner-uint256-assets-uint256-shares)

IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-indexed-from-address-indexed-to-uint256-value)
[Approval(owner, spender, value)](./ERC20.md#approvaladdress-indexed-owner-address-indexed-spender-uint256-value)

#### asset() → address assetTokenAddress
外部#
返回用于Vault的账户、存款和提款的底层代币的地址。

* 必须是一个ERC-20代币合约。
* 不得回滚。

#### totalAssets() → uint256 totalManagedAssets
外部#
返回Vault管理的基础资产的总金额。

* 应包括从收益中产生的任何复利。
* 必须包括对Vault中的资产收取的任何费用。
* 不能回滚。

#### convertToShares(uint256 assets) → uint256 shares
外部#
在理想情况下，返回Vault将为提供的资产交换的股份数量。

* 必须不包括针对Vault中资产收取的任何费用。
* 必须不显示根据调用者而异的任何变化。
* 在执行实际交换时，必须不反映滑点或其他链上条件。
* 必须不回滚。

> NOTE
此计算可能不反映“每个用户”的每股价格，而应反映“平均用户”的每股价格，即用户在兑换时应该期望看到的价格。

#### convertToAssets(uint256 shares) → uint256 assets
外部#
这个函数返回在理想情况下，保险库会为提供的份额兑换的资产数量。

* 不应包含在保险库中对资产收取的任何费用。

* 不应根据调用者的不同显示任何变化。

* 在执行实际交换时，不应反映滑点或其他链上条件。

* 不应发生回退。

> NOTE
这个计算可能不反映“每用户”的每份价格，而应反映“平均用户”的每份价格，也就是平均用户在兑换时应该期望看到的价格。

#### maxDeposit(address receiver) → uint256 maxAssets
外部#
返回接收者可以通过存款调用向保险库存入的基础资产的最大数量。

如果接收者受到某些存款限制，必须返回一个有限的值。

如果对可以存入的资产的最大数量没有限制，必须返回 2 ** 256 - 1。

不得出现回滚。

#### previewDeposit(uint256 assets) → uint256 shares
外部#
允许链上或链下用户模拟其存款在当前区块的影响，考虑当前链上条件。

* 在同一笔交易中，必须返回与预览存款相同或更多的 Vault 份额。换句话说，如果在同一笔交易中调用存款函数，存款应该返回与预览存款相同或更多的份额。

* 不应考虑存款限制，比如从最大存款返回的限制，无论用户是否已经批准足够的代币。

* 必须包括存款费用。集成商应意识到存款费用的存在。

* 不得出现错误回滚。

> NOTE
在 convertToShares 和 previewDeposit 之间的不利差异应被视为份额价格滑点或其他类型的条件，这意味着存款人将通过存款失去资产。

#### deposit(uint256 assets, address receiver) → uint256 shares
外部#
Mints（铸造）通过存入与基础代币数量完全相等的份额将Vault（保险库）份额分享给接收者。

* 必须触发Deposit（存款）事件。
* 可以支持一种额外的流程，在存款执行之前，基础代币由Vault合约拥有，并在存款期间进行核算。
* 如果无法存入所有资产（由于达到存款限制、滑点、用户未向Vault合约批准足够的基础代币等），必须回滚（revert）。

> NOTE
大多数实现都需要预先批准Vault对其基础资产代币的使用权。

#### maxMint(address receiver) → uint256 maxShares
外部#
如果通过铸造调用，返回接收者可以铸造的Vault股份的最大数量。- 如果接收者受到某些铸造限制，则必须返回有限值。- 如果没有关于可以铸造的最大股份数的限制，则必须返回2 ** 256 - 1。- 不能回滚。

#### previewMint(uint256 shares) → uint256 assets
外部#
允许在链上或链外的用户在当前区块模拟其铸币操作的影响，根据当前链上的条件。

* 必须返回尽可能接近并且不少于在同一交易中进行铸币调用时将存入的确切资产数量。也就是说，如果在同一交易中调用，铸币应该返回与预览铸币相同或更少的资产数量。
*  不得考虑像从maxMint返回的铸币限制，并且始终应该像铸币将被接受一样行事，无论用户是否拥有足够的代币批准等。
* 必须包括存款费用。集成者应该意识到存款费用的存在。
* 不得回滚。

convertToAssets和previewMint之间的任何不利差异应被视为份额价格滑点或其他类型的条件，这意味着存款人通过铸币将失去资产。

#### mint(uint256 shares, address receiver) → uint256 assets
外部#
Mints（铸币）通过存入相应数量的基础代币将Vault（保险库）的份额准确地分享给接收者。

* 必须触发Deposit（存款）事件。
* 可以支持另一种流程，即在铸币执行之前，基础代币由Vault合约拥有，并在铸币过程中予以记录。
* 如果无法铸造所有份额（由于达到存款限制、滑点、用户未向Vault合约批准足够的基础代币等原因），必须回滚（revert）。

> NOTE
大多数实现将要求预先批准Vault合约与Vault的基础资产代币。

#### maxWithdraw(address owner) → uint256 maxAssets
外部#
通过提款调用，返回从Vault中的所有者余额中可以提取的基础资产的最大金额。

* 如果所有者受到提款限制或时间锁的限制，必须返回有限的值。
* 不得回滚。

#### previewWithdraw(uint256 assets) → uint256 shares
外部#
允许链上或链下用户模拟其在当前区块中提现的影响，根据当前的链上条件。

* 必须返回尽可能接近并且不少于在同一笔交易中调用提现时将烧毁的确切Vault股份数量。换句话说，如果在同一笔交易中调用，提现应该返回与previewWithdraw相同或更少的股份。
* 不得考虑像从maxWithdraw返回的提现限制，并且应始终表现为接受提现，无论用户是否拥有足够的股份等。
* 必须包括提现费用。集成者应意识到存在提现费用。
* 不得回滚。

> NOTE
convertToShares和previewWithdraw之间的任何不利差异应被视为份额价格滑点或其他类型的条件，这意味着存款人将通过存款损失资产。

#### withdraw(uint256 assets, address receiver, address owner) → uint256 shares
外部#
将所有者的股份销毁，并将底层代币的确切资产发送给接收者。

* 必须发出Withdraw事件。
* 可以支持一种额外的流程，在执行提款操作之前，底层代币由Vault合约拥有，并在提款期间进行核算。
* 如果无法提取所有资产（由于达到提款限额、滑点、所有者股份不足等），必须回滚。

请注意，某些实现可能要求在执行提款之前向Vault发出预请求。这些方法应在单独执行。

#### maxRedeem(address owner) → uint256 maxShares
外部#
通过赎回调用，返回可以从Vault中的所有者余额中赎回的最大数量的Vault股份。

* 如果所有者受到某种提款限制或时间锁定的限制，必须返回有限的值。
* 如果所有者没有受到任何提款限制或时间锁定的限制，必须返回balanceOf(owner)。
* 不能回滚。

#### previewRedeem(uint256 shares) → uint256 assets
外部#
允许链上或链下用户模拟在当前区块下他们的赎回效果允许链上或链下用户在当前区块模拟其赎回的效果，给定当前链上条件。

* 在同一交易中，必须返回尽可能接近且不超过赎回调用中将提取的确切资产金额。也就是说，如果在同一交易中调用redeem，它应返回与previewRedeem相同或更多的资产。

* 不应考虑像从maxRedeem返回的赎回限制，并且应始终假设赎回将被接受，无论用户是否拥有足够的份额等。

* 必须包括提现费用。集成者应意识到提现费用的存在。

* 不得回滚。

> NOTE
convertToAssets和previewRedeem之间的任何不利差异应被视为份额价格滑动或其他类型的条件，这意味着存款人通过赎回将损失资产。

#### redeem(uint256 shares, address receiver, address owner) → uint256 assets
外部#
准确地从所有者处销毁份额，并将基础代币的资产发送给接收者。

* 必须发出撤回事件。
* 可以支持一种额外的流程，在赎回执行之前，基础代币由Vault合约拥有，并在赎回过程中进行核算。
* 如果无法赎回所有份额（由于达到提款限额、滑点、所有者份额不足等），必须回滚。

> NOTE
一些实现将要求在进行提款之前向Vault预请求。这些方法应该分别执行。

#### Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares)
event#

#### Withdraw(address indexed sender, address indexed receiver, address indexed owner, uint256 assets, uint256 shares)
event#

### IERC5313
```
import "@openzeppelin/contracts/interfaces/IERC5313.sol";
```

为轻合约所有权标准的接口。

需要一个标准化的最小接口来识别控制合约的账户。
*从v4.9版本开始可用。*

**FUNCTIONS**
[owner()](#owner-→-address)

#### owner() → address
外部#
获取所有者的地址。

### IERC5267
```
import "@openzeppelin/contracts/interfaces/IERC5267.sol";
```

**FUNCTIONS**
[eip712Domain()](#eip712domain-→-bytes1-fields-string-name-string-version-uint256-chainid-address-verifyingcontract-bytes32-salt-uint256-extensions)

**EVENTS**
[EIP712DomainChanged()](#eip712domainchanged)

#### eip712Domain() → bytes1 fields, string name, string version, uint256 chainId, address verifyingContract, bytes32 salt, uint256[] extensions
外部#
返回描述此合约用于EIP-712签名的域分隔符使用的字段和值。

#### EIP712DomainChanged()
event#
可能会发出信号，表明域名可能已更改。

### IERC5805
```
import "@openzeppelin/contracts/interfaces/IERC5805.sol";
```

**FUNCTIONS**

IVOTES
[getVotes(account)](./Governance.md)
[getPastVotes(account, timepoint)](./Governance.md)
[getPastTotalSupply(timepoint)](./Governance.md)
[delegates(account)](./Governance.md)
[delegate(delegatee)](./Governance.md)
[delegateBySig(delegatee, nonce, expiry, v, r, s)](./Governance.md)

IERC6372
[clock()](#clock-→-uint48)
[CLOCK_MODE()](#clock_mode-→-string)

**EVENTS**

IVOTES
[DelegateChanged(delegator, fromDelegate, toDelegate)](./Governance.md)
[DelegateVotesChanged(delegate, previousBalance, newBalance)](./Governance.md)

### IERC6372
```
import "@openzeppelin/contracts/interfaces/IERC6372.sol";
```

**FUNCTIONS**
[clock()](#clock-→-uint48)

[CLOCK_MODE()](#clock_mode-→-string)

#### clock() → uint48
外部#
用于标记检查点的时钟。可以被重写以实现基于时间戳的检查点（和投票）。

#### CLOCK_MODE() → string
外部#
时钟描述