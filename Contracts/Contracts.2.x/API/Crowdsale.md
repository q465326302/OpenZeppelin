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

#### _preValidatePurchase(address beneficiary, uint256 weiAmount)
内部#
扩展父类行为，要求购买必须遵守资金上限。

### IndividuallyCappedCrowdsale
每个受益人有限制的众筹销售。

**MODIFIERS**
CAPPERROLE
onlyCapper()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
setCap(beneficiary, cap)

getCap(beneficiary)

getContribution(beneficiary)

_preValidatePurchase(beneficiary, weiAmount)

_updatePurchasingState(beneficiary, weiAmount)

CAPPERROLE
constructor()

isCapper(account)

addCapper(account)

renounceCapper()

_addCapper(account)

_removeCapper(account)

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

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CAPPERROLE
CapperAdded(account)

CapperRemoved(account)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### setCap(address beneficiary, uint256 cap)
外部#
设定特定受益人的最大捐款额。

#### getCap(address beneficiary) → uint256
公开#
返回特定受益人的上限。

#### getContribution(address beneficiary) → uint256
公开#
返回特定受益人迄今为止的捐款金额。

#### _preValidatePurchase(address beneficiary, uint256 weiAmount)
内部#
扩展父级行为以尊重受益人的资金上限。

#### _updatePurchasingState(address beneficiary, uint256 weiAmount)
内部#
扩展父级行为以更新受益人的捐款。

### PausableCrowdsale
Crowdsale合约的扩展，其中购买可以由暂停者角色暂停和取消暂停。

**MODIFIERS**
PAUSABLE
whenNotPaused()

whenPaused()

PAUSERROLE
onlyPauser()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
_preValidatePurchase(_beneficiary, _weiAmount)

PAUSABLE
constructor()

paused()

pause()

unpause()

PAUSERROLE
isPauser(account)

addPauser(account)

renouncePauser()

_addPauser(account)

_removePauser(account)

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
PAUSABLE
Paused(account)

Unpaused(account)

PAUSERROLE
PauserAdded(account)

PauserRemoved(account)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### _preValidatePurchase(address _beneficiary, uint256 _weiAmount)
内部#
验证即将进行的购买。使用require语句在条件不满足时回滚状态。使用super来连接验证。添加验证，众筹销售不能暂停。

#### TimedCrowdsale
众筹只接受在特定时间范围内的捐款。

**MODIFIERS**
onlyWhileOpen()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor(openingTime, closingTime)

openingTime()

closingTime()

isOpen()

hasClosed()

_preValidatePurchase(beneficiary, weiAmount)

_extendTime(newClosingTime)

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
TimedCrowdsaleExtended(prevClosingTime, newClosingTime)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### onlyWhileOpen()
修饰符#
如果不在众筹时间范围内，则进行还原。

#### constructor(uint256 openingTime, uint256 closingTime)
公开#
构造函数，接受众筹开始和结束时间。

#### openingTime() → uint256
公开#

#### closingTime() → uint256
公开#

#### isOpen() → bool
公开#

#### hasClosed() → bool
公开#
检查众筹开放的期限是否已经过去。

#### _preValidatePurchase(address beneficiary, uint256 weiAmount)
内部#
延长要求在贡献期内的父级行为。

#### _extendTime(uint256 newClosingTime)
内部#
延长众筹活动。

#### TimedCrowdsaleExtended(uint256 prevClosingTime, uint256 newClosingTime)
事件#

### WhitelistCrowdsale
只有被列入白名单的用户才能参与的众筹活动。

**MODIFIERS**
REENTRANCYGUARD
nonReentrant()

WHITELISTEDROLE
onlyWhitelisted()

WHITELISTADMINROLE
onlyWhitelistAdmin()

**FUNCTIONS**
_preValidatePurchase(_beneficiary, _weiAmount)

CROWDSALE
constructor(rate, wallet, token)

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

WHITELISTEDROLE
isWhitelisted(account)

addWhitelisted(account)

removeWhitelisted(account)

renounceWhitelisted()

_addWhitelisted(account)

_removeWhitelisted(account)

WHITELISTADMINROLE
isWhitelistAdmin(account)

addWhitelistAdmin(account)

renounceWhitelistAdmin()

_addWhitelistAdmin(account)

_removeWhitelistAdmin(account)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

WHITELISTEDROLE
WhitelistedAdded(account)

WhitelistedRemoved(account)

WHITELISTADMINROLE
WhitelistAdminAdded(account)

WhitelistAdminRemoved(account)

#### _preValidatePurchase(address _beneficiary, uint256 _weiAmount)
内部#
扩展父行为，要求受益人必须在白名单中。请注意，对发送交易的账户没有任何限制。

## Distribution

### FinalizableCrowdsale

将TimedCrowdsale扩展为一次性的最终化操作，可以在完成后进行额外的工作。

**MODIFIERS**
TIMEDCROWDSALE
onlyWhileOpen()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor()

finalized()

finalize()

_finalization()

TIMEDCROWDSALE
openingTime()

closingTime()

isOpen()

hasClosed()

_preValidatePurchase(beneficiary, weiAmount)

