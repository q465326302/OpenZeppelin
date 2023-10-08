# Counterfactual deployments
反事实合约是一种我们可以在实际在链上部署之前就与之交互的合约。例如，我们可以向尚不存在的合约发送资金或分配权限。为什么呢？因为Starknet的部署是确定性的，使我们能够预测我们的合约将被部署在哪个地址。我们可以利用这个属性让合约为自己的部署支付费用，只需提前发送资金。我们称之为反事实部署。

这个过程可以用以下步骤来描述：

> TIP
如果你想测试这个流程，你可以查看[Starknet Foundry](https://foundry-rs.github.io/starknet-foundry/starknet/account.html)或[Starkli](https://book.starkli.rs/accounts#account-deployment)的部署账户指南。

1. 根据class_hash，salt和constructor calldata确定性地预计算contract_address。注意，class_hash必须先前声明，部署才能成功。

2. 将资金发送到contract_address。通常，你会先估计交易费用。现有的工具通常会为你做这个。

3. 向网络发送一个DeployAccount类型的交易。

4. 然后，协议将使用要部署的合约的__validate_deploy__入口点验证交易。

5. 如果验证成功，协议将收取费用，然后注册合约为已部署。

> NOTE
尽管这种方法非常受欢迎用于部署账户，但它适用于任何类型的合约。

## Deployment validation
要反事实部署，部署合约必须实现__validate_deploy__入口点，当向网络发送DeployAccount交易时，协议会调用它。
```
trait IDeployable {
    /// Must return 'VALID' when the validation is successful.
    fn __validate_deploy__(
        class_hash: felt252, contract_address_salt: felt252, _public_key: felt252
    ) -> felt252;
}
```