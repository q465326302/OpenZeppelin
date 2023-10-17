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