_extendTime(newClosingTime)

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
CrowdsaleFinalized()

TIMEDCROWDSALE
TimedCrowdsaleExtended(prevClosingTime, newClosingTime)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### constructor()
内部#

#### finalized() → bool
公开#

#### finalize()
公开#
必须在众筹结束后调用，以完成一些额外的最终工作。调用合约的最终化函数。

#### _finalization()
内部#
可以被覆盖以添加最终化逻辑。覆盖的函数应该调用super._finalization()来确保完整地执行最终化链。

#### CrowdsaleFinalized()
事件#

### PostDeliveryCrowdsale
众筹活动将令代币在结束之前无法提取。

**MODIFIERS**
TIMEDCROWDSALE
onlyWhileOpen()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
withdrawTokens(beneficiary)

balanceOf(account)

_processPurchase(beneficiary, tokenAmount)

TIMEDCROWDSALE
constructor(openingTime, closingTime)

openingTime()

closingTime()

isOpen()

hasClosed()

_preValidatePurchase(beneficiary, weiAmount)

_extendTime(newClosingTime)

CROWDSALE
fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_postValidatePurchase(beneficiary, weiAmount)

_deliverTokens(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

_forwardFunds()

CONTEXT
_msgSender()

_msgData()

**EVENTS**
TIMEDCROWDSALE
TimedCrowdsaleExtended(prevClosingTime, newClosingTime)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### withdrawTokens(address beneficiary)
公开#
在众筹结束后才能提取代币。

#### balanceOf(address account) → uint256
公开#

#### _processPurchase(address beneficiary, uint256 tokenAmount)
内部#
通过存储到期余额并将代币交付给保险库，覆盖了父级的功能。这样可以确保代币在提取时可用（如果稍后调用_deliverTokens可能不会这样）。

### RefundableCrowdsale
扩展FinalizableCrowdsale合约，添加了一个筹款目标，并且如果目标未达到，用户可以获得退款的可能性。

已弃用，请改用RefundablePostDeliveryCrowdsale。请注意，如果在目标达成之前允许代币交易，那么可能会发生一种攻击，攻击者从众筹中购买代币，并在发现目标不太可能达到时以折扣价出售代币。当众筹完成时，攻击者将获得退款，而从他们那里购买的用户将得到一些毫无价值的代币。

**MODIFIERS**
TIMEDCROWDSALE
onlyWhileOpen()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
constructor(goal)

goal()

claimRefund(refundee)

goalReached()

_finalization()

_forwardFunds()

FINALIZABLECROWDSALE
finalized()

finalize()

TIMEDCROWDSALE
openingTime()

closingTime()

isOpen()

hasClosed()

_preValidatePurchase(beneficiary, weiAmount)

_extendTime(newClosingTime)

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

CONTEXT
_msgSender()

_msgData()

**EVENTS**
FINALIZABLECROWDSALE
CrowdsaleFinalized()

TIMEDCROWDSALE
TimedCrowdsaleExtended(prevClosingTime, newClosingTime)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### constructor(uint256 goal)
公开#
构造函数，创建RefundEscrow。

#### goal() → uint256
公开#

#### claimRefund(address payable refundee)
公开#
如果众筹活动不成功，投资者可以在这里申请退款。

#### goalReached() → bool
公开#
检查是否达到了筹款目标。

#### _finalization()
内部#
当调用finalize()时，进行的最终化任务。

#### _forwardFunds()
内部#
覆盖众筹资金转发，将资金发送到托管账户。

### RefundablePostDeliveryCrowdsale
RefundableCrowdsale合约的扩展，只有在众筹结束并达到目标后才交付代币，防止向代币持有人发放退款。

**MODIFIERS**
TIMEDCROWDSALE
onlyWhileOpen()

REENTRANCYGUARD
nonReentrant()

**FUNCTIONS**
withdrawTokens(beneficiary)

POSTDELIVERYCROWDSALE
balanceOf(account)

_processPurchase(beneficiary, tokenAmount)

REFUNDABLECROWDSALE
constructor(goal)

goal()

claimRefund(refundee)

goalReached()

_finalization()

_forwardFunds()

FINALIZABLECROWDSALE
finalized()

finalize()

TIMEDCROWDSALE
openingTime()

closingTime()

isOpen()

hasClosed()

_preValidatePurchase(beneficiary, weiAmount)

_extendTime(newClosingTime)

CROWDSALE
fallback()

token()

wallet()

rate()

weiRaised()

buyTokens(beneficiary)

_postValidatePurchase(beneficiary, weiAmount)

_deliverTokens(beneficiary, tokenAmount)

_updatePurchasingState(beneficiary, weiAmount)

_getTokenAmount(weiAmount)

CONTEXT
_msgSender()

_msgData()

**EVENTS**
FINALIZABLECROWDSALE
CrowdsaleFinalized()

TIMEDCROWDSALE
TimedCrowdsaleExtended(prevClosingTime, newClosingTime)

CROWDSALE
TokensPurchased(purchaser, beneficiary, value, amount)

#### withdrawTokens(address beneficiary)
公开#