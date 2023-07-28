# Utilities
OpenZeppelin Contracts提供了许多在项目中可以使用的实用工具。以下是其中一些较受欢迎的工具。

## Cryptography

### 在链上检查签名
*ECDSA*提供了用于恢复和管理以太坊账户ECDSA签名的函数。这些签名通常通过[web3.eth.sign](https://web3js.readthedocs.io/en/v1.2.4/web3-eth.html#sign)生成，并且是一个65字节的数组（在Solidity中为bytes类型），按照以下方式排列：[[v（1）]，[r（32）]，[s（32）]]。

可以使用*ECDSA.recover*来恢复数据签名者，并将其地址与签名进行验证。大多数钱包会对要签名的数据进行哈希处理，并添加前缀'\x19Ethereum Signed Message:\n'，因此在尝试恢复以太坊签名消息哈希的签名者时，您需要使用*toEthSignedMessageHash*。

```
using ECDSA for bytes32;

function _verify(bytes32 data, address account) pure returns (bool) {
    return keccack256(data)
        .toEthSignedMessageHash()
        .recover(signature) == account;
}
```

> WARNING
正确进行签名验证并不是一件简单的事情：确保你完全阅读并理解了*ECDSA*的文档。

### 验证Merkle证明
*MerkleProof*提供了*verify*函数，可以证明某个值是[Merkle树](https://en.wikipedia.org/wiki/Merkle_tree)的一部分。

### Introspection

在Solidity中，了解一个合约是否支持你想要使用的接口通常是有帮助的。ERC165是一个标准，用于在运行时进行接口检测。Contracts提供了一些辅助函数，用于在你的合约中实现ERC165并查询其他合约：

* *IERC165* - 这是定义*supportsInterface*函数的ERC165接口。当实现ERC165时，你将符合这个接口。

* *ERC165* - 如果你想要使用合约存储中的查找表来支持接口检测，请继承这个合约。你可以使用*_registerInterface(bytes4)*来注册接口：在ERC721实现中可以查看示例用法。

* *ERC165Checker* - ERC165Checker简化了检查合约是否支持你关心的接口的过程。

* 使用ERC165Checker for address; 导入并使用ERC165Checker库。

* *myAddress._supportsInterface(bytes4)* - 检查合约地址myAddress是否支持给定接口(bytes4)。

* *myAddress._supportsAllInterfaces(bytes4[])* - 检查合约地址myAddress是否支持给定接口数组(bytes4[])中的所有接口。

```
contract MyContract {
    using ERC165Checker for address;

    bytes4 private InterfaceId_ERC721 = 0x80ac58cd;

    /**
     * @dev transfer an ERC721 token from this contract to someone else
     */
    function transferERC721(
        address token,
        address to,
        uint256 tokenId
    )
        public
    {
        require(token.supportsInterface(InterfaceId_ERC721), "IS_NOT_721_TOKEN");
        IERC721(token).transferFrom(address(this), to, tokenId);
    }
}
```

## Math
最受欢迎的与数学相关的库OpenZeppelin Contracts提供了*SafeMath*，它提供了数学函数，可以保护您的合约免受溢出和下溢的影响。

在合约中包含使用SafeMath for uint256;，然后调用以下函数：

* myNumber.add(otherNumber)

* myNumber.sub(otherNumber)

* myNumber.div(otherNumber)

* myNumber.mul(otherNumber)

* myNumber.mod(otherNumber)

很简单！

## Payment
想要在多个人之间分割一些付款吗？也许您有一个应用程序，将艺术购买的30％发送给原始创作者，将70％的利润发送给当前所有者；您可以使用*PaymentSplitter*构建这个功能！

在Solidity中，盲目向帐户发送资金存在一些安全问题，因为这允许它们执行任意代码。您可以在[以太坊智能合约最佳实践网站](https://consensys.github.io/smart-contract-best-practices/)上阅读有关这些安全问题的信息。修复重新进入和停滞问题的一种方法是，不要立即向需要资金的帐户发送以太币，而是使用*PullPayment*，它提供了一个*_asyncTransfer*函数，用于将资金发送给某个人，并要求他们稍后通过*withdrawPayments()*提取。

如果您想要托管一些资金，请查看*Escrow*和*ConditionalEscrow*来管理某些托管以太币的释放。

## Collections
如果您需要比Solidity的本地数组和映射更强大的集合支持，请查看*EnumerableSet*。它类似于映射，因为它以恒定时间存储和删除元素，并且不允许重复的条目，但它还支持枚举，这意味着您可以轻松地查询集合的所有元素，无论是在链上还是离线。

## Collections
想要检查地址是否为合约吗？使用*Address*和*Address.isContract()*。

想要跟踪一些每次都需要增加1的数字吗？请查看*Counter*。这对于许多事情都很有用，比如创建递增的标识符，正如*ERC721指南*所示。