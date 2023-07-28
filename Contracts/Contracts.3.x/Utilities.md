# Utilities
OpenZeppelin Contracts提供了许多有用的工具，您可以在项目中使用这些工具。以下是其中一些较受欢迎的工具。

## 密码学

### 在链上检查签名
*ECDSA*提供了用于恢复和管理以太坊账户ECDSA签名的函数。这些签名通常通过[web3.eth.sign](https://web3js.readthedocs.io/en/v1.2.4/web3-eth.html#sign)生成，是一个65字节的数组（在Solidity中为bytes类型），排列方式如下：[[v（1）]，[r（32）]，[s（32）]]。

可以使用*ECDSA.recover*来恢复数据的签名者，并将其地址与签名进行验证。大多数钱包会对要签名的数据进行哈希处理，并添加前缀'\x19Ethereum Signed Message:\n'，因此在尝试恢复以太坊签名消息哈希的签名者时，您需要使用*toEthSignedMessageHash*。

```
using ECDSA for bytes32;

function _verify(bytes32 data, address account) pure returns (bool) {
    return keccack256(data)
        .toEthSignedMessageHash()
        .recover(signature) == account;
}
```
> WARNING
正确验证签名并不容易：确保您完全阅读并理解ECDSA的文档。

## 验证默克尔证明
*MerkleProof*提供了*verify*函数，可以证明某个值是[默克尔树](https://en.wikipedia.org/wiki/Merkle_tree)的一部分。

## Introspection
在Solidity中，了解一个合约是否支持您想使用的接口通常很有帮助。ERC165是一个标准，用于帮助运行时接口检测。Contracts提供了帮助程序，用于在您的合约中实现ERC165并查询其他合约：

* IERC165 - 这是定义*supportsInterface*函数的ERC165接口。当实现ERC165时，您将符合这个接口。

* ERC165 - 如果您希望使用合约存储中的查找表支持接口检测，请继承此合约。您可以使用_registerInterface(bytes4)注册接口：请查看ERC721实现的示例用法。

* ERC165Checker - ERC165Checker简化了检查合约是否支持您关心的接口的过程。

* 包括使用ERC165Checker的地址;

* *myAddress._supportsInterface(bytes4)*

* *myAddress._supportsAllInterfaces(bytes4[])*

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

## 数学
最受欢迎的与数学相关的库OpenZeppelin Contracts提供了SafeMath，它提供了数学函数，可以保护您的合约免受溢出和下溢的影响。

在合约中包含使用*SafeMath*的代码，类型为uint256;，然后调用这些函数：

* myNumber.add(otherNumber)

* myNumber.sub(otherNumber)

* myNumber.div(otherNumber)

* myNumber.mul(otherNumber)

* myNumber.mod(otherNumber)

很简单！

## 支付
想要将一些支付分配给多个人吗？也许您有一个应用程序，将艺术品购买的30％发送给原始创作者，将利润的70％发送给当前所有者；您可以使用*PaymentSplitter*来构建这个功能！

在Solidity中，盲目向账户发送资金存在一些安全问题，因为这使得它们能够执行任意代码。您可以在以[太坊智能合约最佳实践网站](https://consensys.github.io/smart-contract-best-practices/)上阅读有关这些安全问题的信息。解决重入和停滞问题的一种方法是，不是立即将以太币发送到需要的账户，而是使用*PullPayment*，它提供了一个*_asyncTransfer*函数，用于将资金发送到某个地方，并要求他们稍后通过*withdrawPayments()*函数提取。

如果您想要托管一些资金，请查看*Escrow*和*ConditionalEscrow*来管理释放的一些托管以太币。

## 集合
如果您需要比Solidity的本机数组和映射更强大的集合支持，请查看*EnumerableSet*和*EnumerableMap*。它们类似于映射，因为它们以恒定的时间存储和删除元素，并且不允许重复的条目，但它们还支持枚举，这意味着您可以轻松地在链上和链下查询所有存储的条目。

## 其他
想要检查一个地址是否是合约吗？使用*Address*和*Address.isContract()*。

想要跟踪一些每次需要另一个数字时递增1的数字吗？看看*Counters*。这对于许多事情都很有用，比如在*ERC721指南*中所示的创建递增标识符。