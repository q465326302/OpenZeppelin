# Using with Truffle
该软件包为你的Truffle迁移和测试添加了功能，以便你可以为你的合约部署和升级代理。

> NOTE
目前还不支持从[Truffle外部脚本](https://www.trufflesuite.com/docs/truffle/getting-started/writing-external-scripts)中使用。

> TIP
请查看[逐步教程](https://forum.openzeppelin.com/t/openzeppelin-truffle-upgrades-step-by-step-tutorial/3579)，从创建、测试和部署开始，一直到使用Gnosis Safe进行升级。

## 安装

```
npm install --save-dev @openzeppelin/truffle-upgrades
```
此包要求使用Truffle [5.1.35版本](https://github.com/trufflesuite/truffle/releases/tag/v5.1.35)或更高。

### 在迁移中的使用

#### 代理
要在迁移中部署一个可升级的合约实例，请使用deployProxy函数：
```
// migrations/NN_deploy_upgradeable_box.js
const { deployProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  const instance = await deployProxy(Box, [42], { deployer });
  console.log('Deployed', instance.address);
};
```

这将自动检查Box合约是否可升级，如果需要的话，设置代理管理员，部署一个Box合约的实现合约（除非先前已经有一个），创建一个代理，并通过调用initialize(42)来初始化它。

然后，在将来的迁移中，你可以使用upgradeProxy函数将部署的实例升级到新版本。新版本可以是一个不同的合约（比如BoxV2），或者你可以只是修改现有的Box合约并重新编译它-插件会注意到它的变化。
```
// migrations/MM_upgrade_box_contract.js
const { upgradeProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

module.exports = async function (deployer) {
  const existing = await Box.deployed();
  const instance = await upgradeProxy(existing.address, BoxV2, { deployer });
  console.log("Upgraded", instance.address);
};
```

插件将负责比较BoxV2与先前版本，以确保它们在升级时兼容，部署新的BoxV2实现合约（除非已经存在先前的部署），并将现有的代理升级到新的实现。

### Beacon代理

你还可以使用此插件通过deployBeacon函数部署一个可升级的beacon合约，然后通过使用deployBeaconProxy函数部署一个或多个指向该beacon的beacon代理。

```
// migrations/NN_deploy_upgradeable_box.js
const { deployBeacon, deployBeaconProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');

module.exports = async function (deployer) {
  const beacon = await deployBeacon(Box);
  console.log('Beacon deployed', beacon.address);

  const instance = await deployBeaconProxy(beacon, Box, [42], { deployer });
  console.log('Box deployed', instance.address);
};
```

在未来的迁移中，你可以使用erc1967.getBeaconAddress函数从部署的beacon代理实例中获取beacon地址，然后调用upgradeBeacon函数将该beacon升级到新版本。当beacon升级时，所有指向它的beacon代理将使用新的合约实现。

```
// migrations/MM_upgrade_box_contract.js
const { erc1967, upgradeBeacon } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

module.exports = async function (deployer) {
  const existing = await Box.deployed();

  const beaconAddress = await erc1967.getBeaconAddress(existing.address);
  await upgradeBeacon(beaconAddress, BoxV2, { deployer });
  console.log("Beacon upgraded", beaconAddress);

  const instance = await BoxV2.at(existing.address);
};
```

## 在测试中的使用

如果你想为升级合约添加测试的话（你应该这样做！），你也可以在Truffle测试中使用插件的函数。API与迁移中的API相同，只是没有部署参数。

### 代理
```
const { deployProxy, upgradeProxy } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

describe('upgrades', () => {
  it('works', async () => {
    const box = await deployProxy(Box, [42]);
    const box2 = await upgradeProxy(box.address, BoxV2);

    const value = await box2.value();
    assert.equal(value.toString(), '42');
  });
});
```

### Beacon代理
```
const { deployBeacon, deployBeaconProxy, upgradeBeacon } = require('@openzeppelin/truffle-upgrades');

const Box = artifacts.require('Box');
const BoxV2 = artifacts.require('BoxV2');

describe('upgrades', () => {
  it('works', async () => {
    const beacon = await deployBeacon(Box);
    const box = await deployBeaconProxy(beacon, Box, [42]);

    await upgradeBeacon(beacon, BoxV2);
    const box2 = await BoxV2.at(box.address);

    const value = await box2.value();
    assert.equal(value.toString(), '42');
  });
});
```
