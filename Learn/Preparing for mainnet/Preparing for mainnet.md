# 准备主网
在*测试网络上运行项目*一段时间后，您会希望将其部署到主要的以太坊网络（也称为主网）。然而，计划去主网的工作应该比您计划的发布日期早得多。

在本指南中，我们将介绍将项目推向生产环境时需要考虑的以太坊特定因素，例如：
* *审计与安全*

* *验证源代码*

* *安全管理密钥*

* *处理项目治理*

请记住，尽管在测试网和主网中管理合约在技术上是相同的，但在主网上存在重要的区别，因为您的项目现在为您的用户管理真实价值。

## 审计和安全
虽然安全性影响着软件开发的所有领域，但在智能合约中，安全性尤为重要。任何人都可以直接向您的合约发送任何有效负载的交易，而您的所有合约代码和状态都是公开可访问的。更糟糕的是，在您被黑客攻击的情况下，没有任何追索权来取回被盗的资金-它们在分散网络中永远消失了。

因此，在开发的所有阶段中，安全性应是主要关注的问题。这意味着**安全性不是您在发布前一周才考虑的问题，而是从项目启动的第一天就是一个指导原则**。

在开始编码时，审查[智能合约安全最佳实践](https://consensys.github.io/smart-contract-best-practices/)，参与我们*论坛中的安全讨论*，并确保通过我们的*质量检查清单*，以确保您的项目健康。

完成后，现在是向一个或多个审计公司请求审计的好时机。您可以向OpenZeppelin研究团队*请求审计*-我们是一个经验丰富、*拥有悠久记录*的团队。

请记住，审计并不能保证不存在漏洞，但有几位经验丰富的安全研究人员审查您的代码肯定会有所帮助。

## 验证您的源代码
在将合约部署到主网后，您应该**验证其源代码**。这个过程涉及将Solidity代码提交给第三方，如[Etherscan](https://etherscan.io/)或[Etherchain](https://www.etherchain.org/)，他们将对其进行编译并验证其与部署的汇编相匹配。这使得任何用户都可以在区块浏览器中查看您的合约代码，并知道它对应于实际在该地址运行的汇编。

您可以在[Etherscan](https://etherscan.io/verifyContract)网站上手动验证您的合约。

>TIP
当您部署可升级合约时，用户与之交互的合约将只是代理，实际逻辑将位于实现合约中。[Etherscan确实支持正确显示OpenZeppelin代理及其实现](https://medium.com/etherscan-blog/and-finally-proxy-contract-support-on-etherscan-693e3da0714b)，但其他浏览器可能不支持。

## 密钥管理
在主网上工作时，您需要特别注意保护私钥的安全。您用于部署和与合约交互的账户将持有真正的以太币，这些以太币具有真正的价值，是黑客的诱人目标。采取一切预防措施来保护您的私钥，并在必要时考虑使用[硬件钱包](https://docs.ethhub.io/using-ethereum/wallets/hardware/)。

>NOTE
与测试网不同，您无法从水龙头获取主网以太币。您需要前往交易所以其他加密货币或法定货币交换真正的以太币来部署您的合约。

另外，您可以定义某些帐户在系统中拥有特殊权限 - 并且您应该特别注意保护它们。

### 管理员账户
管理员账户是系统中拥有特殊权限的账户。例如，管理员可能有*暂停*合约的能力。如果这样的账户落入恶意用户手中，他们可能会在您的系统中造成混乱。

保护管理员账户的一个好选择是使用特殊合约，如多重签名合约，而不是普通的外部拥有账户。多重签名是一种合约，只要预定义数量的信任成员同意，就可以执行任何操作。[Gnosis Safe](https://safe.gnosis.io/multisig)是一个好的多重签名合约。

### 升级管理员
*OpenZeppelin Upgrades Plugins*项目中的特殊管理员帐户是具有*升级*其他合约权限的帐户。默认情况下，这是用于部署合约的外部拥有的帐户：虽然这对于本地或测试网部署足够好，但在主网中，您需要更好地保护您的合约。获取您的升级管理员帐户的攻击者可以更改系统中的任何合约！

考虑到这一点，最好在部署后**更改ProxyAdmin的所有权**-例如，更改为多签名。要执行此操作，可以使用admin.transferProxyAdminOwnership将ProxyAdmin合约的所有权转移。

当您需要升级合约时，我们可以使用prepareUpgrade来验证并部署新的实现合约，以便在更新代理时使用。

## 项目治理
可以说，管理员账户反映了一个项目实际上并不是去中心化的。毕竟，如果一个账户可以单独更改系统中的任何合约，我们并没有创建一个无需信任的环境。

这就是治理的作用。在许多情况下，您的项目中会有一些需要特殊权限的操作，从微调系统参数到运行完整的合约升级。您需要选择如何决定这些操作：是由[一小群](https://safe.gnosis.io/multisig)受信任的开发人员决定，还是由所有项目利益相关者的[公共投票](https://daostack.io/)决定。

这里没有正确的答案。您选择项目的治理方案将在很大程度上取决于您正在构建什么以及您的社区是谁。

## 下一步
恭喜！您已经完成了开发之旅，从编写第一个合约到部署到生产环境。但工作还远没有结束。现在，您需要开始收集实时用户反馈，通过合约升级添加新功能到您的系统中！监控您的应用程序，最终扩展您的项目。

在本网站上，您可以获得OpenZeppelin平台中所有项目的详细指南和参考，以便您选择所需的内容来构建您的以太坊应用程序。愉快的编码！