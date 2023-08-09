# Query examples

## ERC20

### 总供应量和最大代币持有者
```
{
  erc20Contract(id: "<token-address-in-lowercase>") {
    totalSupply {
      value
    }
    balances(orderBy: valueExact, orderDirection: desc, where: { account_not: null }) {
      account {
        id
      }
      value
    }
  }
}
```

### 最近的转账交易ID
```
{
  erc20Contract(id: "<token-address-in-lowercase>") {
    transfers(orderBy: timestamp, orderDirection: desc) {
      from { id }
      to { id }
      value
      transaction { id }
    }
  }
}
```

### 一个用户的所有已索引的ERC20余额
```
{
  account(id: "<user-address-in-lowercase>") {
    ERC20balances {
      contract{ name, symbol, decimals }
    	value
    }
  }
}
```

## ERC721

### 一个用户的所有带有元数据的ERC721代币
```
{
  account(id: "<user-address-in-lowercase>") {
    ERC721tokens {
      contract { id }
      identifier
      uri
    }
  }
}
```

### 一个合约上的所有ERC721代币，包括当前所有者。\
```
{
  erc721Contract(id: "<registry-address-in-lowercase>") {
    tokens {
      identifier
      owner { id }
    }
  }
}
```

### 给定代币的所有转账历史记录
```
{
  erc721Tokens(where: {
    contract: "<registry-address-in-lowercase>",
    identifier: "<token-identifier-in-decimal>",
  }) {
    transfers {
      timestamp
      from { id }
      to { id }
    }
  }
}
```

## ERC1155

### 一个具有对应的totalSupply和余额的注册表的所有代币
```
{
  erc1155Contract(id: "<registry-address-in-lowercase>") {
    tokens {
    identifier
      totalSupply { value }
      balances(where: { account_not: null }) {
        account { id }
        value
      }
  }
  }
}
```

## Ownable
### 可拥有合约的转移历史
```
{
  ownable(id : "<ownable-address-in-lowercase>") {
    ownershipTransferred(orderBy: timestamp, orderDirection: asc) {
      timestamp
      owner { id }
    }
  }
}
```

### 所有可拥有的合约都可以由一个账户提供帮助。
```
{
  account(id: "<user-address-in-lowercase>") {
    ownerOf {
      id
    }
  }
}
```

## AccessControl

### 所有角色，包括细节和当前AccessControl合约成员
```
{
  accessControl(id: "<accesscontrol-contract-in-lowercase>") {
    roles {
      role { id }
      admin { role { id } }
      adminOf { role { id } }
      members { account { id } }
    }
  }
}
```

### 一个帐户拥有的所有访问控制角色
```
{
  account(id: "<user-address-in-lowercase>") {
    membership {
      accesscontrolrole {
        contract { id }
        role { id }
      }
    }
  }
}
```

## Timelock

### 对时间锁的待处理操作，包括子调用和截止日期的详细信息。
```
{
  timelock(id: "<timelock-address-in-lowercase>") {
    id
    operations(where: { status: "SCHEDULED"}) {
      calls {
        target { id }
        value
        data
      }
      timestamp
    }
  }
}
```

### 所有针对某个地址的时间锁定操作，包括操作状态和调用细节
```
{
  account(id: "<address-of-the-target-in-lowercase>") {
    timelockedCalls {
      operation {
        contract { id }
        timestamp
        status
      }
      value
      data
    }
  }
}
```
