# API Reference
这份文档还在进行编辑中：如果有疑问，请前往[tests目录](https://github.com/OpenZeppelin/openzeppelin-test-helpers/tree/master/test/src)查看每个辅助函数的使用示例。

所有返回的数字都是[BN](https://github.com/indutny/bn.js)类型。

## balance
用于检查特定账户的以太余额的辅助函数。

所有这些函数都返回BN实例，默认以'wei'为单位。

### current
```
async function balance.current(account, unit = 'wei')
```
返回一个账户的当前余额。

const balance = await balance.current(account)
// 等同于 new BN(web3.eth.getBalance(account))

```
const balanceEth = await balance.current(account, 'ether')
// 等同于 new BN(web3.utils.fromWei(await web3.eth.getBalance(account), 'ether'))
```

### tracker
```
async function balance.tracker(account, unit = 'wei')
```
创建一个余额追踪器的实例，用于跟踪账户的以太余额变化。

```
const tracker = await balance.tracker(account)

```

#### tracker.get
```
async function tracker.get(unit = tracker.unit)
```
返回一个账户的当前余额。

```
const tracker = await balance.tracker(account) // 实例化
const currentBalance = await tracker.get() // 返回账户的当前余额
```

#### tracker.delta
```
async function tracker.delta(unit = tracker.unit)
```

返回自上次查询（使用get、delta或deltaWithFees）以来的余额变化。。
```
const tracker = await balance.tracker(receiver, 'ether')
send.ether(sender, receiver, ether('10'))
(await tracker.delta()).should.be.bignumber.equal('10');
(await tracker.delta()).should.be.bignumber.equal('0');
```

或者使用get():
```
const tracker = await balance.tracker(account) // 实例化
const currentBalance = await tracker.get() // 返回账户的当前余额
(await tracker.delta()).should.be.bignumber.equal('0');
```

一个追踪器还可以以特定单位返回所有余额和增量：

```
const tracker = await balance.tracker(account, 'gwei');
const balanceGwei = tracker.get(); // 以gigawei为单位
const balanceEther = tracker.get('ether'); // 以ether为单位
```

#### tracker.deltaWithFees
```
async function tracker.deltaWithFees(unit = tracker.unit)
```

返回一个包含自上次查询（使用get、delta或deltaWithFees）以来余额变化和此期间支付的燃气费用的对象。

```
const tracker = await balance.tracker(account, 'gwei');
...
const { delta, fees } = await tracker.deltaWithFees();
```

## BN
一个[bn.js](https://github.com/indutny/bn.js)对象。使用new BN(number)创建BN实例。

## 常量
一组有用的常量。

### ZERO_ADDRESS
地址类型变量的初始值，即Solidity中的address(0)。

### ZERO_BYTES32
bytes32类型变量的初始值，即Solidity中的bytes32(0x00)。

### MAX_UINT256
以BN表示的最大无符号整数2^256 - 1。

### MAX_INT256
以BN表示的最大有符号整数2^255 - 1。

### MIN_INT256
以BN表示的最小有符号整数-2^255。

## ether
将以太的值转换为wei。

## expectEvent
```
function expectEvent(receipt, eventName, eventArgs = {})
```
断言receipt中的日志包含一个名称为eventName的事件，并且参数与eventArgs中指定的参数匹配。receipt应该是由web3合约或truffle-contract调用返回的对象。

```
const web3Receipt = await MyWeb3Contract.methods.foo('bar').send();
expectEvent(web3Receipt, 'Foo', { value: 'bar' });

const truffleReceipt = await MyTruffleContract.foo('bar');
expectEvent(truffleReceipt, 'Foo', { value: 'bar' });
```

注意，可以不指定某些或所有事件参数：expectEvent只会检查提供的参数并忽略其余部分。
```
const receipt = await MyWeb3Contract.methods.foo('bar').send();

// 这两个断言都将通过
expectEvent(receipt, 'Foo', { value: 'bar' });
expectEvent(receipt, 'Foo');
```

### inTransaction
```
async function expectEvent.inTransaction(txHash, emitter, eventName, eventArgs = {})
```
和expectEvent一样，但是用于在任意事务中（txHash的哈希）由任意合约（emitter，合约实例）发出的事件，即使它是间接调用的（即它是由另一个智能合约而不是外部拥有的账户调用的）。

注意：emitter必须是发出预期事件的部署合约实例。
```
// 使用web3合约
const contract = await MyContract.deploy().send();
const { transactionHash } = await contract.methods.foo('bar').send();
await expectEvent.inTransaction(transactionHash, contract, 'Foo', { value: 'bar' });

// 使用truffle合约
const contract = await MyContract.new();
const { txHash } = await contract.foo('bar');
await expectEvent.inTransaction(txHash, contract, 'Foo', { value: 'bar' });
```

### inConstruction
```
async function expectEvent.inConstruction(emitter, eventName, eventArgs = {})
```
与inTransaction相同，但用于在emitter构造期间发出的事件。注意，目前仅支持truffle合约。

### notEmitted
为了测试事件是否未发出，可以使用expectEvent.notEmitted。有几个与前面提到的函数API相似的变体：

* expectEvent.notEmitted(receipt, eventName) 类似于expectEvent()

* expectEvent.notEmitted.inTransaction(txHash, emitter, eventName) 类似于expectEvent.inTransaction()

* expectEvent.notEmitted.inConstruction(emitter, eventName) 类似于expectEvent.inConstruction()

## expectRevert
```
async function expectRevert(promise, message)
```

用于事务失败的辅助函数（类似于[chai的throw](https://www.chaijs.com/api/bdd/#method_throw)）：断言promise由于事务失败而被拒绝。

它还会检查回滚原因是否包含message。当回滚原因未知时，请使用expectRevert.unspecified。

例如，给定以下合约：
```
contract Owned {
    address private _owner;

    constructor () {
        _owner = msg.sender;
    }

    function doOwnerOperation() public view {
        require(msg.sender == _owner, "Unauthorized");
        ....
    }
}
```

可以如下测试doOwnerOperation函数中的require语句：
```
const { expectRevert } = require('@openzeppelin/test-helpers');

const Owned = artifacts.require('Owned');

contract('Owned', ([owner, other]) => {
  beforeEach(async function () {
    this.owned = Owned.new({ from: owner });
  });

  describe('doOwnerOperation', function() {
    it('当由非所有者账户调用时失败', async function () {
      await expectRevert(
        this.owned.doOwnerOperation({ from: other }),
        "Unauthorized"
      );
    });
  });
  ...
```

### unspecified
```
async function expectRevert.unspecified(promise)
```
与expectRevert相同，断言promise由于require或revert语句导致的事务回滚而被拒绝，但不检查回滚原因。

### assertion
```
async function expectRevert.assertion(promise)
```
断言promise由于assert语句或无效操作码导致的事务回滚而被拒绝。

### outOfGas
```
async function expectRevert.outOfGas(promise)
```
断言promise由于事务耗尽了gas而被拒绝。

## makeInterfaceId

### ERC165
```
function makeInterfaceId.ERC165(interfaces = [])
```
计算合约的[ERC165](https://eips.ethereum.org/EIPS/eip-165)接口ID，给定一系列函数签名。

### ERC1820
```
function makeInterfaceId.ERC1820(name)
```
计算合约的[ERC1820](https://eips.ethereum.org/EIPS/eip-1820)接口哈希，给定其名称。

## send

### ether
```
async function send.ether(from, to, value)
```
从from发送value个以太到to。

### transaction
```
async function send.transaction(target, name, argsTypes, argsValues, opts = {})
```
发送一个交易到目标合约target，调用名称为name的方法，参数值为argValues，参数类型为argTypes（根据方法的签名）。

## singletons

### ERC1820Registry
```
async function singletons.ERC1820Registry(funder)
```
返回一个按照规范部署的[ERC1820Registry](https://eips.ethereum.org/EIPS/eip-1820)实例（即注册表位于规范地址）。可以多次调用以获取相同的实例。

## time

### advanceBlock
```
async function time.advanceBlock()
```
强制挖掘一个区块，增加区块高度。

### advanceBlockTo
```
async function time.advanceBlockTo(target)
```
强制挖掘区块，直到达到目标区块高度。

注意：使用此函数来提前太多的区块可能会导致测试变得非常缓慢。请尽量减少使用。

### latest
```
async function time.latest()
```
返回最新挖掘的区块的时间戳。应与advanceBlock配合使用以获取当前的区块链时间。

### latestBlock
```
async function time.latestBlock()
```
返回最新挖掘的区块号码。

### increase
```
async function time.increase(duration)
```
将区块链的时间增加[duration](#duration)（以秒为单位），并挖掘一个具有该时间戳的新区块。

### increaseTo
```
async function time.increaseTo(target)
```
与increase相同，但指定了目标时间而不是持续时间。

### duration
```
function time.duration()
```
将不同的时间单位转换为秒的辅助函数。可用的辅助函数有：seconds、minutes、hours、days、weeks和years。

```
await time.increase(time.duration.years(2));
```

### snapshot
```
async function snapshot()
```
返回一个带有“restore”函数的快照对象，该函数将区块链恢复到捕获的状态。

```
const snapshotA = await snapshot()
// ...
await snapshotA.restore()
```