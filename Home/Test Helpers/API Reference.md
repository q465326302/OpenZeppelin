# API Reference
这份文档还在进行编辑中：如果有疑问，请前往[tests目录](https://github.com/OpenZeppelin/openzeppelin-test-helpers/tree/master/test/src)查看每个辅助函数的使用示例。

所有返回的数字都是[BN](https://github.com/indutny/bn.js)类型。

## balance
用于检查特定账户的以太币余额的辅助函数。

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
创建一个余额追踪器的实例，用于跟踪账户的以太币余额变化。

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
返回自上次查询余额（使用get、delta或deltaWithFees）以来余额的变化。

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

追踪器还可以以特定单位返回所有余额和变化：

```
const tracker = await balance.tracker(account, 'gwei');
const balanceGwei = tracker.get(); // 以gigawei为单位
const balanceEther = tracker.get('ether'); // 以ether为单位
```

#### tracker.deltaWithFees
```
async function tracker.deltaWithFees(unit = tracker.unit)
```


返回一个包含自上次查询余额（使用get、delta或deltaWithFees）以来余额变化和支付的燃气费用的对象。

const tracker = await balance.tracker(account, 'gwei');
...
const { delta, fees } = await tracker.deltaWithFees();
BN
一个bn.js对象。使用new BN(number)创建BN实例。