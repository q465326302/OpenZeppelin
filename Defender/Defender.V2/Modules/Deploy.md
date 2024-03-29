# Deploy
Deploy允许你跨链安全地部署和升级智能合约。你可以证明链上运行的代码与经过审计的实现相匹配，并最小化可能导致损失或问题的关键错误。

## Use cases
* 使用精细度配置生产和测试环境中的部署者地址。
* 自动支付部署的gas。
* 确保所有智能合约部署在区块浏览器上得到验证。
* 从一个地方管理多链部署。
* 自动化多签名批准流程以进行升级。
* 与CI/CD完全兼容，用于自动化发布。
* 跟踪每次部署或升级的详细信息和历史记录。

## Environments
Deploy分为生产和测试环境，前者使用主网网络，后者使用测试网网络。每个环境都与其类型相对应的一个或多个网络关联，这样可以安全地将测试合约与生产合约分开。为了便于入门，每个环境的设置都通过向导进行配置，该向导将快速引导你完成以下步骤。

### Wizard
#### Step 1: Networks
在第一步中，你选择要使用的网络。Deploy支持多链部署，允许用户将合约部署到多个网络上的相同地址。你可以随时通过环境页面添加或删除网络。

> NOTE
为了保持最佳安全实践，向导将仅显示根据所选环境类型的网络，以防止混合网络。

#### Step 2: Deploying
在这一步中，用户为每个网络选择默认的批准流程。与此批准流程关联的资源将用于支付和执行部署交易；然而，你可以在Hardhat脚本中为每次部署指定不同的批准流程。如果你没有网络的批准流程，向导将允许你创建Relayer、Safe或EOA（“外部拥有的账户”）类型之一。你可以在[这里](../Manage/Manage.md#approval-processes)了解更多关于批准流程的信息。

#### Step 3: Upgrading
这最后一步是可选的，但如果你计划部署可升级的合约，建议执行此步骤。在[这里](../Manage/Manage.md#approval-processes)，你为每个网络选择升级的批准流程。你可以在这里找到支持的流程列表。

## Environment Page
进入环境后，你可以查看环境内部署的配置和活动。

### Configuration
你可以编辑环境中任何网络的配置。为此，请点击编辑按钮进入配置页面，在那里你可以添加或删除网络并更改默认批准流程。我们使用系统区块浏览器API密钥自动在Etherscan上验证合约。你可以在配置页面管理你自己的密钥。

### History
历史表显示了环境内的活动，允许你检查部署或升级的状态。有三种可能的状态：

* COMPLETE：部署已在网络上成功执行。

* PENDING：升级正在等待批准。

* FAILED：部署失败。确保与批准流程关联的中继器或EOA有足够的资金，并且智能合约有效。

## Deploying and Upgrading
配置环境后，你可以用它进行部署和升级。为此，我们提供了一个[API](https://www.npmjs.com/package/@openzeppelin/defender-sdk-deploy-client)和一个[Hardhat插件](https://www.npmjs.com/package/@openzeppelin/hardhat-upgrades)。Deploy API推荐用于非Hardhat项目。另一方面，Hardhat插件在Hardhat项目中非常容易使用，因为它只需要几行代码就可以实现。

API和Hardhat插件都允许为每个合约部署使用不同的批准流程。然而，如果没有指定，Defender 2.0将识别目标网络并使用关联的默认批准流程。如果环境配置了区块浏览器API密钥，Defender 2.0还将自动验证合约。

> NOTE
我们提供了一个快速入门教程，使用Defender 2.0部署和升级智能合约。在[这里](../Tutorials/Deploy/Deploy.md)查看！

> WARNING
使用CREATE2可能会影响msg.sender的行为；有关详细信息，请参阅[这里](../Tutorials/Deploy/Deploy.md#caveats)的文档！