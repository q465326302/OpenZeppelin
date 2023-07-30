# Test Helpers
用于以太坊智能合约测试的断言库。确保您的合约的行为符合预期！

检查事务是否以正确的原因回滚

验证事件是否以正确的值进行了发射

优雅地跟踪余额变化

处理非常大的数字

模拟时间的流逝

概述
安装
npm install --save-dev @openzeppelin/test-helpers
Hardhat（以前的Buidler）
安装web3和hardhat-web3插件。

npm install --save-dev @nomiclabs/hardhat-web3 web3
请记得按照安装说明将插件包含在您的配置中。

用法
在您的测试文件中导入@openzeppelin/test-helpers以访问不同的断言和实用程序。