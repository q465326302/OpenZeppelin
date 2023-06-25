# Subgraph examples

## 基本应用

### 应用描述
我们将考虑一个虚构的dapp，在Goerli网络上，它包括：

* 一个ERC20代币，地址为0x163AE1e077232D6C34E1BF14fA58aA74518886Cc，部署在块5059780上。

### 清单
该子图的清单可能如下所示。请注意，根据您的文件夹布局，相对路径可能会更改：
```
specVersion: 0.0.2
schema:
  file: ../node_modules/@openzeppelin/subgraphs/generated/erc20.schema.graphql
dataSources:
  - kind: ethereum/contract
    name: erc20
    network: goerli
    source:
      address: "0x163AE1e077232D6C34E1BF14fA58aA74518886Cc"
      abi: IERC20
      startBlock: 5059780
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - ERC20Contract
      abis:
        - name: IERC20
          file: ../node_modules/@openzeppelin/contracts/build/contracts/IERC20Metadata.json
      eventHandlers:
        - event: Approval(indexed address,indexed address,uint256)
          handler: handleApproval
        - event: Transfer(indexed address,indexed address,uint256)
          handler: handleTransfer
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/erc20.ts
```

## 应用程序与2个数据源

### 应用程序描述
对于这个第二个例子，我们将扩展我们的示例以包括更多功能：

* 一个ERC20代币，在地址0x163AE1e077232D6C34E1BF14fA58aA74518886Cc处具有自己的所有权和暂停功能，并在块5059780上部署。

* 一个OpenZeppelin TimelockController，在地址0x9949d9507A26b9639B882E15A897A0A79DDf2c94处内部使用AccessControl模式，并在块5059781上部署。

### 清单
对于这个第二个例子，我们必须支持多个模块，为了简单起见，我们使用all.schema.graphql。
```
specVersion: 0.0.2
schema:
  file: ../node_modules/@openzeppelin/subgraphs/generated/all.schema.graphql
dataSources:
  - kind: ethereum/contract
    name: erc20
    network: goerli
    source:
      address: "0x163AE1e077232D6C34E1BF14fA58aA74518886Cc"
      abi: IERC20
      startBlock: 5059780
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - ERC20Contract
      abis:
        - name: IERC20
          file: ../node_modules/@openzeppelin/contracts/build/contracts/IERC20Metadata.json
      eventHandlers:
        - event: Approval(indexed address,indexed address,uint256)
          handler: handleApproval
        - event: Transfer(indexed address,indexed address,uint256)
          handler: handleTransfer
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/erc20.ts
  - kind: ethereum/contract
    name: ownable
    network: goerli
    source:
      address: "0x163AE1e077232D6C34E1BF14fA58aA74518886Cc"
      abi: Ownable
      startBlock: 5059780
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - Ownable
      abis:
        - name: Ownable
          file: ../node_modules/@openzeppelin/contracts/build/contracts/Ownable.json
      eventHandlers:
        - event: OwnershipTransferred(indexed address,indexed address)
          handler: handleOwnershipTransferred
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/ownable.ts
  - kind: ethereum/contract
    name: pausable
    network: goerli
    source:
      address: "0x163AE1e077232D6C34E1BF14fA58aA74518886Cc"
      abi: Pausable
      startBlock: 5059780
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - Pausable
      abis:
        - name: Pausable
          file: ../node_modules/@openzeppelin/contracts/build/contracts/Pausable.json
      eventHandlers:
        - event: Paused(address)
          handler: handlePaused
        - event: Unpaused(address)
          handler: handleUnpaused
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/pausable.ts
  - kind: ethereum/contract
    name: timelock
    network: goerli
    source:
      address: "0x9949d9507A26b9639B882E15A897A0A79DDf2c94"
      abi: Timelock
      startBlock: 5059781
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - Timelock
      abis:
        - name: Timelock
          file: ../node_modules/@openzeppelin/contracts/build/contracts/TimelockController.json
      eventHandlers:
        - event: CallScheduled(indexed bytes32,indexed uint256,address,uint256,bytes,bytes32,uint256)
          handler: handleCallScheduled
        - event: CallExecuted(indexed bytes32,indexed uint256,address,uint256,bytes)
          handler: handleCallExecuted
        - event: Cancelled(indexed bytes32)
          handler: handleCancelled
        - event: MinDelayChange(uint256,uint256)
          handler: handleMinDelayChange
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/timelock.ts
  - kind: ethereum/contract
    name: accesscontrol
    network: goerli
    source:
      address: "0x9949d9507A26b9639B882E15A897A0A79DDf2c94"
      abi: AccessControl
      startBlock: 5059781
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.4
      language: wasm/assemblyscript
      entities:
        - AccessControl
      abis:
        - name: AccessControl
          file: ../node_modules/@openzeppelin/contracts/build/contracts/AccessControl.json
      eventHandlers:
        - event: RoleAdminChanged(indexed bytes32,indexed bytes32,indexed bytes32)
          handler: handleRoleAdminChanged
        - event: RoleGranted(indexed bytes32,indexed address,indexed address)
          handler: handleRoleGranted
        - event: RoleRevoked(indexed bytes32,indexed address,indexed address)
          handler: handleRoleRevoked
      file: ../node_modules/@openzeppelin/subgraphs/src/datasources/accesscontrol.ts
```
