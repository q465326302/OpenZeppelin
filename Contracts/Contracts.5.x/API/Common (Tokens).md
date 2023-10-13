# Common (Tokens)
在多个令牌标准中常见的功能。

* *ERC2981*：NFT版税兼容ERC721和ERC1155。

* 对于ERC721，考虑使用*ERC721Royalty*，在销毁时从存储中清除版税信息。

## Contracts

### [ERC2981](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/common/ERC2981.sol)
```
import "@openzeppelin/contracts/token/common/ERC2981.sol";
```

实现了NFT版税标准，这是一种标准化的方式来检索版税支付信息。

版税信息可以通过*_setDefaultRoyalty*全局指定所有的令牌id，和/或者通过*_setTokenRoyalty*单独指定特定的令牌id。后者优先于前者。

版税被指定为销售价格的一部分。*_feeDenominator*是可以覆盖的，但默认为10000，这意味着默认情况下，费用是以基点来指定的。

> IMPORTANT
ERC-2981只指定了一种表示版税信息的方式，并没有强制支付。参见EIP中的[理由](https://eips.ethereum.org/EIPS/eip-2981#optional-royalty-payments)。市场预计会自愿支付版税和销售，但请注意，这个标准还没有得到广泛的支持。

**FUNCTIONS**
supportsInterface(interfaceId)

royaltyInfo(tokenId, salePrice)

_feeDenominator()

_setDefaultRoyalty(receiver, feeNumerator)

_deleteDefaultRoyalty()

_setTokenRoyalty(tokenId, receiver, feeNumerator)

_resetTokenRoyalty(tokenId)

**ERRORS**
ERC2981InvalidDefaultRoyalty(numerator, denominator)

ERC2981InvalidDefaultRoyaltyReceiver(receiver)

ERC2981InvalidTokenRoyalty(tokenId, numerator, denominator)

ERC2981InvalidTokenRoyaltyReceiver(tokenId, receiver)

#### supportsInterface(bytes4 interfaceId) → bool
public#
请查阅 **IERC165.supportsInterface**.

#### royaltyInfo(uint256 tokenId, uint256 salePrice) → address, uint256
public#
返回应支付多少版税以及给谁，基于可能以任何交易单位计价的销售价格。版税金额以该交易单位计价，并应以该单位支付。

#### _feeDenominator() → uint96
internal#
用于解释*_setTokenRoyalty*和*_setDefaultRoyalty*中设定的费用的分母，作为销售价格的一部分。默认值为10000，因此费用以基点表示，但可以通过覆盖进行自定义。

#### _setDefaultRoyalty(address receiver, uint96 feeNumerator)
internal#
设置此合约中所有id默认的版权信息。

要求：
* 接收者不能是零地址。

* 费用分子不能大于费用分母。

#### _deleteDefaultRoyalty()
internal#
删除默认版权信息。

#### _setTokenRoyalty(uint256 tokenId, address receiver, uint96 feeNumerator)
internal#
为特定的令牌id设置版权信息，覆盖全局默认设置。

要求：
* 接收者不能是零地址。

* 费用分子不能大于费用分母。

#### _resetTokenRoyalty(uint256 tokenId)
internal#
将代币ID的版权信息重置回全球默认值。

#### ERC2981InvalidDefaultRoyalty(uint256 numerator, uint256 denominator)
error#
默认的版税设置无效（例如，（分子/分母）>= 1）。

#### ERC2981InvalidDefaultRoyaltyReceiver(address receiver)
error#
默认的版权收益接收者无效。

#### ERC2981InvalidTokenRoyalty(uint256 tokenId, uint256 numerator, uint256 denominator)
error#
为特定tokenId设置的版权费无效（例如，（分子/分母）>= 1）。

#### ERC2981InvalidTokenRoyaltyReceiver(uint256 tokenId, address receiver)
error#
tokenId的版权接收者无效。