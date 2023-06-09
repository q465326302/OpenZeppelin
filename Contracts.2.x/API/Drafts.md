# Drafts
此类别中的合约应被视为不稳定。它们与OpenZeppelin Contracts中的其他内容一样经过了彻底的审查，但我们对其API表示怀疑，因此我们不承诺向后兼容性。这意味着这些合约在次要版本中可能会发生重大变化，因此在升级时应特别注意变更日志。对于此类别之外的任何内容，您可以阅读更多关于*API稳定性*的信息。

> NOTE
此页面尚未完成。我们正在努力改进它以供下一版本发布时使用。敬请关注！

## ERC 20

### ERC20Migrator
此合约可用于将ERC20代币从一个合约迁移到另一个合约，其中每个代币持有者必须选择加入迁移。要选择加入，用户必须批准此合约要迁移的代币数量。一旦设置了津贴，任何人都可以触发迁移到新代币合约。通过这种方式，代币持有者将“转入”他们的旧余额，并将在新代币中铸造相等数量的代币。新代币合约必须是可铸造的。有关精确的接口，请参考OpenZeppelin的*ERC20Mintable*，但只需要的函数是*MinterRole.isMinter*和*ERC20Mintable.mint*。迁移者将检查它是否是代币的铸造者。从旧代币中迁移的余额将被转移到迁移者，并永远保留在那里。尽管此合约可以在许多不同的场景中使用，但主要动机是提供一种将ERC20代币迁移到使用ZeppelinOS的可升级版本的方法。要了解如何使用此实现进行迁移的更多信息，请参阅ZeppelinOS的官方文档网站：https://docs.zeppelinos.org/docs/erc20_onboarding.html
用法示例:
```
const migrator = await ERC20Migrator.new(legacyToken.address);
await newToken.addMinter(migrator.address);
await migrator.beginMigration(newToken.address);

```
**FUNCTIONS**
constructor(legacyToken)

legacyToken()

newToken()

beginMigration(newToken_)

migrate(account, amount)

migrateAll(account)

#### constructor(contract IERC20 legacyToken)
公开#

#### legacyToken() → contract IERC20
公开#
返回正在迁移的传统令牌。

#### newToken() → contract IERC20
公开#
返回我们正在迁移的新令牌。

#### beginMigration(contract ERC20Mintable newToken_)
公开#
通过设置将要铸造的新代币开始迁移。这个合约必须是新代币的铸造者。

#### migrate(address account, uint256 amount)
公开#
将账户中旧代币的部分余额转移到该合约，并为该账户铸造同等数量的新代币。

#### migrateAll(address account)
公开#
将账户的允许余额中的所有旧代币转移到该合约，并为该账户铸造相同数量的新代币。

### ERC20Snapshot
受Jordi Baylina的[MiniMeToken](https://github.com/Giveth/minimd/blob/ea04d950eea153a04c51fa510b068b9dded390cb/contracts/MiniMeToken.sol)的启发，记录历史余额。

当进行快照时，会记录快照时的余额和总供应量，以便以后访问。

要进行快照，请调用*Snapshot*函数，该函数将发出*Snapshot*事件并返回快照ID。要从快照中获取总供应量，请调用*totalSupplyAt*函数并提供快照ID。要从快照中获取账户余额，请调用*balanceOfAt*函数并提供快照ID和账户地址。

**FUNCTIONS**
snapshot()

balanceOfAt(account, snapshotId)

totalSupplyAt(snapshotId)

_transfer(from, to, value)

_mint(account, value)

_burn(account, value)

ERC20
totalSupply()

balanceOf(account)

transfer(recipient, amount)

allowance(owner, spender)

approve(spender, amount)

transferFrom(sender, recipient, amount)

increaseAllowance(spender, addedValue)

decreaseAllowance(spender, subtractedValue)

_approve(owner, spender, amount)

_burnFrom(account, amount)

CONTEXT
constructor()

_msgSender()

_msgData()

**EVENTS**
Snapshot(id)

IERC20
Transfer(from, to, value)

Approval(owner, spender, value)

#### snapshot() → uint256
公开#

#### balanceOfAt(address account, uint256 snapshotId) → uint256
公开#

#### totalSupplyAt(uint256 snapshotId) → uint256
公开#

#### _transfer(address from, address to, uint256 value)
内部#

#### _mint(address account, uint256 value)
内部#

#### _burn(address account, uint256 value)
内部#

#### Snapshot(uint256 id)
事件#

### TokenVesting
一个代币持有者合约，可以像典型的锁定期计划一样逐步释放其代币余额，包括一个起始期和锁定期。可由所有者选择性地撤销。

**MODIFIERS**
OWNABLE
onlyOwner()

**FUNCTIONS**
constructor(beneficiary, start, cliffDuration, duration, revocable)

beneficiary()

cliff()

start()

duration()

revocable()

released(token)

revoked(token)

release(token)

revoke(token)

OWNABLE
owner()

isOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
TokensReleased(token, amount)

TokenVestingRevoked(token)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

#### constructor(address beneficiary, uint256 start, uint256 cliffDuration, uint256 duration, bool revocable)
公开#
创建一个分期释放合约，将其ERC20代币的余额逐渐线性释放给受益人，直到开始时间+持续时间为止。到那时，所有余额都将被释放完毕。

#### beneficiary() → address
公开#

#### cliff() → uint256
公开#

#### start() → uint256
公开#

#### duration() → uint256
公开#

#### revocable() → bool
公开#

#### released(address token) → uint256
公开#

#### revoked(address token) → bool
公开#

#### release(contract IERC20 token)
公开#

#### revoke(contract IERC20 token)
公开#

#### revoke(contract IERC20 token)
事件#

#### TokenVestingRevoked(address token)
事件#

## Miscellaneous

### Counters
提供只能逐一递增或递减的计数器。这可以用于跟踪映射中的元素数量、发行ERC721 id或计数请求id。

在使用Counters for Counters.Counter时，请包含以下内容；由于通过递增一无法使256位整数溢出，因此递增可以跳过*SafeMath*溢出检查，从而节省燃气。但是，这假设正确使用，即永远不直接访问底层的_value。

**FUNCTIONS**
current(counter)

increment(counter)

decrement(counter)

#### current(struct Counters.Counter counter) → uint256
内部#

#### increment(struct Counters.Counter counter)
内部#

#### decrement(struct Counters.Counter counter)
内部#

### SignedSafeMath
带有安全检查并在出错时回滚的签名数学操作。

**FUNCTIONS**
mul(a, b)

div(a, b)

sub(a, b)

add(a, b)

#### mul(int256 a, int256 b) → int256
内部#
乘以两个有符号整数，溢出时返回。

#### div(int256 a, int256 b) → int256
内部#
两个有符号整数的整数除法，截断商，除以零时会回退。

#### sub(int256 a, int256 b) → int256
内部#
减去两个有符号整数，溢出时回滚。

#### add(int256 a, int256 b) → int256
内部#
将两个有符号整数相加，溢出时返回反向值。

## ERC 1046

