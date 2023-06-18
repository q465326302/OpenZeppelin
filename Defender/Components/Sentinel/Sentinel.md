# Sentinel
Defender Sentinel服务提供3种类型的Sentinels，分别是Contract Sentinels、Forta Sentinels和Forta Local Mode Sentinels。Contract Sentinels允许您通过定义事件、函数、交易参数上的条件来监控合约的交易。Forta Sentinels允许您通过定义Forta Bots、合约地址、警报ID和严重程度的条件来监控Forta警报。如果Sentinel根据您定义的条件匹配交易或Forta警报，它将通过电子邮件、slack、telegram、discord、*Autotasks*等通知您。

## 用例
当您需要了解或响应涉及您的智能合约、其他需要监视的智能合约或Forta Bots的交易和Forta Alerts时，请使用Sentinel。您的Sentinel将监视每个交易和Forta Alert，并将您关心的内容发送到您选择的通知方式。

* **监视敏感功能**，如转移所有权、暂停或升级

* **警报**可能危险的合约交易

* 当**关键事件**发生时**执行逻辑**进行**响应**

* 通过Slack、Telegram、Discord、电子邮件或自定义Autotask集成与现有工具集成

### 何时使用合同而不是Forta Sentinel
合约哨兵特别适用于监测单个合约的交易。通过合约条件，您可以按照各种交易属性进行过滤，例如事件、函数、参数、gasPrice、价值等。

Forta哨兵适用于使用更复杂的条件来监测一个或多个智能合约，通过连接到Forta机器人开发者定义的事件和条件。Forta机器人比合约哨兵更具灵活性，可以定义更多的交易条件。

## 合约哨兵中有什么？
合约哨兵会监视与您选择的地址相关的交易，通过您指定的任何条件过滤这些交易，然后在这些交易发生时通知您。

>IMPORTANT
目前，Sentinels 在除 Fantom 之外的所有网络上都得到支持。此外，Sentinels 只能监视 Mainnet、Ropsten 和 Kovan 上的内部函数调用。如果内部调用发出事件，请考虑监视该事件，因为从内部调用发出的事件在所有支持的网络上都可以检测到。

### 指定地址
* **地址**：选择智能合约或外部拥有的账户进行监控。每个哨兵只能有一个ABI，如果选择多个智能合约，则它们必须遵守相同的ABI/接口。

* **ABI**：指定合约的ABI。如果源代码已经验证，Defender将自动加载ABI。

* **确认块**：如果您希望在交易被接受的一定程度上获得通知，请选择较高的确认块级别，但如果您希望尽快收到通知并且对重新组织容忍，请选择较低的确认块级别。在接受安全和最终化块标签的链中，您也可以选择它们作为确认块级别。

## 匹配规则
为了触发通知，交易必须满足以下所有条件：

* 交易必须有一个与配置地址匹配的收件人、发件人或地址（从日志中获取）。

* 如果指定了交易条件
    *  交易必须符合交易条件。

* 如果选择了事件
    * 交易必须发出任何选定的事件并匹配事件条件（如果有）。

* 如果选择了函数
    * 交易必须直接调用任何选定的函数（合约调用目前不被检测到），并匹配函数条件（如果有）。
  
### 事件和函数
选择您希望监视的任何事件和函数调用。选择事件和函数作为OR子句，即如果交易匹配所选的任何事件或函数，则会触发通知。

>NOTE
配置事件和函数检测需要ABI。

>NOTE
如果未指定任何事件或函数，则Sentinel将监视所有发送到或从您监视的地址的交易。

>IMPORTANT
Sentinel无缝支持所有网络上由智能合约发出的事件的通知，无论它们是直接触发还是通过第三方合约的内部调用触发。然而，跟踪和提供针对合约内部函数调用的通知能力目前仅限于以太坊主网。

### Hedera支持
Hedera主网和测试网都在Defender Sentinels中得到支持。在配置Sentinel时，存在一些独特的考虑因素：

