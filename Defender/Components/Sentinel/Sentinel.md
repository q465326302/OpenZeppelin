# Sentinel
Defender Sentinel服务提供3种类型的Sentinels，分别是Contract Sentinels、Forta Sentinels和Forta Local Mode Sentinels。Contract Sentinels允许你通过定义事件、函数、交易参数上的条件来监控合约的交易。Forta Sentinels允许你通过定义Forta Bots、合约地址、警报ID和严重程度的条件来监控Forta警报。如果Sentinel根据你定义的条件匹配交易或Forta警报，它将通过电子邮件、slack、telegram、discord、[Autotasks](../Autotasks/Autotasks.md)等通知你。

## 用例
当你需要了解或响应涉及你的智能合约、你需要监控的其他智能合约或Forta Bots的交易和Forta Alerts时，请使用Sentinel。你的Sentinel将监视每个交易和Forta Alert，并将你关心的内容发送到你选择的通知方式。

* **监视关键功能**，如转移所有权、暂停或升级

* **警报**可能危险的合约交易

* 当**关键事件**发生时**执行逻辑响应**

* 通过Slack、Telegram、Discord、电子邮件或自定义Autotask集成与现有工具集成

* **了解**何时发生了意外的交易或警报数量增加的情况
  
### 何时使用合约Sentinel vs Forta Sentinel对比
合约Sentinels特别适用于监测单个合约的交易。通过[合约条件](#合约条件是什么)，你可以按照各种交易属性进行过滤，例如事件、函数、参数、gasPrice、value等筛选交易。

FortaSentinels适用于使用更复杂的条件来监测一个或多个智能合约，通过连接到Forta Bot开发人员定义的事件和条件。Forta机器人比合约Sentinels更具灵活性，可以定义更多的交易条件。

## 合约Sentinels中有什么？
合约Sentinels会监视与你选择的地址相关的交易，通过你指定的任何条件过滤这些交易，然后在这些交易发生时通知你。

> IMPORTANT
目前，Sentinels 在除 Fantom 之外的所有网络上都得到支持。此外，Sentinels只能在Mainnet、Ropsten和Kovan上监视内部函数调用。如果内部调用发出事件，请考虑监视该事件，因为从内部调用发出的事件可以在所有支持的网络上检测到。

### 指定地址
* **Addresses**：选择智能合约或外部拥有的账户进行监控。每个Sentinels只能有一个ABI，如果选择多个智能合约，则它们必须遵守相同的ABI/接口。

* **ABI**：指定合约的ABI。如果源代码已经验证，Defender将自动加载ABI。

* **Confirmation Blocks**：如果你希望在交易被接受的一定程度上获得通知，请选择较高的确认块级别，但如果你希望尽快收到通知并且对重新组织容忍，请选择较低的确认块级别。在接受安全和最终化块标签的链上，你也可以选择它们作为确认块级别。

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
选择你希望监视的任何事件和函数调用。选择事件和函数相当于使用OR逻辑，即如果交易与所选事件或函数中的任何一个匹配，将触发通知。

> NOTE
配置事件和函数检测需要ABI。

> NOTE
如果未指定任何事件或函数，则Sentinel将监视所有发送到或从你监视的地址的交易。

> IMPORTANT
Sentinel无缝支持所有网络上由智能合约发出的事件的通知，无论是是直接触发还是通过第三方合约的内部调用触发。然而，目前仅限于以太坊主网才能跟踪和提供针对合约内部函数调用的特定通知能力。

### Hedera支持
Hedera主网和测试网都在Defender Sentinels中得到支持。在配置Sentinel时，存在一些独特的考虑因素：

* Sentinel监视合约的EVM地址（以“0x”开头），而不是Hedera合约ID。EVM地址可以在Hedera Explorer（如[HashScan](https://hashscan.io/)）上找到。

* 目前没有远程资源可用于获取合约ABI，因此必须使用[Solidity编译器从Solidity代码生成它们](https://docs.soliditylang.org/en/latest/installing-solidity.html)。

* 对于ERC20代币转账，只有通过智能合约调用的转账可以使用Sentinels进行监视，而不能使用[Hedera Token Service（HTS）](https://hedera.com/token-service)进行监视。

* 只能监视Hedera内部交易（合约调用合约）的事件条件。其他条件不会触发Sentinels。

## 什么是Forta Sentinel？

Forta Sentinel是一个监视Forta Bot发送的Forta Alerts或影响特定地址的警报的工具。它可以根据你指定的条件对这些警报进行过滤，并在发生警报时通知你。

### Forta本地模式Sentinels
Forta提供了在本地运行扫描节点的能力，这对于仅运行特定的检测机器人并在内部处理警报而不发布到网络上非常有用。你可以通过创建一个新的Forta本地模式Sentinel并指定你的扫描节点地址来监视在本地模式下运行的扫描器节点。创建后，你的新Sentinel将具有一个Webhook URI，你可以将其添加到扫描器节点配置文件中，以将警报转发给Defender。你可以在[Forta文档](https://docs.forta.network/en/latest/scanner-local-mode)中找到有关本地模式扫描器节点的更多信息。

### 指定Forta检测机器人和地址

> NOTE
Forta Local Mode Sentinels无法指定Bot IDs，因为在本地模式下运行的扫描节点不支持Bot IDs。

Forta Sentinel可以根据Bot IDs或Addresses进行过滤，并要求至少设置其中一个过滤器。可以指定多个Bot IDs和Addresses进行过滤。

* **Bot ID**：指定要过滤的Bot ID。这些可以用逗号分隔以指定多个Bot ID。

* **Address**：选择要过滤的智能合约或外部拥有的账户。

### Forta匹配规则

为了触发通知，Forta警报必须满足以下**所有**条件：

* Forta警报**必须**来自指定的Bot ID和/或指定的地址之一。

* 如果指定了**Alert ID条件**
    * 警报中的Alert ID**必须**与指定的Alert ID之一匹配。

* 如果指定了**严重性条件**
    * 警报的严重性**必须**与指定的严重性匹配或更严重。

### 严重性和警报ID
你可以指定警报严重性和警报ID进行监视。同时指定严重性和警报ID将作为OR子句，即如果警报与所选的任何警报ID匹配或与所选的严重性匹配，则会触发通知。

> NOTE
如果未指定严重性或警报ID，则Sentinel将监视与你指定的Bot ID和/或地址匹配的所有警报。

## 合约条件是什么?
条件充当过滤器，允许你进一步缩小交易范围。这些条件以表达式的形式输入，提供了很大的灵活性。条件非常类似于JavaScript表达式。为了适应校验和与非校验和地址的比较，比较是不区分大小写的。

> NOTE
如果你想要接收涉及你选择的事件/函数的所有交易，则不要指定任何条件。

* 条件可以使用AND、OR、NOT和()

* 条件可以使用==、<、>、>=、<=进行比较

* 十六进制（0xabc123）或十进制（10000000000）可以引用数字值

* 字符串值只能通过==进行比较

* 包括基本数学运算符：+、-、*、/、^

## 交易条件
> IMPORTANT
如果指定了交易条件，则必须满足该条件才能触发通知。

交易条件可以引用以下属性：

* **to**是交易的收件人地址

* **from**是交易的发件人地址

* **gasPrice**是发送交易的gas价格。在EIP1559交易中，它等于或低于maxFeePerGas。

* **maxFeePerGas**是交易愿意支付的最高价格。仅存在于EIP1559交易中。

* **maxPriorityFeePerGas**是交易愿意支付给矿工的超过BASE_FEE的wei数量的最高金额。仅存在于EIP1559交易中。

* **gasLimit**是发送交易的gas限制

* **gasUsed**是交易中使用的gas量

* **value**是发送交易的价值

* **nonce**是特定交易的nonce

* **status**是一个派生值，可以与 **'success'** 或 **'success'**进行比较。

### 示例条件
被撤销的交易
```
status == "failed"
```

排除来自0xd5180d374b6d1961ba24d0a4dbf26d696fda4cad的交易。
```
from != "0xd5180d374b6d1961ba24d0a4dbf26d696fda4cad"
```

具有gasPrice高于50 gwei和gasUsed高于20000的交易
```
gasPrice > 50000000000 and gasUsed > 20000
```

### 事件和功能条件
事件和函数条件进一步缩小了触发通知的交易集合。这些条件可以通过名称（如果参数有名称）或索引（例如$0、$1等）引用签名中的参数。在你指定这些函数时，用户界面会显示可用的变量。

#### 示例条件
发出值在1到100 ETH之间（以十六进制表示）的Transfer（...）事件的交易
```
// 事件签名：Transfer（地址to，地址from，uint256 value）
value > 0xde0b6b3a7640000 and value < 0x56bc75e2d63100000
```

发出包含第一个元素等于5的数组的 ValsEvent(…​) 事件的交易
```
// 事件签名：ValsEvent（uint256 [3] vals）
vals[0] == 5
```

调用一个未命名字符串为“hello”的greet（…）函数的事务
```
// 函数签名：greet（地址，字符串）
$1 == "hello"
```

### Autotask条件
如果指定了自动任务条件，那么它将被调用，并且会传入一个给定块的匹配列表。这使得Sentinels可以使用其他数据源和自定义逻辑来评估一个交易是否匹配。

> NOTE
只有符合其他条件（事件、函数、交易）的交易才会调用自动任务条件。

> NOTE
每次调用最多可以包含25个交易。

### 请求模式
请求正文将包含以下结构。 如果你在TypeScript中编写Autotasks，则可以使用[defender-autotask-utils](https://www.npmjs.com/package/defender-autotask-utils)软件包中的SentinelConditionRequest类型。
```
{
  "events": [
  {
    "transaction": {                     // eth_getTransactionReceipt response body
      ...                                // see https://eips.ethereum.org/EIPS/eip-1474
    },
    "blockHash": "0xab..123",            // block hash from where this transaction was seen
    "matchReasons": [                    // the reasons why sentinel triggered
      {
        "type": "event",                 // event, function, or transaction
        "address": "0x123..abc",         // address of the event emitting contract
        "signature": "...",              // signature of your event/function
        "condition": "value > 5",        // condition expression (if any)
        "args": ["5"],                   // parameters by index (unnamed are present)
        "params": { "value": "5" }       // parameters by name (unnamed are not present)
      }
    ],
    "matchedAddresses": ["0x000..000"],  // the addresses from this transaction your are monitoring
    "sentinel": {
      "id": "44a7d5...31df5",            // internal ID of your sentinel
      "name": "Sentinel Name",           // name of your sentinel
      "abi": [...],                      // abi of your addresses (or undefined)
      "addresses": ["0x000..000"],       // addresses your sentinel is watching
      "confirmBlocks": 0,                // number of blocks sentinel waits (can be 'safe' or 'finalized' on PoS clients)
      "network": "rinkeby"               // network of your addresses
      "chainId": 4                       // chain Id of the network
    }
  }
  ]
}
```

### 响应模式（Response Schema）
Autotask必须返回一个包含所有匹配项的结构。返回一个空对象表示没有匹配发生。此对象的类型为SentinelConditionResponse。

> IMPORTANT
错误将被视为不匹配。所有执行可以在Autotask的运行页面中找到。

```
{
  "matches": [
    {
      "hash": "0xabc...123",   // transaction hash
      "metadata": {
        "foo": true            // any object to be shared with notifications
      }
    },
    {
      "hash": "0xabc...123"    // example with no metadata specified
    }
  ]
}
```

Autotask条件示例
```
exports.handler = async function(payload) {
  const conditionRequest = payload.request.body;
  const matches = [];
  const events = conditionRequest.events;
  for(const evt of events) {

    // add custom logic for matching here

    // metadata can be any JSON-marshalable object (or undefined)
    matches.push({
       hash: evt.hash,
       metadata: {
        "id": "customId",
        "timestamp": new Date().getTime(),
        "numberVal": 5,
        "nested": { "example": { "here": true } }
       }
    });
  }
  return { matches }
}
```

#### 测试条件
在条件表单的右侧，有一个“测试Sentinels条件”的工具。该工具在一定范围内搜索与Sentinels条件匹配的交易。如果指定了自动任务条件，则测试还会调用它。

选项

* **“最近的区块”**搜索网络最新区块之前的一定范围内的区块。

* **“特定区块”**将搜索指定的区块。

* **“特定交易”**将尝试匹配一个交易哈希（0xabc…def）。

搜索使用当前表单中的条件。

注意：运行测试不会触发通知。

## 什么是 Forta 条件？
Forta 条件作为过滤器，允许你进一步缩小 Forta 警报范围。

### 严重性条件
严重性条件允许你仅收到大于某个影响级别的警报通知。你将收到与你选择的严重性值匹配或具有更大影响级别的任何警报通知。

Forta 警报可能具有以下 5 种严重性值之一，表示不同的影响级别：

* **关键** - 可利用的漏洞，对用户/资金有巨大影响

* **高** - 在更特定的条件下可利用，对用户/资金有重大影响

* **中** - 显着的意外行为，对用户/资金的影响适中至较低

* **低** - 次要的疏忽，对用户/资金影响微不足道

* **信息** - 值得通报的其他行为

### 警报 ID 条件
警报 ID 条件允许你过滤警报，并仅收到特定类型的发现的通知。可以指定一个或多个警报 ID。

#### 示例条件
```
FORTA-1, NETHFORTA-1
```

### 自动任务条件

如果指定了自动任务条件，那么它将被调用并带有匹配列表。这允许Sentinels使用其他数据源和自定义逻辑来评估事务是否匹配。

> NOTE
只有符合其他条件（严重程度、警报 ID）的警报才会调用自动任务条件。

#### 请求架构

请求正文将包含以下结构。

> NOTE
我们根据新的 [Forta API](https://docs.forta.network/en/latest/api/) 更新了 Forta 警报架构。进行了以下更改：alert_id → alertId，scanner_count → scanNodeCount，type → findingType，tx_hash → transactionHash，chain_Id → chainId，删除了 Bot 名称，agent → bot。旧属性现已弃用，但我们将继续发送两者以保持向后兼容性。

> NOTE
Forta 已更改 “Agent” 的术语为 “Detection Bot”。我们将继续暂时将它们称为 “agents”。 sentinel.agents 将是你的机器人 ID 列表。
```
{
  "events": [
    {
      "alert": {                            // Forta Alert
        "addresses": [ "0xab..123" ],       // map of addresses involved in the transaction
        "alertId": "NETHFORTA-1",           // unique string to identify this class of finding
        "name": "High Gas Used",            // human-readable name of finding
        "description": "Gas Used: 999999",  // brief description
        "hash": "0xab..123",                // Forta Alert transaction hash
        "protocol": "ethereum",             // specifies which network the transaction was mined
        "scanNodeCount": 1,
        "severity": "MEDIUM",               // indicates impact level of finding
        "findingType": "SUSPICIOUS",        // indicates type of finding: Exploit, Suspicious, Degraded, Info
        "metadata": { "gas": "999999" },    // metadata for the alert
        "source": {
          "transactionHash": "0xab..123",   // network transaction hash  e.g ethereum transaction hash
          "bot": {
            "id": "0xab..123",              // Bot ID
          },
          "block": {
            "chainId": 1,                   // Chain ID of the originating network
            "hash": "0xab..123",            // network block hash  e.g ethereum block hash
          }
        }
      },
      "matchReasons": [                     // the reasons why sentinel triggered
        {
          "type": "alert-id",               // Alert ID or Severity
          "value": "NETHFORTA-1"            // Condition Value
        }
      ],
      "sentinel": {
        "id": "forta_id",                   // internal ID of your sentinel
        "name": "forta sentinel",           // name of your sentinel
        "addresses": [ "0xab..123" ],       // addresses your sentinel is monitoring
        "agents": [ "0xab..123" ]           // Bot IDs your sentinel is monitoring
        "network": "mainnet"                // network your sentinel is monitoring
        "chainId": 1                        // chain Id of the network
      }
    }
  ]
}
```

#### 响应模式
Autotask必须返回包含所有匹配项的结构。返回空对象表示没有匹配项。此对象的类型为SentinelConditionResponse。

错误将被视为不匹配。所有执行都可以在Autotask的运行页面上找到。
```
{
  "matches": [
    {
      "hash": "0xabc...123",   // Forta Alert hash i.e events[0].alert.hash
      "metadata": {
        "foo": true            // any object to be shared with notifications
      }
    },
    {
      "hash": "0xabc...123"    // example with no metadata specified
    }
  ]
}
```

### 示例Autotask条件
```
exports.handler = async function(payload) {
  const conditionRequest = payload.request.body;
  const matches = [];
  const events = conditionRequest.events;
  for(const evt of events) {

    // add custom logic for matching here
    // metadata can be any JSON-marshalable object (or undefined)
    matches.push({
       hash: evt.hash,
       metadata: {
        "id": "customId",
        "timestamp": new Date().getTime(),
        "numberVal": 5,
        "nested": { "example": { "here": true } }
       }
    });
  }
  return { matches }
}
```

## 通知
当触发时，Sentinel可以通知一个或多个Slack Webhooks、Telegram机器人、Discord Webhooks、电子邮件列表、Datadog指标、自定义Webhooks或执行自动任务。

### Slack配置
请参阅[Slack Webhook文档](https://api.slack.com/messaging/webhooks)以配置Slack Webhook。配置完Slack后，在Defender中输入Webhook URL。

* **Alias**是此Slack配置的名称。例如，你可以根据频道的名称来命名它。

* **Webhook URL**是用于通知的Slack管理控制台中的URL。

### Emails配置
* **Alias**是此电子邮件列表的名称。（例如，开发人员）

* **Emails**是你希望通知的电子邮件列表。这些可以用逗号或分号分隔。

### Discord配置
请参阅[Discord Webhook文档](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)以配置Discord频道的Webhook。

* **Alias**是此Discord配置的名称。

* **Webhook URL**是你的Discord频道中用于通知的URL。

### Datadog配置
Datadog配置允许Defender将自定义指标转发到你的Datadog帐户。有关自定义指标的更多信息，请参见[Datadog指标文档](https://docs.datadoghq.com/developers/metrics/)。

我们发送的指标是一个COUNT指标，表示触发Sentinel的交易数量。如果Sentinel没有触发，则不发送零，因此可以预期没有数据。对于每个指标，我们发送两个标签：网络（Rinkeby、Mainnet等）和Sentinel（Sentinel的名称）。

> NOTE
新的自定义指标可能需要几分钟才能在Datadog控制台中显示。

* **Alias**是此Datadog配置的名称。

* **API密钥**是来自你的Datadog管理的API密钥。

* **指标前缀**将在所有指标名称之前。例如，使用defender.作为前缀，Sentinels将发送一个名为defender.sentinel的指标。

### PagerDuty配置
请查看[PagerDuty集成文档](https://support.pagerduty.com/docs/services-and-integrations)，配置一个可以创建变更和警报事件的PagerDuty API v2集成。

* **事件类型** PagerDuty分类的事件类型（警报或变更）

* **路由密钥**服务或全局规则集上集成的集成密钥（32个字符）

* **事件动作**：事件的操作类型（触发、确认或解决）

* **去重密钥** 用于关联触发器和解决方案的去重键（最多255个字符）

* **严重程度**事件描述的状态相对于受影响的系统的感知严重性（关键、错误、警告或信息）

* **组件** 事件源机器负责的组件

* **组**服务组件的逻辑分组

* **类**事件的类/类型

* **自定义详细信息**键值对映射，提供有关事件和受影响系统的其他详细信息

### Telegram配置
请参阅[Telegram机器人文档](https://core.telegram.org/bots#6-botfather)，使用BotFather配置Telegram Bot

> NOTE
Telegram机器人必须添加到你的频道并具有发布消息的权限。

要查找频道的聊天ID，请执行以下curl（使用你的机器人代币值），并提取聊天的id值。如果你没有收到任何条目的响应，请先发送测试消息到你的聊天中。
```
curl https://api.telegram.org/bot$BOT_TOKEN/getUpdates
{
  "ok": true,
  "result": [
    {
      "update_id": 98xxxx98,
      "channel_post": {
        "message_id": 26,
        "sender_chat": {
          "id": -100xxxxxx5976,
          "title": "Defender Sentinel Test",
          "type": "channel"
        },
        "chat": {
          "id": -100xxxxxx5976, // <--- This is your chat ID
          "title": "Defender Sentinel Test",
          "type": "channel"
        },
        "date": 1612809138,
        "text": "test"
      }
    }
  ]
}
```
* **Alias**是Telegram配置的名称。

* **Chat ID**是Telegram聊天的ID。

* **Bot Token**是你在创建Telegram Bot时从BotFather获得的代币。

### 自定义Webhook配置
要配置自定义的Webhook通知渠道，你只需要提供Webhook端点URL和用于显示目的的别名。

* **Alias**是此Webhook端点的名称。

* **Webhook URL**是Sentinel将发送匹配事件的URL。

为了避免在高匹配数下用许多并发请求压倒接收Webhook，Sentinel发送一个包含事件的JSON对象，其中包含一个包含所有匹配事件的数组。
```
{
  events: [...] // See Event Schema for details on the contents of this array
}
```
事件模式与 _Event Schema_中所述的完全相同。你还可以使用测试通知功能向你的Webhook发送测试通知。

### Opsgenie配置
请查看[Opsgenie集成文档](https://support.atlassian.com/opsgenie/docs/create-a-default-api-integration/)，以配置可创建警报的Opsgenie API集成。

* **API密钥** 集成设置中可以找到的API密钥

* **实例位置** Opsgenie实例服务器所在的位置

* **响应者** 将警报路由到的团队、用户、升级和计划，以发送通知。每个项目都必须具有type字段，其中可能的值为团队、用户、升级和计划。如果API密钥属于团队集成，则此字段将被重写为所有者团队。每个响应者的id或name都应提供。你可以参考下面的示例值（50个团队、用户、升级或计划）

* **可见** 团队和用户，警报将对其可见，而不发送任何通知。每个项目都必须具有type字段，其中可能的值为团队和用户。除了type字段外，团队应给出id或name，用户应给出id或username。请注意：警报将默认对指定在responder字段中的团队可见，因此无需在visibleTo字段中重新指定它们。你可以参考下面的示例值（总共50个团队或用户）

* **Alias** 警报的客户定义标识符，也是警报[去重的关键元素](https://support.atlassian.com/opsgenie/docs/what-is-alert-de-duplication/)（最多512个字符）

* **Priority** 警报的优先级级别。可能的值为P1、P2、P3、P4和P5。默认值为P3

* **Entity** 警报的实体字段通常用于指定警报所涉及的域（最多512个字符）

* **Actions** 可用于警报的自定义操作（10个x 50个字符）

* **Note** 创建警报时添加的附加注释（最多25000个字符）

* **Details** 键值对映射，用作警报的自定义属性（最多8000个字符）

* **Tags** 警报的标签（20个x 50个字符）

### Autotask
如果选择自动任务，则自动任务将接收一个包含触发事件的详细信息的body属性，可以是触发交易的交易详细信息或触发警报的Forta警报详细信息。自动任务可以执行自定义逻辑并根据需要访问外部API。

> IMPORTANT
自动任务执行受配额限制。配额用尽后，自动任务将不再执行。如果你需要提高自动任务执行配额，请发送电子邮件至defender@openzeppelin.com，并描述你的用例。

## Autotask事件
sentinel将向你的自动任务传递有关交易的信息。如果你使用TypeScript编写自动任务，则可以使用[defender-autotask-utils](https://www.npmjs.com/package/defender-autotask-utils)包中的BlockTriggerEvent类型进行合约sentinel，使用FortaTriggerEvent类型进行FortaSentinels。

### Autotask示例
```
exports.handler = async function(params) {
  const payload = params.request.body;
  const matchReasons = payload.matchReasons;
  const sentinel = payload.sentinel;

  // if contract sentinel
  const transaction  = payload.transaction;
  const abi = sentinel.abi;

  // if Forta sentinel
  const alert  = payload.alert;



  // custom logic...
}
```

### 事件模式（Event Schema）

#### 合约Sentinel
```
{
  "transaction": {                     // eth_getTransactionReceipt response body
    ...                                // see https://eips.ethereum.org/EIPS/eip-1474
  },
  "blockHash": "0xab..123",            // block hash from where this transaction was seen
  "matchReasons": [                    // the reasons why sentinel triggered
    {
      "type": "event",                 // event, function, or transaction
      "address": "0x123..abc",         // address of the event emitting contract
      "signature": "...",              // signature of your event/function
      "condition": "value > 5",        // condition expression (if any)
      "args": ["5"],                   // parameters by index (unnamed are present)
      "params": { "value": "5" }       // parameters by name (unnamed are not present)
    }
  ],
  "matchedAddresses":["0x000..000"]    // the addresses from this transaction your are monitoring
  "sentinel": {
    "id": "44a7d5...31df5",            // internal ID of your sentinel
    "name": "Sentinel Name",           // name of your sentinel
    "abi": [...],                      // abi of your address (or undefined)
    "addresses": ["0x000..000"],       // addresses your sentinel is watching
    "confirmBlocks": 0,                // number of blocks sentinel waits (can be 'safe' or 'finalized' on PoS clients)
    "network": "rinkeby"               // network of your address
    "chainId": 4                       // chain Id of the network
  },
  "value": "0x16345785D8A0000"         // value of the transaction
  "metadata": {...}                // metadata injected by Autotask Condition (if applicable)
}
```

#### Forta Sentinel

> NOTE
我们根据新的Forta API更新了Forta Alert模式。进行了以下更改：alert_id → alertId，scanner_count → scanNodeCount，type → findingType，tx_hash → transactionHash，chain_Id → chainId，删除了Bot名称，agent → bot。旧属性现在已被弃用，但我们将继续发送两者以保持向后兼容性。

> NOTE
Forta已将“Agent”的术语更改为“Detection Bot”。我们暂时将继续称之为“agents”。sentinel.agents将是你的Bot ID列表。

```
{
  "alert": {                            // Forta Alert
    "addresses": [ "0xab..123" ],       // map of addresses involved in the transaction
    "alertId": "NETHFORTA-1",           // unique string to identify this class of finding
    "name": "High Gas Used",            // human-readable name of finding
    "description": "Gas Used: 999999",  // brief description
    "hash": "0xab..123",                // Forta Alert transaction hash
    "protocol": "ethereum",             // specifies which network the transaction was mined
    "scanNodeCount": 1,
    "severity": "MEDIUM",               // indicates impact level of finding
    "findingType": "SUSPICIOUS",        // indicates type of finding: Exploit, Suspicious, Degraded, Info
    "metadata": { "gas": "999999" },    // metadata for the alert
    "source": {
      "transactionHash": "0xab..123",   // network transaction hash  e.g ethereum transaction hash
      "bot": {
        "id": "0xab..123",              // Bot ID
      },
      "block": {
        "chainId": 1,                   // Chain ID of the originating network
        "hash": "0xab..123",            // network block hash  e.g ethereum block hash
      }
    }
  },
  "matchReasons": [                     // the reasons why sentinel triggered
    {
      "type": "alert-id",               // Alert ID or Severity
      "value": "NETHFORTA-1"            // Condition Value
    }
  ],
  "sentinel": {
    "id": "forta_id",                   // internal ID of your sentinel
    "name": "forta sentinel",           // name of your sentinel
    "addresses": [ "0xab..123" ],       // addresses your sentinel is monitoring
    "agents": [ "0xab..123" ]           // Bot IDs your sentinel is monitoring
    "network": "mainnet"                // network your sentinel is monitoring
    "chainId": 1                        // chain Id of the network
  },
  "value": undefined                    // value will always be undefined for FORTA sentinels
}
```

## 定制通知消息
你可以选择性地使用通知渠道选择器下方的复选框来修改消息主题（仅限纯文本）和正文内容以及格式。

### 例如

#### 模板

```
**Sentinel Name**

{{ sentinel.name }}

**Network**

{{ sentinel.network }}

**Block Hash**

{{ blockHash }}

**Transaction Hash**

{{ transaction.transactionHash }}

**Transaction Link**

[Block Explorer]({{ transaction.link }})

{{ matchReasonsFormatted }}

**value**

{{ value }}
```

#### 预览
**Sentinel名称**
Sentinel

**网络**
rinkeby

**块哈希**
0x22407d00e953e5f8dabea57673b9109dad31acfc15d07126b9dc22c33521af52

**交易哈希**
0x1dc91b98249fa9f2c5c37486a2427a3a7825be240c1c84961dfb3063d9c04d50
[块浏览器](https://rinkeby.etherscan.io/tx/0x1dc91b98249fa9f2c5c37486a2427a3a7825be240c1c84961dfb3063d9c04d50)

**匹配原因 1**
类型：函数
匹配地址：0x1bb1b73c4f0bda4f67dca266ce6ef42f520fbb98
签名：greet(name)
条件：name == 'test'
参数：
name：test

**匹配原因 2**
类型：交易
条件：gasPrice > 10

**价值**
0x16345785D8A0000

### 消息语法
自定义消息主题（电子邮件和Opsgenie）仅支持纯文本，而自定义消息正文内容支持有限的Markdown语法：

* 加粗（**此文本加粗**）

* 斜体（*此文本*和_此文本_为斜体）

* 链接（这是一个[链接](http://example.com)）

对于其他Markdown语法，如标题、块引用和表格，不同的消息平台支持程度不同。电子邮件支持完整的HTML并具有最丰富的功能集，但其他消息平台存在局限性，同时支持的功能组合（例如粗体和斜体的文本）也有不同的支持程度。如果你的Markdown包含任何具有混合平台支持的语法，编辑器下方将显示警告消息。

### 动态内容
自定义通知模板使用内联模板呈现动态内容。任何用双花括号括起来的字符串都将根据 _Event Schema_解析。可以使用点表示法访问深度嵌套的项（包括数组中的项）。

除了标准事件模式外，还注入了以下参数以供在自定义通知消息中使用：

* transaction.link

* matchReasonsFormatted

### 字符限制
如果消息超过平台的字符限制，消息将被截断。最佳做法是将消息限制为1900个字符。

## 控制通知速率
一旦你根据需要指定了条件，有两种方法可以限制通知数量：警报和超时。这些应该一起使用，以实现广泛的警报行为。

### 警报阈值
要在匹配交易超过阈值时发出警报，请使用警报阈值。

> NOTE
该阈值对每个交易进行评估。一旦超过阈值，那么通知将继续触发，直到在当前交易之前的时间窗口内金额低于阈值。考虑使用超时值以防止后续通知。

* 金额是此Sentinels必须触发的次数。

* 窗口是考虑的秒数

例子：
在**一小时内至少5次**应指定Amount为5，Window为3600秒。

### 超时
如果你不希望以超过一定速率接收通知，请考虑使用超时。这将在发送通知后有效地防止一定时间内的通知。

* 超时是等待通知之间的秒数

例子：
**避免每小时通知超过一次**应指定超时为3600

## 暂停
暂停Sentinels将暂停监视你的地址。