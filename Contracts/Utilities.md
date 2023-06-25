# Utilities
OpenZeppelin Contracts提供了许多有用的工具，您可以在项目中使用。以下是一些比较流行的工具。

## 加密学

### 在链上检查签名
*ECDSA*提供了用于恢复和管理以太坊账户ECDSA签名的函数。这些签名通常是通过[web3.eth.sign](https://web3js.readthedocs.io/en/v1.7.3/web3-eth.html#sign)生成的，是一个65字节的数组（在Solidity中是bytes类型），排列方式如下：[[v(1)],[r(32)],[s(32)]]。

数据签名者可以通过*ECDSA.recover*恢复，并将其地址与签名进行比较以验证签名。大多数钱包将哈希数据进行签名，并添加前缀'\x19Ethereum Signed Message:\n'，因此在尝试恢复以太坊签名消息哈希的签名者时，您需要使用*toEthSignedMessageHash*。
```
using ECDSA for bytes32;

function _verify(bytes32 data, bytes memory signature, address account) internal pure returns (bool) {
    return data
        .toEthSignedMessageHash()
        .recover(signature) == account;
}
```

>WARNING
获取签名验证正确并不容易：确保您完全阅读并理解 *ECDSA* 的文档。

### 验证 Merkle 证明
*MerkleProof* 提供了：

* *verify* - 可以证明某个值是 [Merkle 树](https://en.wikipedia.org/wiki/Merkle_tree)的一部分。

* *multiProofVerify* - 可以证明多个值是 Merkle 树的一部分。

## 内省
在 Solidity 中，了解一个合约是否支持您想要使用的接口通常是有帮助的。ERC165 是一种标准，可以帮助进行运行时接口检测。合约提供了帮助程序，用于实现您的合约中的 ERC165 并查询其他合约：

* *IERC165* - 这是定义 *supportsInterface* 的 ERC165 接口。在实现 ERC165 时，您将符合此接口。

* *ERC165* - 如果您想使用合同存储中的查找表支持接口检测，请继承此合约。您可以使用 *_registerInterface(bytes4)* 注册接口：请查看作为 ERC721 实现的示例用法。

* ERC165Checker - ERC165Checker 简化了检查合约是否支持您关心的接口的过程。

* 包括使用 ERC165Checker for address;

* myAddress._supportsInterface(bytes4)

* myAddress._supportsAllInterfaces(bytes4[])
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
OpenZeppelin Contracts是最受欢迎的与数学相关的库，它提供了*SafeMath*，它提供了数学函数，保护您的合约免受溢出和下溢的影响。

使用SafeMath for uint256; 包含具有函数调用：

* myNumber.add(otherNumber)
* myNumber.sub(otherNumber)
* myNumber.div(otherNumber)
* myNumber.mul(otherNumber)
* myNumber.mod(otherNumber)
很容易！

## 支付

想要在多个人之间分配一些付款吗？也许您有一个应用程序，可以将艺术品购买的30％发送给原始创建者，并将70％的利润发送给当前所有者。您可以使用*PaymentSplitter*构建它！

在Solidity中，盲目向帐户发送资金存在一些安全问题，因为它允许它们执行任意代码。您可以在[以太坊智能合约最佳实践网站](https://consensys.github.io/smart-contract-best-practices/)上阅读有关这些安全问题的信息。解决重入和停顿问题的一种方法是，而不是立即向需要资金的帐户发送以太币，您可以使用*PullPayment*，它提供了一个*_asyncTransfer*函数，用于向某个东西发送资金并请求他们稍后将其*withdrawPayments()*。

如果您想托管一些资金，请查看*Escrow*和*ConditionalEscrow*，以管理一些托管以太币的释放。

## 收集品

如果您需要比Solidity的本地数组和映射更强大的收集支持，请查看*EnumerableSet*和*EnumerableMap*。它们类似于映射，因为它们以恒定的时间存储和删除元素，并且不允许重复条目，但是它们还支持枚举，这意味着您可以轻松地查询所有存储的条目，无论在链上还是链下。

## 杂项
想要检查地址是否是合约吗？使用*Address*和*Address.isContract()*。

想要跟踪一些数字，每次需要另一个数字时增加1吗？请查看*Counters*。这对于许多事情都很有用，例如创建递增标识符，如*ERC721指南*所示。

### Base64
*Base64* util允许您将bytes32数据转换为其Base64字符串表示形式。

这对于构建适用于*ERC721*或*ERC1155*的URL安全的tokenURI特别有用。此库提供了一种聪明的方式，以提供符合URL[安全数据URI](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/Data_URIs/)标准的字符串，以便在链上数据结构中使用。

以下是使用ERC721通过Base64数据URI发送JSON元数据的示例：
```
// contracts/My721Token.sol
// SPDX-License-Identifier: MIT

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/utils/Strings.sol";
import "@openzeppelin/contracts/utils/Base64.sol";

contract My721Token is ERC721 {
    using Strings for uint256;

    constructor() ERC721("My721Token", "MTK") {}

    ...

    function tokenURI(uint256 tokenId)
        public
        pure
        override
        returns (string memory)
    {
        bytes memory dataURI = abi.encodePacked(
            '{',
                '"name": "My721Token #', tokenId.toString(), '"',
                // Replace with extra ERC721 Metadata properties
            '}'
        );

        return string(
            abi.encodePacked(
                "data:application/json;base64,",
                Base64.encode(dataURI)
            )
        );
    }
}
```

### Multicall
Multicall抽象合约带有一个multicall函数，它将多个调用捆绑在单个外部调用中。使用它，外部账户可以执行由多个函数调用组成的原子操作。这不仅对EOA在单个交易中进行多个调用很有用，而且还是在后续调用失败时回滚先前调用的一种方式。

考虑这个虚拟合约：
```
// contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Multicall.sol";

contract Box is Multicall {
    function foo() public {
        ...
    }

    function bar() public {
        ...
    }
}
```
这是如何使用Truffle调用multicall函数，允许在单个交易中调用foo和bar：
```
// scripts/foobar.js

const Box = artifacts.require('Box');
const instance = await Box.new();

await instance.multicall([
    instance.contract.methods.foo().encodeABI(),
    instance.contract.methods.bar().encodeABI()
]);
```
