# Crowdsales
> NOTE
这个页面还没有完成。我们正在努力改进它，以便在下一次发布时发布。请保持关注！

## 核心

### 众筹销售
Crowdsale（众筹销售）是一个用于管理代币众筹的基本合约，允许投资者使用以太币购买代币。该合约以其最基本的形式实现了这种功能，并可以扩展以提供额外的功能和/或自定义行为。外部接口表示购买代币的基本接口，并符合众筹的基本架构。它不打算被修改/覆盖。内部接口符合可扩展和可修改的众筹表面。通过重写方法来添加功能。在适当的地方考虑使用 'super' 来连接行为。

**MODIFIERS**
REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor(rate, wallet, token)

fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_preValidatePurchase(beneficiary, weiAmount)

_postValidatePurchase(beneficiary, weiAmount)

_deliverTokens(beneficiary, tokenAmount)

_processPurchase(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
TokensPurchased(purchaser, beneficiary, value, amount)

#### constructor(uint256 rate, address payable wallet, contract IERC20 token)
公开#
汇率是以wei和最小且不可分割的代币单位之间的转换关系。因此，如果您使用的是一个名为TOK的ERC20Detailed代币，其小数位为3位，且汇率为1，那么1 wei将给您1个单位，或者0.001 TOK。

#### fallback()
外部#
回退函数**不要覆盖**。请注意，其他合约在转移资金时会附带2300个基本燃气补贴，这不足以调用buyTokens。考虑直接调用buyTokens来购买代币。

#### token() → contract IERC20
公开#

#### wallet() → address payable
公开#

#### rate() → uint256
公开#

#### weiRaised() → uint256
公开#

#### buyTokens(address beneficiary)
公开#
低级代币购买请勿**覆盖此函数**。此函数具有非重入保护，因此不应由另一个非重入函数调用。

#### _preValidatePurchase(address beneficiary, uint256 weiAmount)
内部#
验证进货的有效性。使用require语句在条件不满足时恢复状态。在继承自Crowdsale的合约中使用super来扩展验证。CappedCrowdsale.sol的_preValidatePurchase方法的示例：super._preValidatePurchase（受益人，weiAmount）; require（weiRaised（）。add（weiAmount）⇐ cap）;

#### _postValidatePurchase(address beneficiary, uint256 weiAmount)
内部#
验证已执行的购买。观察状态并使用回滚语句来撤销无效条件。

#### _deliverTokens(address beneficiary, uint256 tokenAmount)
内部#
令牌的来源。重写此方法以修改众筹最终获取和发送其令牌的方式。

#### _processPurchase(address beneficiary, uint256 tokenAmount)
内部#
当购买已经验证并准备执行时执行。不一定发出/发送代币。

#### _updatePurchasingState(address beneficiary, uint256 weiAmount)
内部#
对于需要内部状态来检查有效性的扩展进行覆盖（例如当前用户的贡献等）。

#### _getTokenAmount(uint256 weiAmount) → uint256
内部#
覆盖以扩展将以太币转换为代币的方式。

#### _forwardFunds()
内部#
确定以太坊在购买时如何存储/转发。

#### TokensPurchased(address purchaser, address beneficiary, uint256 value, uint256 amount)
事件#

## Emission

### AllowanceCrowdsale
众售的扩展，其中代币由一个钱包持有，并批准向众售提供津贴。

**MODIFIERS**
REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor(tokenWallet)

tokenWallet()

remainingTokens()

_deliverTokens(beneficiary, tokenAmount)

CROWDSALE
fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_preValidatePurchase(beneficiary, weiAmount)

_postValidatePurchase(beneficiary, weiAmount)

_processPurchase(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### constructor(address tokenWallet)
公开#

#### tokenWallet() → address
公开#

#### remainingTokens() → uint256
公开#
检查津贴中剩余的代币数量。

#### _deliverTokens(address beneficiary, uint256 tokenAmount)
内部#
通过从钱包中转移代币来覆盖父级行为。

### MintedCrowdsale
扩展Crowdsale合约，其中每次购买都会铸造代币。代币所有权应转移到MintedCrowdsale以进行铸币。

**MODIFIERS**
REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
_deliverTokens(beneficiary, tokenAmount)

CROWDSALE
constructor(rate, wallet, token)

fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_preValidatePurchase(beneficiary, weiAmount)

_postValidatePurchase(beneficiary, weiAmount)

_processPurchase(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### _deliverTokens(address beneficiary, uint256 tokenAmount)
内部#
通过购买时铸造代币来覆盖交付。

## Validation

### CappedCrowdsale
有总贡献限制的众筹销售。

**MODIFIERS**
REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor(cap)

cap()

capReached()

_preValidatePurchase(beneficiary, weiAmount)

CROWDSALE
fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_postValidatePurchase(beneficiary, weiAmount)

_deliverTokens(beneficiary, tokenAmount)

_processPurchase(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### constructor(uint256 cap)
公开#
构造函数，接受众筹中接受的最大wei金额。

#### cap() → uint256
公开#

#### capReached() → bool
公开#
检查是否达到了上限。