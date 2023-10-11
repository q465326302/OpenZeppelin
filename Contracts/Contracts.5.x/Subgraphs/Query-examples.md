# Query examples

## ERC20

### 总供应量和最大的代币持有者

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

### 最近的转账记录及其交易ID
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

### 用户的所有索引的ERC20余额
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

### 用户的所有ERC721代币，包括元数据
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

### 合约上的所有ERC721代币，以及当前的所有者
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

### 给定代币的所有转账历史
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

### 具有相应总供应量和余额的注册表的所有代币
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

### 可拥有合约的转账历史
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

### 账户持有的所有可拥有的合约
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

### 访问控制合约的所有角色，包括详细信息和当前成员
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

### 账户持有的所有访问控制角色
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

### 定时锁定的待处理操作，包括子调用的详细信息和截止日期
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

### 定时锁定到一个地址的所有操作，包括状态和调用的详细信息
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