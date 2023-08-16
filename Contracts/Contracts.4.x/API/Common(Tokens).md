# Common (Tokens)
适用于多个代币标准的常见功能。

* [ERC2981](#erc2981)：NFT版税与ERC721和ERC1155兼容。
    * 对于ERC721，请考虑[ERC721Royalty](./ERC721.md#erc721royalty)，它会在销毁时从存储中清除版税信息。

## 合约

## ERC2981
```
import“@openzeppelin/contracts/token/common/ERC2981.sol”;
```

实现NFT版税标准，一种检索版税支付信息的标准化方式。

版税信息可以通过[_setDefaultRoyalty](#_setdefaultroyaltyaddress-receiver-uint96-feenumerator)为所有代币ID全局指定，也可以通过[_setTokenRoyalty](#_settokenroyaltyuint256-tokenid-address-receiver-uint96-feenumerator)为特定代币ID单独指定。后者优先于前者。

版税被指定为销售价格的一部分。 [_feeDenominator](#_feedenominator-→-uint96)是可重写的，但默认为10000，这意味着默认情况下以基点指定费用。

> IMPORTANT
ERC-2981仅指定了一种信号版税信息的方式，不强制执行其支付。请参见EIP中的[Rationale](https://eips.ethereum.org/EIPS/eip-2981#optional-royalty-payments)。市场预计将与销售一起自愿支付版税，但请注意，此标准尚未得到广泛支持。

*自v4.5起可用。*

**FUNCTIONS**
[supportsInterface(interfaceId)](#supportsinterfacebytes4-interfaceid-→-bool)
[royaltyInfo(tokenId, salePrice)](#royaltyinfouint256-tokenid-uint256-saleprice-→-address-uint256)
[_feeDenominator()](#_feedenominator-→-uint96)
[_setDefaultRoyalty(receiver, feeNumerator)](#_setdefaultroyaltyaddress-receiver-uint96-feenumerator)
[_deleteDefaultRoyalty()](#_deletedefaultroyalty)
[_setTokenRoyalty(tokenId, receiver, feeNumerator)](#_settokenroyaltyuint256-tokenid-address-receiver-uint96-feenumerator)
[_resetTokenRoyalty(tokenId)](#_resettokenroyaltyuint256-tokenid)

#### supportsInterface(bytes4 interfaceId) → bool
公开#
请参阅[IERC165.supportsInterface](./Utils.md#supportsinterfacebytes4-interfaceid-→-bool)。

#### royaltyInfo(uint256 tokenId, uint256 salePrice) → address, uint256
公开#
根据可能以任何交换单位计价的销售价格，返回应支付的版税金额及其受益人。版税金额应以同一交换单位计价并支付。

#### _feeDenominator() → uint96
内部#
[_setTokenRoyalty](#_setdefaultroyaltyaddress-receiver-uint96-feenumerator)和[_setDefaultRoyalty](#_settokenroyaltyuint256-tokenid-address-receiver-uint96-feenumerator)中设置的费用的分母，用于将其解释为销售价格的分数。默认为10000，因此费用以基点表示，但可以通过重写进行自定义。

#### _setDefaultRoyalty(address receiver, uint96 feeNumerator)
内部#
设置所有合约中的ID默认的版税信息。
要求：
* 接收方地址不能为零地址。
* 费用分子不能大于费用分母。

#### _deleteDefaultRoyalty()
内部#
移除默认版权信息。

#### _setTokenRoyalty(uint256 tokenId, address receiver, uint96 feeNumerator)
内部#
设置特定代币ID的版税信息，重写全局默认设置。
要求：
* 接收者不能为零地址。
* 费用分子不能大于费用分母。

#### _resetTokenRoyalty(uint256 tokenId)
内部#
将代币ID的版税信息重置为全局默认值。