* Sentinel监视合约的EVM地址（以“0x”开头），而不是Hedera合约ID。EVM地址可以在Hedera Explorer（如[HashScan](https://hashscan.io/)）上找到。

* 目前没有远程资源可用于获取合约ABI，因此必须使用[Solidity编译器从Solidity代码生成它们](https://docs.soliditylang.org/en/latest/installing-solidity.html)。

* 对于ERC20代币转账，只有通过智能合约调用的转账可以使用Sentinels进行监视，而不能使用[Hedera Token Service（HTS）](https://hedera.com/token-service)进行监视。

* 仅可以监视Hedera内部交易（合约调用合约）的事件条件。其他条件不会触发Sentinels。

## 什么是Forta Sentinel？

Forta Sentinel包含一个监视来自特定Forta Bot或影响特定地址的Forta Alerts的功能，可以通过您指定的任何条件过滤这些警报，然后在这些警报发生时通知您。

### Forta本地模式Sentinels
Forta提供了在本地运行扫描节点的能力，这对于仅运行特定的检测机器人并在内部处理警报而不发布到网络上非常有用。您可以通过创建一个新的Forta本地模式Sentinel并指定您的扫描节点地址来监视在本地模式下运行的扫描器节点。创建后，您的新Sentinel将具有一个Webhook URI，您可以将其添加到扫描器节点配置文件中，以将警报转发给Defender。您可以在[Forta文档](https://docs.forta.network/en/latest/scanner-local-mode)中找到有关本地模式扫描器节点的更多信息。

### 指定Forta检测机器人和地址

>NOTE
Forta本地模式哨兵没有指定Bot ID的选项，因为在本地模式下运行的扫描节点不支持Bot ID。

Forta哨兵过滤Bot ID或地址，并要求至少设置其中一个过滤器。可以指定多个Bot ID和地址进行过滤。

* **Bot ID**：指定要过滤的Bot ID。这些可以用逗号分隔以指定多个Bot ID。

* **地址**：选择智能合约或外部拥有的帐户进行过滤。

### Forta匹配规则

为了触发通知，Forta警报必须满足以下**所有**条件：

* Forta警报**必须**来自指定的Bot ID和/或指定的地址之一。

* 如果指定了**Alert ID条件**
    * 警报中的Alert ID**必须**与指定的Alert ID之一匹配。

* 如果指定了**严重性条件**
    * 警报的严重性**必须**与指定的严重性匹配或更严重。

### 严重性和警报ID
您可以指定警报严重性和警报ID进行监视。同时指定严重性和警报ID将作为OR子句，即如果警报与所选的任何警报ID匹配或与所选的严重性匹配，则会触发通知。

>NOTE
如果未指定严重性或警报ID，则Sentinel将监视与您指定的Bot ID和/或地址匹配的所有警报。

## 合同条件是什么?
条件作为过滤器，允许您进一步缩小交易范围。这些条件被输入为表达式，提供了很大的灵活性。条件非常像Javascript表达式。为了适应校验和和非校验和地址的比较，比较是不区分大小写的。

>NOTE
如果您想要接收涉及您选择的事件/函数的所有交易，则不要指定任何条件。

* 条件可以使用AND、OR、NOT和()

* 条件可以使用==、<、>、>=、<=进行比较

* 十六进制（0xabc123）或十进制（10000000000）可以引用数字值

* 字符串值只能通过==进行比较

* 包括基本数学运算符：+、-、*、/、^

## 交易条件
>IMPORTANT
如果指定了交易条件，则必须满足该条件才能触发通知。

交易条件可以引用以下属性：

* **to**是交易的收件人地址

* **from**是交易的发件人地址

* **gasPrice**是发送交易的燃气价格。在EIP1559交易中，它等于或低于maxFeePerGas。

* **maxFeePerGas**是交易愿意支付的最高价格。仅存在于EIP1559交易中。

* **maxPriorityFeePerGas**是交易愿意支付给矿工的超过BASE_FEE的wei数量的最高金额。仅存在于EIP1559交易中。

* **gasLimit**是发送交易的燃气限制

* **gasUsed**是交易中使用的燃气量

* **value**是发送交易的价值

* **nonce**是特定交易的nonce

* **status**是一个派生值，可以与 **“成功”** 或 **“失败”**进行比较。