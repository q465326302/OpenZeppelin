# Creating ERC20 Supply
实现在Starknet上构建的代币的标准接口来自于以太坊上流行的代币标准，称为ERC20。[EIP20](https://eips.ethereum.org/EIPS/eip-20)，从中派生出ERC20合约，没有规定如何创建代币。本指南将介绍创建固定和动态代币供应的策略。

## Fixed Supply
假设我们想创建一个名为MyToken的代币，其代币供应量是固定的。我们可以通过在构造函数中设置代币供应量来实现这一点，该构造函数将在部署时执行。
```
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::token::erc20::ERC20;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(
        self: @ContractState,
        initial_supply: u256,
        recipient: ContractAddress
    ) {
        let name = 'MyToken';
        let symbol = 'MTK';

        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_state, name, symbol);
        ERC20::InternalImpl::_mint(ref unsafe_state, recipient, initial_supply);
    }
}
```

在构造函数中，我们首先获取ERC20状态并调用ERC20初始化器来设置代币名称和符号。接下来，我们调用内部的 _mint 函数，该函数创建初始供应的代币并将其分配给接收者。由于我们的合约中没有公开内部的 _mint，因此不可能创建更多的代币。换句话说，我们实现了固定的代币供应！

## Dynamic Supply
具有动态供应的ERC20合约包括创建或销毁代币的机制。让我们对强大的MyToken合约进行一些更改，创建一个铸币机制。
```
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::token::erc20::ERC20;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(self: @ContractState) {
        let name = 'MyToken';
        let symbol = 'MTK';

        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_state, name, symbol);
    }

    #[external(v0)]
    fn mint(
        self: @ContractState,
        recipient: ContractAddress,
        amount: u256
    ) {
        // 这个功能没有保护措施，意味着任何人都可以铸造代币。
        let mut unsafe_state = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::_mint(ref unsafe_state, recipient, amount);
    }
}
```

上述公开的mint将创建代币并将其分配给接收者。现在我们有了我们的铸币机制！

然而，这里有一个大问题。mint并没有包含任何关于谁可以调用这个函数的限制。为了良好的实践，让我们用Ownable实现一个简单的权限机制。
```
#[starknet::contract]
mod MyToken {
    use starknet::ContractAddress;
    use openzeppelin::access::ownable::Ownable;
    use openzeppelin::token::erc20::ERC20;

    #[storage]
    struct Storage {}

    #[constructor]
    fn constructor(self: @ContractState, owner: ContractAddress) {
        // 设置合约所有者
        let mut unsafe_ownable = Ownable::unsafe_new_contract_state();
        Ownable::InternalImpl::initializer(ref unsafe_ownable, owner);

        //  ERC20 接口
        let name = 'MyToken';
        let symbol = 'MTK';

        let mut unsafe_erc20 = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::initializer(ref unsafe_erc20, name, symbol);
    }

    #[external(v0)]
    fn mint(
        self: @ContractState,
        recipient: ContractAddress,
        amount: u256
    ) {
        // 设置可拥有的权限
        let unsafe_ownable = Ownable::unsafe_new_contract_state();
        Ownable::InternalImpl::assert_only_owner(@unsafe_ownable);

        // 如果合约所有者调用，则铸造代币
        let mut unsafe_erc20 = ERC20::unsafe_new_contract_state();
        ERC20::InternalImpl::_mint(ref unsafe_erc20, recipient, amount);
    }
}
```

在构造函数中，我们传递所有者地址以设置MyToken合约的所有者。mint函数包含assert_only_owner，这将确保只有合约所有者才能调用此函数。现在，我们有了一个受保护的ERC20铸币机制来创建动态的代币供应。

> TIP
有关权限机制的更详细解释，请参阅*访问控制*。