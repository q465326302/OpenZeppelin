# Draft EIPs
该目录包含的是仍处于草案状态的EIP（以太坊改进建议）的实现。

由于草案的性质，这些合约的细节可能会发生变化，我们无法保证其[稳定性](../Releases&Stability.md)。OpenZeppelin Contracts的次要发布版本可能会对该目录中的合约进行重大更改，这将在[更新日志](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/CHANGELOG.md)中进行相应公告。这些EIP的实现已被生产项目使用，这使得它们不太可能发生重大变化。

## Cryptography

### EIP712
[EIP 712](https://eips.ethereum.org/EIPS/eip-712)是一种用于对类型化结构化数据进行哈希和签名的标准。

EIP中指定的编码非常通用，因此在Solidity中实现这样的通用编码是不可行的，因此该合约不实现编码本身。协议需要使用abi.encode和keccak256的组合在其合约中实现他们所需的特定类型编码。

该合约实现了作为编码方案的一部分所使用的EIP 712域分隔符（[_domainSeparatorV4](#_domainseparatorv4-→-bytes32)），以及通过ECDSA对消息摘要进行签名的编码的最后一步（[_hashTypedDataV4](#_hashtypeddatav4bytes32-structhash-→-bytes32)）。

> NOTE
域分隔符的实现被设计为尽可能高效，同时仍然能够正确更新链ID，以防止在未来的链分叉上发生重放攻击。

该合约实现了被称为"v4"的编码版本，该版本由MetaMask中的JSON RPC方法[eth_signTypedDataV4实现](https://docs.metamask.io/guide/signing-data.html)。

*自v3.4版本起可用。*

**FUNCTIONS**
[constructor(name, version)](#constructorstring-name-string-version)

[_domainSeparatorV4()](#_domainseparatorv4-→-bytes32)

[_hashTypedDataV4(structHash)](#_hashtypeddatav4bytes32-structhash-→-bytes32)

#### constructor(string name, string version)
内部#
初始化域分隔符和参数缓存。

在[EIP 712](https://eips.ethereum.org/EIPS/eip-712#definition-of-domainseparator)中指定了名称和版本的含义：

* 名称：可供用户阅读的签名域名称，即DApp或协议的名称。

* 版本：签名域的当前主要版本。

> NOTE
除非通过[智能合约升级](../../../Learn/Upgrading-smart-contracts/Upgrading-smart-contracts-hardhat.md)，否则这些参数不能更改。

#### _domainSeparatorV4() → bytes32
内部#
返回当前链的域分隔符。

#### _hashTypedDataV4(bytes32 structHash) → bytes32
内部#
给定一个已经[哈希的结构体](https://eips.ethereum.org/EIPS/eip-712#definition-of-hashstruct)，这个函数返回此域的完全编码的EIP712消息的哈希值。

这个哈希值可以与[ECDSA.recover](./Cryptography.md#recoverbytes32-hash-uint8-v-bytes32-r-bytes32-s-→-address)一起使用，以获取消息的签名者。例如：
```
bytes32 digest = _hashTypedDataV4(keccak256(abi.encode(
    keccak256("Mail(address to,string contents)"),
    mailTo,
    keccak256(bytes(mailContents))
)));
address signer = ECDSA.recover(digest, signature);
```

## ERC 20

### IERC20Permit
ERC20 Permit扩展的接口，允许通过签名进行批准，如[EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)中定义的。

新增了[permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)方法，可以通过提供由账户签名的消息来更改账户的ERC20授权（参见[IERC20.allowance](../API/ERC20.md#allowanceaddress-owner-address-spender-→-uint256)）。通过不依赖于[IERC20.approve](../API/ERC20.md#approveaddress-spender-uint256-amount-→-bool)，代币持有者账户无需发送交易，因此不需要持有任何以太币。

**FUNCTIONS**
[permit(owner, spender, value, deadline, v, r, s)](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)

[nonces(owner)](#noncesaddress-owner-→-uint256)

[DOMAIN_SEPARATOR()](#domain_separator-→-bytes32)

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
外部#
设置spender的津贴为owner的代币，前提是owner已经签署了批准。

> IMPORTANT
与[IERC20.approve](./ERC20.md#approveaddress-spender-uint256-amount-→-bool)相同，此函数也存在与交易排序有关的问题。
触发一个{Approval}事件。

要求：
* spender不能是零地址。

* 截止日期必须是未来的时间戳。

* v、r和s必须是owner对EIP712格式化函数参数的有效secp256k1签名。

* 签名必须使用owner的当前nonce（参见 [nonces](#noncesaddress-owner-→-uint256)）。

有关签名格式的更多信息，请参见相关的[EIP部分](https://eips.ethereum.org/EIPS/eip-2612#specification)。

#### nonces(address owner) → uint256
内部#
返回所有者的当前nonce。无论何时为 [permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)生成签名，都必须包含此值。

每次成功调用[permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s)都会使所有者的nonce增加一。这可以防止签名被多次使用。

#### DOMAIN_SEPARATOR() → bytes32
内部#
返回[EIP712](./Drafts.md#eip712)定义的用于许可签名编码的域分隔符。

### ERC20Permit
实施了ERC20 Permit扩展，允许通过签名进行批准，如[EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)中定义的。

添加了[permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s-1)方法，可以通过提供由账户签名的消息来更改账户的ERC20授权（参见[IERC20.allowance](./ERC20.md#allowanceaddress-owner-address-spender-→-uint256)）。通过不依赖于[IERC20.approve](./ERC20.md#approveaddress-spender-uint256-amount-→-bool)，代币持有人账户不需要发送交易，因此不需要持有任何以太币。

*自v3.4起可用。*

**FUNCTIONS**
[constructor(name)](#constructorstring-name)

[permit(owner, spender, value, deadline, v, r, s)](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s-1)

[nonces(owner)](#noncesaddress-owner-e28692-uint256-1)

[DOMAIN_SEPARATOR()](#domain_separator-e28692-bytes32-1)

EIP712
[_domainSeparatorV4()](#_domainseparatorv4-→-bytes32)

[_hashTypedDataV4(structHash)](#_hashtypeddatav4bytes32-structhash-→-bytes32)

ERC20
[name()](./ERC20.md#name-→-string)

[symbol()](./ERC20.md#symbol-→-string)

[decimals()](./ERC20.md#decimals-→-uint8)

[totalSupply()](./ERC20.md#totalsupply-e28692-uint256-1)

[balanceOf(account)](./ERC20.md#balanceofaddress-account-e28692-uint256-1)

[transfer(recipient, amount)](./ERC20.md#transferaddress-recipient-uint256-amount-e28692-bool-1)

[allowance(owner, spender)](./ERC20.md#allowanceaddress-owner-address-spender-e28692-uint256-1)

[approve(spender, amount)](./ERC20.md#approveaddress-spender-uint256-amount-e28692-bool-1)

[transferFrom(sender, recipient, amount)](./ERC20.md#transferaddress-recipient-uint256-amount-e28692-bool-1)

[increaseAllowance(spender, addedValue)](./ERC20.md#increaseallowanceaddress-spender-uint256-addedvalue-→-bool)

[decreaseAllowance(spender, subtractedValue)](./ERC20.md#decreaseallowanceaddress-spender-uint256-subtractedvalue-→-bool)

[_transfer(sender, recipient, amount)](./ERC20.md#_transferaddress-sender-address-recipient-uint256-amount)

[_mint(account, amount)](./ERC20.md#_mintaddress-account-uint256-amount)

[_burn(account, amount)](./ERC20.md#_burnaddress-account-uint256-amount)

[_approve(owner, spender, amount)](./ERC20.md#_approveaddress-owner-address-spender-uint256-amount)

[_setupDecimals(decimals_)](./ERC20.md#_setupdecimalsuint8-decimals)

[_beforeTokenTransfer(from, to, amount)](./ERC20.md#_beforetokentransferaddress-from-address-to-uint256-amount)

**EVENTS**
IERC20
[Transfer(from, to, value)](./ERC20.md#transferaddress-from-address-to-uint256-value)

[Approval(owner, spender, value)](./ERC20.md#approvaladdress-owner-address-spender-uint256-value)

#### constructor(string name)
内部#
使用名称参数初始化[EIP712](#eip712)域分隔符，并将版本设置为"1"。

使用与ERC20代币名称相同的名称是一个好主意。

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
公开#
请查看[IERC20Permit.permit](#permitaddress-owner-address-spender-uint256-value-uint256-deadline-uint8-v-bytes32-r-bytes32-s).

#### nonces(address owner) → uint256
公开#
请查看[IERC20Permit.nonces](#noncesaddress-owner-→-uint256).

#### DOMAIN_SEPARATOR() → bytes32
外部#
请查看[IERC20Permit.DOMAIN_SEPARATOR](#domain_separator-→-bytes32).