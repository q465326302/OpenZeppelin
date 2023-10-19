# ERC 20
这套接口、合约和工具都与[ERC20代币标准](https://eips.ethereum.org/EIPS/eip-20)有关。

> TIP
要了解ERC20代币的概述以及如何创建代币合约的教程，请阅读我们的*ERC20指南*。

在EIP中规定了一些核心合约的行为：
* *IERC20*：所有ERC20实现应符合的接口。

* *IERC20Metadata*：扩展的ERC20接口，包括名称、符号和小数函数。

* *ERC20*：ERC20接口的实现，包括名称、符号和小数函数，这是对基础接口的可选标准扩展。

此外，还有多个自定义扩展，包括：

* *ERC20Permit*：无需gas费的代币批准（标准化为ERC2612）。

* *ERC20Burnable*：销毁自己的代币。

* *ERC20Capped*：在铸造代币时对总供应量进行上限限制。

* *ERC20Pausable*：能够暂停代币转移。

* *ERC20FlashMint*：通过铸造和销毁短暂代币支持闪电贷款的代币级别支持（标准化为ERC3156）。

* *ERC20Votes*：支持投票和投票代理。

* *ERC20Wrapper*：创建由另一个ERC20支持的ERC20的包装器，具有存款和取款方法。与ERC20Votes一起使用非常有用。

* *ERC4626*：代币化的保险库，管理由资产（另一个ERC20）支持的份额（表示为ERC20）。

最后，有一些实用工具可以以各种方式与ERC20合约进行交互：

* *SafeERC20*：围绕接口的包装器，无需处理布尔返回值。

在代码库中可以找到支持ERC20资产的其他实用工具：

* ERC20代币可以被定时锁定（为受益人持有代币直到指定的时间）或者按照给定的计划进行归属（使用*VestingWallet*）。

> NOTE
这套核心合约设计为无偏见，允许开发者访问ERC20中的内部函数（如*_mint*），并以他们喜欢的方式将其作为外部函数公开。

## 核心

### [IERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/IERC20.sol)
```
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
```

根据EIP定义的ERC20标准的接口。

**FUNCTIONS**
totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

**EVENTS**
Transfer(from, to, value)

Approval(owner, spender, value)

#### totalSupply() → uint256
external#
返回现存代币的价值。

#### balanceOf(address account) → uint256
external#
返回账户拥有的代币的价值。

#### transfer(address to, uint256 value) → bool
external#
将一定数量的代币从调用者的账户转移到to。

返回一个布尔值，表示操作是否成功。

发出一个*transfer*事件。

#### allowance(address owner, address spender) → uint256
external#
返回spender将被允许代表所有者通过*transferFrom*花费的剩余代币数量。默认情况下，这个值是零。

当调用*approve*或*transferFrom*时，此值会改变。

#### approve(address spender, uint256 value) → bool
external#
将一定数量的代币设为调用者代币的支出者的额度。

返回一个布尔值，表示操作是否成功。

> IMPORTANT
请注意，使用此方法更改额度存在风险，因为有可能有人通过不幸的交易排序使用旧的和新的额度。缓解这种竞态条件的一种可能解决方案是首先将支出者的额度降低到0，然后设置所需的值：https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729

发出一个*Approval*事件。

#### transferFrom(address from, address to, uint256 value) → bool
external#
将一定数量的代币从一个账户转移到另一个账户，使用的是授权机制。然后，这个值将从调用者的授权额度中扣除。

返回一个布尔值，表示操作是否成功。

触发一个*transfer*事件。

#### Transfer(address indexed from, address indexed to, uint256 value)
event#
当价值代币从一个账户（from）移动到另一个账户（to）时发出。

请注意，价值可能为零。

#### Approval(address indexed owner, address indexed spender, uint256 value)
event#
当通过调用批准设置所有者对支出者的津贴时发出。值是新的津贴。

### [IERC20Metadata](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/IERC20Metadata.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol";
```

来自ERC20标准的可选元数据功能的接口。

**FUNCTIONS**
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
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### name() → string
external#
返回代币的名称。

#### symbol() → string
external#
返回代币的符号。

#### decimals() → uint8
external#
返回代币的小数位数。

#### [ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/ERC20.sol)
```
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

实现*IERC20*接口。

此实现对代币的创建方式不做任何假设。这意味着需要在派生合约中使用*_mint*添加供应机制。

> TIP
关于如何实现供应机制的详细说明，请参阅[我们的指南](https://forum.openzeppelin.com/t/how-to-implement-erc20-supply-mechanisms/226)。

*小数*的默认值是18。要更改这个值，你应该重写这个函数，使其返回一个不同的值。

我们遵循了OpenZeppelin合约的一般指南：函数在失败时会回滚而不是返回false。然而，这种行为是常规的，并不与ERC20应用的期望冲突。

此外，在调用*transferFrom*时会发出*Approval*事件。这允许应用程序仅通过监听这些事件就能重建所有账户的允许额度。EIP的其他实现可能不会发出这些事件，因为规范并未要求这样做。

**FUNCTIONS**
constructor(name_, symbol_)

name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### constructor(string name_, string symbol_)
internal#
设置*名称*和*符号*的值。

这两个值都是不可变的：它们只能在构造过程中设置一次。

#### name() → string
public#
返回代币的名称。

#### symbol() → string
public#
返回代币的符号，通常是名称的简短版本。

#### decimals() → uint8
public#
返回用于获取其用户表示形式的小数位数。例如，如果小数位数等于2，那么505个代币应显示为5.05（505 / 10 ** 2）。

代币通常选择18作为值，模仿以太坊和Wei之间的关系。这是此函数返回的默认值，除非它被覆盖。

> NOTE
这个信息仅用于显示目的：它不会以任何方式影响合约的算术运算，包括*IERC20.balanceOf*和*IERC20.transfer*。

#### totalSupply() → uint256
public#
请查阅 *IERC20.totalSupply*.

#### balanceOf(address account) → uint256
public#
请查阅 IERC20.balanceOf.

#### transfer(address to, uint256 value) → bool
public#
请查阅*IERC20.transfer*。

要求：
* 接收者不能是零地址。

* 调用者必须至少拥有等于转账值的余额。

#### allowance(address owner, address spender) → uint256
public#
请查阅 *IERC20.allowance*.

#### approve(address spender, uint256 value) → bool
public#
请查阅*IERC20.approve*。

> NOTE
如果值是最大的uint256，那么在transferFrom上不会更新额度。这在语义上等同于无限批准。

要求：
* spender不能是零地址。

#### transferFrom(address from, address to, uint256 value) → bool
public#
请查阅*IERC20.transferFrom*。

发出一个*Approval*事件，表示更新的额度。这不是EIP所要求的。请参阅*ERC20*开头的注释。

> NOTE
如果当前额度是最大的uint256，不更新额度。

要求：
* from和to不能是零地址。

* from必须至少有价值的余额。

* 调用者必须至少有对from的代币的额度。

#### _transfer(address from, address to, uint256 value)
internal#
将一定数量的代币从一个账户转移到另一个账户。

这个内部函数等同于*转账*，可以用来实现自动代币费用，削减机制等。

触发一个 *transfer*事件。

> NOTE
这个函数不是虚拟的，应该重写*_update*函数。

#### _update(address from, address to, uint256 value)
internal#
将一定数量的代币从from转移到to，或者如果from（或to）是零地址，则进行铸币（或销毁）。所有对转账、铸币和销毁的自定义都应通过覆盖此函数来完成。

发出一个*transfer*事件。

#### _mint(address account, uint256 value)
internal#
创建一定数量的代币并将其分配给账户，通过从地址(0)转移。依赖于_update机制

发出一个transfer事件，其中from设置为零地址。

> NOTE
这个函数不是虚拟的，应该重写*_update*。

#### _burn(address account, uint256 value)
internal#
从账户销毁一定数量的代币，降低总供应量。依赖于_update机制。

发出一个*transfer*事件，将to设置为零地址。

> NOTE
这个函数不是虚拟的，应该重写*_update*。

#### _approve(address owner, address spender, uint256 value)
internal#
将值设置为支出者对所有者代币的允许额度。

这个内部函数等同于批准，可以用来例如为某些子系统设置自动允许额度等。

发出一个*Approval*事件。

要求：
* 所有者不能是零地址。

* 支出者不能是零地址。

对这个逻辑的覆盖应该在有一个额外的布尔值emitEvent参数的变体中进行。

#### _approve(address owner, address spender, uint256 value, bool emitEvent)
internal#
批准的变体，带有一个可选标志，用于启用或禁用*Approval*事件。

默认情况下（当调用*_approve*时），标志设置为true。另一方面，由_spendAllowance在transferFrom操作中进行的批准更改将标志设置为false。这通过在transferFrom操作中不发出任何批准事件来节省gas。

任何希望在transferFrom操作上继续发出批准事件的人可以使用以下覆盖强制标志为true：
```
function _approve(address owner, address spender, uint256 value, bool) internal virtual override {
    super._approve(owner, spender, value, true);
}
```
要求与*_approve*相同。

## Extensions

### [IERC20Permit](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/IERC20Permit.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/IERC20Permit.sol";
```

ERC20许可扩展的接口允许通过签名进行批准，如[EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)中定义。

添加了*permit*方法，可以通过呈现账户签名的消息来更改账户的ERC20额度（参见*IERC20.allowance*）。通过不依赖*IERC20.approve*，代币持有者账户不需要发送交易，因此不需要持有以太。

#### 安全考虑
关于使用许可证，有两个重要的考虑因素。第一个是，有效的许可签名表示一个额度，不应假设它传达了额外的含义。特别是，不应将其视为以任何特定方式使用额度的意图。第二个是，由于许可证具有内置的重播保护，并且可以由任何人提交，因此它们可能会被提前运行。使用许可证的协议应考虑到这一点，并允许许可证调用失败。结合这两个方面，可能推荐的一种模式是:
```
function doThingWithPermit(..., uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s) public {
    try token.permit(msg.sender, address(this), value, deadline, v, r, s) {} catch {}
    doThing(..., value);
}

function doThing(..., uint256 value) public {
    token.safeTransferFrom(msg.sender, address(this), value);
    ...
}
```

请注意：1）msg.sender被用作所有者，清楚地表明了签名者的意图，2）使用try/catch允许许可失败，并使代码能够容忍操纵交易。 （也可以参见*SafeERC20.safeTransferFrom*）。

另外，请注意，智能合约钱包（如Argent或Safe）无法生成许可签名，因此合约应具有不依赖许可的入口点。

**FUNCTIONS**
permit(owner, spender, value, deadline, v, r, s)

nonces(owner)

DOMAIN_SEPARATOR()

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
external#
将值设置为支出者对所有者代币的允许额度，前提是所有者已签署批准。

> IMPORTANT
*IERC20.approve*在事务排序方面存在的问题在这里也同样适用。

触发一个*Approval*事件。

要求：
* 支出者不能是零地址。

* 截止日期必须是未来的时间戳。

* v，r和s必须是所有者对EIP712格式化函数参数的有效secp256k1签名。

* 签名必须使用所有者当前的nonce（见*nonces*）。

有关签名格式的更多信息，请参阅[相关的EIP部分](https://eips.ethereum.org/EIPS/eip-2612#specification)。

> CAUTION
请参阅上面的安全性考虑。

#### nonces(address owner) → uint256
external#
返回所有者的当前随机数。每次生成*许可*签名时，都必须包含此值。

每次成功调用*许可*都会将所有者的随机数增加一。这防止了多次使用同一签名。

#### DOMAIN_SEPARATOR() → bytes32
external#
返回用于*许可*签名编码的域分隔符，如*EIP712*所定义。

### [ERC20Permit](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Permit.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Permit.sol";
```
实现ERC20许可扩展，允许通过签名进行批准，如[EIP-2612](https://eips.ethereum.org/EIPS/eip-2612)中定义。

添加了*permit*方法，可以通过呈现账户签名的消息来更改账户的ERC20额度（参见*IERC20.allowance*）。通过不依赖*IERC20.approve*，代币持有者账户不需要发送交易，因此完全不需要持有以太。

**FUNCTIONS**
constructor(name)

permit(owner, spender, value, deadline, v, r, s)

nonces(owner)

DOMAIN_SEPARATOR()

NONCES
_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC5267
EIP712DomainChanged()

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC2612ExpiredSignature(deadline)

ERC2612InvalidSigner(signer, owner)

NONCES
InvalidAccountNonce(account, currentNonce)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### constructor(string name)
internal#
使用名称参数初始化*EIP712*域分隔符，并将版本设置为"1"。

使用与ERC20代币名称定义相同的名称是个好主意。

#### permit(address owner, address spender, uint256 value, uint256 deadline, uint8 v, bytes32 r, bytes32 s)
public#
将值设置为支出者对所有者代币的允许额度，前提是所有者已签署批准。

> IMPORTANT
*IERC20.approve*在交易排序方面存在的问题在这里也同样适用。

触发一个*Approval*事件。

要求：
* 支出者不能是零地址。

* 截止日期必须是未来的时间戳。

* v、r和s必须是所有者对EIP712格式化函数参数的有效secp256k1签名。

* 签名必须使用所有者当前的nonce（参见*nonces*）。

有关签名格式的更多信息，请参阅[相关的EIP部分](https://eips.ethereum.org/EIPS/eip-2612#specification)。

> CAUTION
请参阅上面的安全性考虑。

#### nonces(address owner) → uint256
public#
返回所有者当前的随机数。每次生成*许可*签名时，都必须包含此值。

每次成功调用*许可*都会将所有者的随机数增加一。这防止了签名被多次使用。

#### DOMAIN_SEPARATOR() → bytes32
external#
返回用于*许可*签名编码的域分隔符，如*EIP712*所定义。

#### ERC2612ExpiredSignature(uint256 deadline)
error#
许可证的截止日期已过。

#### ERC2612InvalidSigner(address signer, address owner)
error#
签名不匹配。

### [ERC20Burnable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Burnable.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
```

扩展*ERC20*允许代币持有者销毁他们自己的代币和他们有权使用的代币，这种方式可以在链下（通过事件分析）被识别。

**FUNCTIONS**
burn(value)

burnFrom(account, value)

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### burn(uint256 value)
public#
销毁来自调用者的一定数量的代币。

请查阅*ERC20._burn*。

#### burnFrom(address account, uint256 value)
public#
从账户中销毁一定数量的代币，从调用者的额度中扣除。

参见*ERC20._burn*和*ERC20.allowance*。

要求：
* 调用者必须至少拥有账户代币的额度。

### [ERC20Capped](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Capped.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Capped.sol";
```

扩展ERC20，增加了对代币供应的上限。

**FUNCTIONS**
constructor(cap_)

cap()

_update(from, to, value)

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC20ExceededCap(increasedSupply, cap)

ERC20InvalidCap(cap)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### constructor(uint256 cap_)
internal#
设置上限值。这个值是不可变的，只能在构造过程中设置一次。

#### cap() → uint256
public#
返回代币总供应量的上限。

#### _update(address from, address to, uint256 value)
internal#
请查阅 *ERC20._update*.

#### ERC20ExceededCap(uint256 increasedSupply, uint256 cap)
error#
已超过总供应上限。

#### ERC20InvalidCap(uint256 cap)
error#
提供的帽子不是有效的帽子。

### [ERC20Pausable](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Pausable.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Pausable.sol";
```

ERC20代币具有可暂停的代币转移、铸币和销毁功能。

这对于防止在评估期结束之前进行交易，或者在出现大错误时冻结所有代币转移的紧急开关等场景非常有用。

> IMPORTANT
该合约并未包含公开的暂停和恢复功能。除了继承这个合约，你还必须定义这两个功能，调用*Pausable._pause*和*Pausable._unpause*内部功能，并配以适当的访问控制，例如使用*AccessControl*或*Ownable*。如果不这样做，将使合约的暂停机制无法达到，从而无法使用。

**FUNCTIONS**
_update(from, to, value)

PAUSABLE
paused()

_requireNotPaused()

_requirePaused()

_pause()

_unpause()

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
PAUSABLE
Paused(account)

Unpaused(account)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
PAUSABLE
EnforcedPause()

ExpectedPause()

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### _update(address from, address to, uint256 value)
internal#
请查阅*ERC20._update*。

要求：
* 合约不能被暂停。

### [ERC20Votes](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Votes.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
```

这是ERC20的扩展，支持Compound-like的投票和委托。这个版本比Compound的更通用，并且支持的代币供应量高达2^208-1 ，而COMP的限制是2^96-1。

> NOTE
这个合约并不提供与Compound的COMP代币的接口兼容性。

这个扩展保留了每个账户投票权力的历史（检查点）。投票权力可以通过直接调用{delegate}函数，或者提供一个签名来使用{delegateBySig}进行委托。投票权力可以通过公开访问器{getVotes}和{getPastVotes}进行查询。

默认情况下，代币余额不考虑投票权力。这使得转账更便宜。缺点是，它要求用户委托给自己，以激活检查点并跟踪他们的投票权力。

**FUNCTIONS**
_maxSupply()

_update(from, to, value)

_getVotingUnits(account)

numCheckpoints(account)

checkpoints(account, pos)

VOTES
clock()

CLOCK_MODE()

getVotes(account)

getPastVotes(account, timepoint)

getPastTotalSupply(timepoint)

_getTotalSupply()

delegates(account)

delegate(delegatee)

delegateBySig(delegatee, nonce, expiry, v, r, s)

_delegate(account, delegatee)

_transferVotingUnits(from, to, amount)

_numCheckpoints(account)

_checkpoints(account, pos)

NONCES
nonces(owner)

_useNonce(owner)

_useCheckedNonce(owner, nonce)

EIP712
_domainSeparatorV4()

_hashTypedDataV4(structHash)

eip712Domain()

_EIP712Name()

_EIP712Version()

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IVOTES
DelegateChanged(delegator, fromDelegate, toDelegate)

DelegateVotesChanged(delegate, previousVotes, newVotes)

IERC5267
EIP712DomainChanged()

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC20ExceededSafeSupply(increasedSupply, cap)

VOTES
ERC6372InconsistentClock()

ERC5805FutureLookup(timepoint, clock)

IVOTES
VotesExpiredSignature(expiry)

NONCES
InvalidAccountNonce(account, currentNonce)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### _maxSupply() → uint256
internal#
最大代币供应量默认为type(uint208).max (2^208-1)。

这个最大值在*_update*中被强制执行。它限制了代币的总供应量，否则是一个uint256，因此可以在{*Votes*}使用的Trace208结构中存储检查点。增加这个值不会消除底层的限制，并且会导致*_update*因为在{_transferVotingUnits}中的数学溢出而失败。如果需要额外的逻辑，可以使用覆盖来进一步限制总供应量（到一个较低的值）。在解决此函数上的覆盖冲突时，应返回最小值。

#### _update(address from, address to, uint256 value)
internal#
当代币转移时移动投票权。

发出一个*IVotes.DelegateVotesChanged*事件。

#### _getVotingUnits(address account) → uint256
internal#
返回账户的投票单位。

> WARNING
重写此功能可能会破坏内部投票账目。 ERC20Votes假设代币以1:1的比例映射到投票单位，这不容易改变。

#### numCheckpoints(address account) → uint32
public#
获取帐户的检查点数目。

#### checkpoints(address account, uint32 pos) → struct Checkpoints.Checkpoint208
public#
获取账户的pos位置的检查点。

#### ERC20ExceededSafeSupply(uint256 increasedSupply, uint256 cap)
error#
供应上限已经超过，引入了票数溢出的风险。

### [ERC20Wrapper](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20Wrapper.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Wrapper.sol";
```

扩展ERC20代币合约以支持代币包装。

用户可以存入和取出"基础代币"，并获得相应数量的"包装代币"。这在与其他模块结合使用时非常有用。例如，将这种包装机制与*ERC20Votes*结合，将允许将现有的"基础" ERC20包装成治理代币。

**FUNCTIONS**
constructor(underlyingToken)

decimals()

underlying()

depositFor(account, value)

withdrawTo(account, value)

_recover(account)

ERC20
name()

symbol()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC20InvalidUnderlying(token)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### constructor(contract IERC20 underlyingToken)
internal#

#### decimals() → uint8
public#
请查阅 *ERC20.decimals*.

#### underlying() → contract IERC20
public#
返回被封装的底层ERC-20代币的地址。

#### depositFor(address account, uint256 value) → bool
public#
允许用户存入基础代币，并铸造相应数量的包装代币。

#### withdrawTo(address account, uint256 value) → bool
public#
允许用户销毁一定数量的封装代币，并提取相应数量的基础代币。

#### _recover(address account) → uint256
internal#
创建包装代币以覆盖任何可能被误转的底层代币。这是一个内部函数，如果需要，可以通过访问控制进行公开。

#### ERC20InvalidUnderlying(address token)
error#
底层的令牌无法被封装。

### [ERC20FlashMint](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC20FlashMint.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20FlashMint.sol";
```

实现ERC3156闪电贷款扩展，如[ERC-3156](https://eips.ethereum.org/EIPS/eip-3156)中定义。

添加了*flashLoan*方法，该方法在代币级别提供闪电贷款支持。默认情况下没有费用，但可以通过覆盖*flashFee*来更改。

当这个扩展与*ERC20Capped*或*ERC20Votes*扩展一起使用时，*maxFlashLoan*将不能正确反映可以闪电铸造的最大值。我们建议覆盖*maxFlashLoan*，使其正确反映供应上限。

**FUNCTIONS**
maxFlashLoan(token)

flashFee(token, value)

_flashFee(token, value)

_flashFeeReceiver()

flashLoan(receiver, token, value, data)

ERC20
name()

symbol()

decimals()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC3156UnsupportedToken(token)

ERC3156ExceededMaxLoan(maxLoan)

ERC3156InvalidReceiver(receiver)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### maxFlashLoan(address token) → uint256
public#
返回可借贷的最大代币数量。

#### flashFee(address token, uint256 value) → uint256
public#
返回执行闪电贷款时应用的费用。此函数调用 *_flashFee* 函数，该函数返回执行闪电贷款时应用的费用。

#### _flashFee(address token, uint256 value) → uint256
internal#
返回执行闪电贷款时应用的费用。默认情况下，此实现没有任何费用。这个函数可以被重载，使闪电贷款机制变为通缩的。

#### _flashFeeReceiver() → address
internal#
返回闪电费的接收地址。默认情况下，此实现返回地址(0)，这意味着费用金额将被销毁。可以重载此函数以更改费用接收者。

#### flashLoan(contract IERC3156FlashBorrower receiver, address token, uint256 value, bytes data) → bool
public#
执行闪电贷款。新的代币被铸造并发送给接收者，接收者需要实现*IERC3156FlashBorrower*接口。在闪电贷款结束时，接收者预计将拥有价值+费用的代币，并已经得到批准返回到代币合约本身，以便它们可以被销毁。

#### ERC3156UnsupportedToken(address token)
error#
贷款令牌无效。

#### 超过最大贷款限额(uint256最大贷款)
error#
请求的贷款超过了代币的最大贷款值。

#### ERC3156InvalidReceiver(address receiver)
error#
闪电贷的接收者不是一个有效的{onFlashLoan}实现者。

### [ERC4626](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/extensions/ERC4626.sol)
```
import "@openzeppelin/contracts/token/ERC20/extensions/ERC4626.sol";
```

这是ERC4626 "Tokenized Vault Standard"的实现，定义在[EIP-4626](https://eips.ethereum.org/EIPS/eip-4626)中。

这个扩展允许通过标准化的*deposit*、*mint*、*redeem*和*burn*工作流程，以"资产"为交换铸造和销毁"份额"（使用ERC20继承表示）。这个合约扩展了ERC20标准。任何附加的扩展都会影响这个合约所代表的"份额"代币，而不是"资产"代币，后者是一个独立的合约。

> CAUTION
在空的（或几乎空的）ERC-4626保险库中，存款有很高的被盗风险，通过对保险库的"捐赠"进行前运行，这会使份额的价格膨胀。这被称为捐赠或通胀攻击，本质上是滑点问题。保险库部署者可以通过对资产进行大量的初始存款来防止这种攻击，使价格操纵变得不可行。取款也可能受到滑点的影响。用户可以通过验证收到的金额是否符合预期，使用执行这些检查的包装器如[ERC4626Router](https://github.com/fei-protocol/ERC4626#erc4626router-and-base)来防止这种攻击以及一般的意外滑点。
从v4.9开始，这个实现使用虚拟资产和份额来降低这种风险。_decimalsOffset()对应于底层资产的小数和保险库小数之间的小数表示的偏移。这个偏移也决定了保险库中虚拟份额和虚拟资产的比率，这本身决定了初始的兑换率。虽然这并不能完全防止攻击，但分析显示默认的偏移（0）使得攻击变得不利可图，因为虚拟份额捕获的价值（来自攻击者的捐赠）与攻击者的预期收益相匹配。如果偏移更大，攻击的成本将比其利润大几个数量级。关于底层数学的更多细节可以在*这里*找到。
这种方法的缺点是虚拟份额确实捕获了保险库增值的一小部分。此外，如果保险库遭受损失，用户试图退出保险库，虚拟份额和资产会导致第一个退出的用户损失减少，而最后的用户将承受更大的损失。希望回到v4.9之前行为的开发者只需要覆盖_convertToShares和_convertToAssets函数。
要了解更多信息，请查看我们的*ERC-4626指南*。

**FUNCTIONS**
constructor(asset_)

decimals()

asset()

totalAssets()

convertToShares(assets)

convertToAssets(shares)

maxDeposit()

maxMint()

maxWithdraw(owner)

maxRedeem(owner)

previewDeposit(assets)

previewMint(shares)

previewWithdraw(assets)

previewRedeem(shares)

deposit(assets, receiver)

mint(shares, receiver)

withdraw(assets, receiver, owner)

redeem(shares, receiver, owner)

_convertToShares(assets, rounding)

_convertToAssets(shares, rounding)

_deposit(caller, receiver, assets, shares)

_withdraw(caller, receiver, owner, assets, shares)

_decimalsOffset()

ERC20
name()

symbol()

totalSupply()

balanceOf(account)

transfer(to, value)

allowance(owner, spender)

approve(spender, value)

transferFrom(from, to, value)

_transfer(from, to, value)

_update(from, to, value)

_mint(account, value)

_burn(account, value)

_approve(owner, spender, value)

_approve(owner, spender, value, emitEvent)

_spendAllowance(owner, spender, value)

**EVENTS**
IERC4626
Deposit(sender, owner, assets, shares)

Withdraw(sender, receiver, owner, assets, shares)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

**ERRORS**
ERC4626ExceededMaxDeposit(receiver, assets, max)

ERC4626ExceededMaxMint(receiver, shares, max)

ERC4626ExceededMaxWithdraw(owner, assets, max)

ERC4626ExceededMaxRedeem(owner, shares, max)

IERC20ERRORS
ERC20InsufficientBalance(sender, balance, needed)

ERC20InvalidSender(sender)

ERC20InvalidReceiver(receiver)

ERC20InsufficientAllowance(spender, allowance, needed)

ERC20InvalidApprover(approver)

ERC20InvalidSpender(spender)

#### constructor(contract IERC20 asset_)
internal#
设置底层资产合约。这必须是一个与ERC20兼容的合约（ERC20或ERC777）。

#### decimals() → uint8
小数是通过在底层资产的小数上加上小数偏移来计算的。这个"原始"值在创建保险库合约时被缓存。如果这个读取操作失败（例如，资产还没有被创建），则默认使用18来表示底层资产的小数。

请查阅*IERC20Metadata.decimals*。

#### asset() → address
public#
请查阅 *IERC4626.asset*.

#### totalAssets() → uint256
public#
请查阅 *IERC4626.totalAssets*.

#### convertToShares(uint256 assets) → uint256
public#
请查阅 *IERC4626.convertToShares*.

#### convertToAssets(uint256 shares) → uint256
public#
请查阅 *IERC4626.convertToAssets*.

#### maxDeposit(address) → uint256
public#
请查阅 *IERC4626.maxDeposit*.

#### maxMint(address) → uint256
public#
请查阅 *IERC4626.maxMint*.

#### maxWithdraw(address owner) → uint256
public#
请查阅 *IERC4626.maxWithdraw*.

#### maxRedeem(address owner) → uint256
public#
请查阅 *IERC4626.maxRedeem*.

#### previewDeposit(uint256 assets) → uint256
public#
请查阅 *IERC4626.previewDeposit*.

#### previewMint(uint256 shares) → uint256
public#
请查阅 *IERC4626.previewMint*.

#### previewWithdraw(uint256 assets) → uint256
public#
请查阅 IERC4626.previewWithdraw.

#### previewRedeem(uint256 shares) → uint256
public#
请查阅 IERC4626.previewRedeem.

#### deposit(uint256 assets, address receiver) → uint256
public#
请查阅 IERC4626.deposit.

#### mint(uint256 shares, address receiver) → uint256
public#
请查阅*IERC4626.mint*。

与存款不同，即使保险库处于股份价格为零的状态，也允许铸造。在这种情况下，可以在不存入任何资产的情况下铸造股份。

#### withdraw(uint256 assets, address receiver, address owner) → uint256
public#
请查阅 *IERC4626.withdraw*.

#### redeem(uint256 shares, address receiver, address owner) → uint256
public#
请查阅 *IERC4626.redeem*.

#### _convertToShares(uint256 assets, enum Math.Rounding rounding) → uint256
internal#
支持取舍方向的资产转股份的内部转换功能。

#### _convertToAssets(uint256 shares, enum Math.Rounding rounding) → uint256
internal#
内部转换函数（从股份到资产），支持舍入方向。

#### _deposit(address caller, address receiver, uint256 assets, uint256 shares)
internal#
存款/铸币常见工作流程。

#### _withdraw(address caller, address receiver, address owner, uint256 assets, uint256 shares)
internal#
常见的提现/兑换工作流程。

#### _decimalsOffset() → uint8
internal#

#### ERC4626ExceededMaxDeposit(address receiver, uint256 assets, uint256 max)
error#
尝试存入的资产超过了接收者的最大金额。

#### ERC4626ExceededMaxMint(address receiver, uint256 shares, uint256 max)
error#
尝试为接收者铸造超过最大数量的股份。

#### ERC4626ExceededMaxWithdraw(address owner, uint256 assets, uint256 max)
error#
尝试提取超过接收者的最大金额的资产。

#### ERC4626ExceededMaxRedeem(address owner, uint256 shares, uint256 max)
error#
尝试兑换的股份超过接收者的最大数量。

## Utilities

### [SafeERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/token/ERC20/utils/SafeERC20.sol)
```
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```

这是一个在ERC20操作失败时抛出异常的封装器（当代币合约返回false时）。也支持返回无值的代币（在失败时反转或抛出异常），并假定非反转调用是成功的。要使用这个库，你可以在你的合约中添加一个使用SafeERC20 for IERC20;的声明，这样你就可以调用安全操作，如token.safeTransfer(…​)等。

**FUNCTIONS**
safeTransfer(token, to, value)

safeTransferFrom(token, from, to, value)

safeIncreaseAllowance(token, spender, value)

safeDecreaseAllowance(token, spender, requestedDecrease)

forceApprove(token, spender, value)

**ERRORS**
SafeERC20FailedOperation(token)

SafeERC20FailedDecreaseAllowance(spender, currentAllowance, requestedDecrease)

#### safeTransfer(contract IERC20 token, address to, uint256 value)
internal#
将代币的转账值从调用合约转到to。如果代币没有返回值，那么不会撤销的调用被认为是成功的。

#### safeTransferFrom(contract IERC20 token, address from, address to, uint256 value)
internal#
将代币的转账值从“from”转移到“to”，消耗“from”给予调用合约的批准。如果代币没有返回值，那么不会撤销的调用被认为是成功的。

#### safeIncreaseAllowance(contract IERC20 token, address spender, uint256 value)
internal#
增加呼叫合约对支出者的额度值。如果代币没有返回值，那么不会撤销的调用被认为是成功的。

#### safeDecreaseAllowance(contract IERC20 token, address spender, uint256 requestedDecrease)
internal#
将调用合约对支出者的允许额度减少requestedDecrease。如果代币没有返回值，那么不会撤销的调用被认为是成功的。

#### forceApprove(contract IERC20 token, address spender, uint256 value)
internal#
将呼叫合约的允许额度设置为向花费者的值。如果代币没有返回值，那么不会撤销的呼叫被认为是成功的。这是为了配合那些需要在设置为非零值之前将批准设置为零的代币（如USDT）而设计的。

#### SafeERC20FailedOperation(address token)
error#
一个ERC20代币的操作失败了。

#### SafeERC20FailedDecreaseAllowance(address spender, uint256 currentAllowance, uint256 requestedDecrease)
error#
表示减少许可请求失败。