# Interfaces

## List of standardized interfaces
这些接口可作为.sol文件和编译器.json ABI文件（通过npm包）提供。它们对于与实现这些接口的第三方合约进行交互非常有用。
* IERC20

* IERC20Metadata

* IERC165

* IERC721

* IERC721Receiver

* IERC721Enumerable

* IERC721Metadata

* IERC777

* IERC777Recipient

* IERC777Sender

* IERC1155

* IERC1155Receiver

* IERC1155MetadataURI

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
ERC1363兼容合约的接口，如EIP所定义的。

定义了一个支持在转账或transferFrom之后执行接收者代码，或在approve之后执行spender代码的ERC20令牌接口。

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
transfer(to, amount)
allowance(owner, spender)
approve(spender, amount)
transferFrom(from, to, amount)

IERC165
supportsInterface(interfaceId)

**EVENTS**

IERC20
Transfer(from, to, value)
Approval(owner, spender, value)

#### transferAndCall(address to, uint256 amount) → bool
外部#
从msg.sender地址转移代币到另一个地址，然后调用接收方的onTransferReceived函数。

#### transferAndCall(address to, uint256 amount, bytes data) → bool
外部#
从msg.sender地址将代币转移到另一个地址，然后调用接收者的onTransferReceived方法。

#### transferFromAndCall(address from, address to, uint256 amount) → bool
外部#
将令牌从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived函数。

#### transferFromAndCall(address from, address to, uint256 amount, bytes data) → bool
外部#
将令牌从一个地址转移到另一个地址，然后在接收者上调用onTransferReceived函数。

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
任何希望支持 *IERC1363.transferAndCall* 或 *IERC1363.transferFromAndCall* 的合约的接口，来自 {ERC1363} 代币合约。

**FUNCTIONS**
onTransferReceived(operator, from, amount, data)

#### onTransferReceived(address operator, address from, uint256 amount, bytes data) → bytes4
外部#
在转账或从智能合约中转账后，任何ERC1363智能合约都会调用此函数。此函数可能会抛出错误并拒绝转账。返回值如果不是预定的特殊值，必须导致交易被撤销。注意：代币合约地址始终为消息发送者。

#### IERC1363Spender
```
import "@openzeppelin/contracts/interfaces/IERC1363Spender.sol";
```

任何合约希望支持来自{ERC1363}代币合约的*IERC1363.approveAndCall*的接口。

**FUNCTIONS**
onApprovalReceived(owner, amount, data)

#### onApprovalReceived(address owner, uint256 amount, bytes data) → bytes4
外部#
在approve之后，任何ERC1363智能合约都会调用此函数来操作接收者。该函数可能会抛出异常以回滚并拒绝授权。返回值若非特定的魔术值，必须导致交易被回滚。注意：代币合约地址始终为消息发送者。

### IERC1822Proxiable
```
import "@openzeppelin/contracts/interfaces/draft-IERC1822.sol";
```
ERC1822: 通用可升级代理标准（UUPS）记录了一种通过简化代理实现完全由当前实现控制的升级能力的方法。

**FUNCTIONS**
proxiableUUID()

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
permit(owner, spender, value, deadline, v, r, s)
nonces(owner)
DOMAIN_SEPARATOR()

### IERC2981
```
import "@openzeppelin/contracts/interfaces/IERC2981.sol";
```
NFT版税标准的接口。

这是一种标准化的方式，用于检索非同质化代币（NFT）的版税支付信息，以实现在所有NFT市场和生态系统参与者之间普遍支持版税支付。

*自版本4.5起可用。*

**FUNCTIONS**
royaltyInfo(tokenId, salePrice)

IERC165
supportsInterface(interfaceId)

### royaltyInfo(uint256 tokenId, uint256 salePrice) → address receiver, uint256 royaltyAmount
外部#
根据可能以任何货币单位表示的销售价格，返回应支付的版税金额以及版税支付对象。版税金额应以相同的货币单位支付。

### IERC3156FlashLender
```
import "@openzeppelin/contracts/interfaces/IERC3156FlashLender.sol";
```

*ERC-3156*定义的ERC3156 FlashLender的接口。

*从v4.1版本开始可用。*