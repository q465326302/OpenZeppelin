# Manage
管理与Defender 2.0配置相关的一切，包括中继器、通知渠道、地址、团队成员、API密钥等。

## Relayers
中继器用于通过Defender 2.0自动向区块链发送交易。在*中继器部分*阅读更多信息。

## Approval Processes
审批流程作为执行链上交易的包装器。它们目前包装以下内容：

* 多签名钱包

* EOA（在Defender 2.0之外）

* 中继器（在Defender 2.0内部）

* 治理合约

* 定时锁合约

* Fireblocks

它们按网络定义，并在Defender 2.0中用于执行交易，例如在*部署向导*、*操作*和*事件响应*中。
![manage-approvals](img/manage-approvals.png)

## Notification Channels
当触发时，Defender 2.0可以将通知发送到一个或多个配置的通知渠道，这些渠道可以包括Slack、电子邮件、Telegram、Discord、Datadog、PagerDuty、OpsGenie或自定义webhooks。这些通知渠道可以在监视器、操作和事件响应中使用。
![manage-notify-channels](img/manage-notify-channels.png)

### Slack Configuration
