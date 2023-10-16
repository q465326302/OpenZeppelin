# Finance
这个目录包含了金融系统的基本元素：

* VestingWallet处理给定受益人的Ether和ERC20代币的归属权。可以将多种代币的保管权交给这个合约，该合约将按照给定的、可定制的归属时间表将代币释放给受益人。

## Contracts

### [VestingWallet](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/v5.0.0/contracts/finance/VestingWallet.sol)
```
import "@openzeppelin/contracts/finance/VestingWallet.sol";
```
归属钱包是一个可拥有的合约，可以接收本地货币和ERC20代币，并根据归属时间表将这些资产释放给钱包所有者，也称为"受益人"。

任何转移到这个合约的资产都将按照从一开始就被锁定的归属时间表进行。因此，如果归属已经开始，任何发送到这个合约的代币数量都将（至少部分）立即可释放。

通过将持续时间设置为0，可以将此合约配置为一个资产定时锁，该锁将代币保留给受益人，直到指定的时间。

> NOTE
由于钱包是可拥有的，且*所有权*可以转移，因此可以出售未归属的代币。在智能合约中防止这种情况是困难的，考虑到：1）受益人地址可能是一个反事实部署的合约，2）在不久的将来，EOAs可能会有成为合约的迁移路径。

> NOTE
在使用这个合约与任何其余额自动调整的代币（即，一个重组代币）时，确保在归属时间表中考虑供应/余额调整，以确保归属金额如预期那样。

**FUNCTIONS**
constructor(beneficiary, startTimestamp, durationSeconds)

receive()

start()

duration()

end()

released()

released(token)

releasable()

releasable(token)

release()

release(token)

vestedAmount(timestamp)

vestedAmount(token, timestamp)

_vestingSchedule(totalAllocation, timestamp)

**OWNABLE**
owner()

_checkOwner()

renounceOwnership()

transferOwnership(newOwner)

_transferOwnership(newOwner)

EVENTS
EtherReleased(amount)

ERC20Released(token, amount)

OWNABLE
OwnershipTransferred(previousOwner, newOwner)

**ERRORS**
OWNABLE
OwnableUnauthorizedAccount(account)

OwnableInvalidOwner(owner)

#### constructor(address beneficiary, uint64 startTimestamp, uint64 durationSeconds)
public#
将发送者设置为初始所有者，受益人设置为待定所有者，设置待释放钱包的开始时间戳和待释放期限。

#### receive()
external#
合约应该能够接收以太。

#### start() → uint256
public#
获取开始时间戳的方法。

#### duration() → uint256
public#
获取待归属期限的方法。

#### end() → uint256
public#
获取结束时间戳。

#### released() → uint256
public#
已经释放的以太数量

#### released(address token) → uint256
public#
已经发布的代币数量

#### releasable() → uint256
public#
获取可释放以太的数量。

#### releasable(address token) → uint256
public#
获取可释放代币数量的获取器。代币应该是IERC20合约的地址。

#### release()
public#
释放已经归属的本地代币（以太）。

发出一个*EtherReleased*事件。

#### vestedAmount(uint64 timestamp) → uint256
public#
计算已经归属的以太数量。默认实现是线性归属曲线。

#### vestedAmount(address token, uint64 timestamp) → uint256
public#
计算已经归属的代币数量。默认实现是线性归属曲线。

#### _vestingSchedule(uint256 totalAllocation, uint64 timestamp) → uint256
internal#
虚拟实现归属权公式。这是一个随时间变化的函数，返回资产的归属金额，该金额是根据资产的历史总分配量计算得出的。

#### EtherReleased(uint256 amount)
event#

#### ERC20Released(address indexed token, uint256 amount)
event#