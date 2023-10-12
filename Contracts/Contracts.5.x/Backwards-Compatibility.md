# Backwards Compatibility
OpenZeppelin合约使用语义版本来表达其API和存储布局的向后兼容性。补丁和次要更新通常会向后兼容，但也有少数例外，如下文所述。大的更新应被认为与以前的版本不兼容。在这个页面上，我们提供了关于这些保证的详细信息。

## API
在向后兼容的版本中，所有的变化应该是添加或修改内部实现细节。大部分代码应该能继续编译并按预期运行。这个规则的例外如下。

### 安全性
偶尔，一个补丁或次要更新会以破坏性的方式移除或改变一个API，但只有在之前的API被认为不安全的情况下才会这样做。这些破坏性的变化将在更新日志和发布说明中注明，并伴随着安全公告发布。

### 草案或预最终ERCs
未最终确定的ERC可能会以不兼容的方式改变。因此，我们避免在ERC最终确定之前发布它们的实现。对于已经发布了很长时间且似乎不太可能改变的ERC，我们会做一些例外。在这些情况下，ERC规范的破坏性变化仍然是可能的，所以这些实现会在名为draft-*.sol的文件中发布，以明确说明这种情况。

### 虚拟和覆盖
这个库中几乎所有的函数都是虚拟的，但这并不意味着鼓励覆盖。有一部分函数是设计用来被覆盖的。如果你在这个子集之外定义覆盖，那么你可能依赖于内部实现细节。我们会努力保持向后兼容性，即使在这些情况下，但这是非常困难的，很容易意外地破坏。建议谨慎。

此外，一些次要更新可能会导致新的编译错误，如"两个或更多的基类定义了具有相同名称和参数类型的函数"或"需要指定被覆盖的合约"，这是由于Solidity认为继承函数中存在歧义。这应该通过添加一个调用函数的覆盖来解决。

有关虚拟和覆盖的更多信息，请参见*扩展合约*。

### 结构体
带有下划线前缀的结构体成员应被视为"私有"，并可能在次要版本中发生变化。结构体数据只应通过库函数访问和修改。

### 错误
除非另有说明，否则不应假定特定错误格式和reverts附带的数据是稳定的。

### 主要版本
主要版本应被认为是不兼容的。然而，如果合约的外部接口是标准化的，或者维护者认为改变它们会对生态系统造成重大压力，那么它们将保持兼容。

主要版本可能会破坏的一个重要方面是"升级兼容性"，特别是存储布局兼容性。对于在线合约来说，从一个主要版本升级到另一个主要版本永远不会是安全的。

## Storage Layout
次要和补丁更新始终保持存储布局的兼容性。这意味着一个在线合约可以从一个次要版本升级到另一个版本，而不会破坏存储布局。在某些情况下，升级时可能需要初始化新的状态变量，尽管我们预计这种情况不会频繁发生。

我们建议使用OpenZeppelin升级插件或CLI来确保升级的存储布局安全。

## Solidity版本
编译合约所需的最低Solidity版本在次要和补丁更新中将保持不变。在次要版本中引入的新合约可能会使用更新的Solidity特性，并需要更近期的编译器版本。