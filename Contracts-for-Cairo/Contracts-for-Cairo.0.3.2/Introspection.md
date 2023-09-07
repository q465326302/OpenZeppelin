# Introspection
> CAUTION
预计这个模块会不断发展。

## 
目录
- [Introspection](#introspection)
  - [](#)
  - [ERC165](#erc165)
    - [接口计算](#接口计算)
    - [注册接口](#注册接口)
    - [查询接口](#查询接口)
    - [IERC165](#ierc165)
    - [IERC165 API Specification](#ierc165-api-specification)
      - [supportsInterface](#supportsinterface)
    - [ERC165 Library Functions](#erc165-library-functions)
      - [supports\_interface](#supports_interface)
      - [register\_interface](#register_interface)

## ERC165
ERC165标准允许智能合约对其他合约[进行类型内省](https://en.wikipedia.org/wiki/Type_introspection)，即检查可以调用哪些函数。这通常被称为合约的接口。

与以太坊合约一样，Cairo合约没有本地的接口概念，因此应用程序通常必须简单地相信它们没有进行错误的调用。对于受信任的设置，这不是一个问题，但通常需要与未知和不受信任的第三方地址进行交互。甚至可能没有直接调用它们的方式！（例如，ERC20代币可能被发送到一个缺乏将其转出的方法的合约中，从而永久锁定它们）。在这些情况下，声明接口的合约可以非常有助于防止错误。

应注意的是，[常量库](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/utils/constants/library.cairo)包括引用这些合约中使用的所有接口ID的常量变量。这使得代码更易读，例如使用IERC165_ID而不是0x01ffc9a7。

### 接口计算
为了确保EVM/StarkNet的兼容性，接口标识符不是在StarkNet上计算的。相反，接口ID是从函数签名的哈希的前四个字节的选择器计算得出的，而Cairo的选择器长度为252字节。由于这种差异，将EVM已经计算过的接口ID硬编码是遵循EIP165标准和EVM兼容性的最一致的方法。

### 注册接口
为了声明对给定接口的支持，合约应该导入ERC165库并注册其支持。建议通过构造函数直接或间接（作为初始化器）在合约部署时注册接口支持，就像这样。
```
from openzeppelin.introspection.erc165.library import ERC165

INTERFACE_ID = 0x12345678

@constructor
func constructor{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }():
    ERC165.register_interface(INTERFACE_ID)
    return ()
end
```

### 查询接口
要查询目标合约对接口的支持情况，查询合约应该通过IERC165调用supportsInterface函数，传入目标合约的地址和要查询的接口ID。以下是一个示例：
```
from openzeppelin.introspection.erc265.IERC165 import IERC165

INTERFACE_ID = 0x12345678

func check_support{
        syscall_ptr: felt*,
        pedersen_ptr: HashBuiltin*,
        range_check_ptr
    }(
        target_contract: felt,
    ) -> (success: felt):
    let (is_supported) = IERC165.supportsInterface(target_contract, INTERFACE_ID)
    return (is_supported)
end
```

> NOTE
supportsInterface是使用驼峰命名法的，因为它是ERC165接口中的一个公开的合约方法。这与库方法（例如[ERC165库](https://github.com/OpenZeppelin/cairo-contracts/blob/ad399728e6fcd5956a4ed347fb5e8ee731d37ec4/src/openzeppelin/introspection/erc165/library.cairo)中的supports_interface）不同，库方法使用下划线命名法，并且不是公开的。有关更多细节，请参阅[函数名称和编码风格](./Extensibility.md#函数名称和编码风格)。

### IERC165
```
@contract_interface
namespace IERC165:
    func supportsInterface(interfaceId: felt) -> (success: felt):
    end
end
```

### IERC165 API Specification
```
func supportsInterface(interfaceId: felt) -> (success: felt):
end
```

#### supportsInterface
如果此合约实现了由interfaceId定义的接口，则返回true。

参数:
```
interfaceId: felt
```

返回：
```
success: felt
```

### ERC165 Library Functions
```
func supports_interface(interface_id: felt) -> (success: felt):
end

func register_interface(interface_id: felt):
end
```

#### supports_interface
如果此合约实现了由interface_id定义的接口，则返回true。

参数:
```
interface_id: felt
```

返回：
```
success: felt
```

#### register_interface
调用合约声明对由interface_id定义的特定接口的支持。

参数：
```
interface_id: felt
```

返回： 无