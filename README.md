# 😁 BetDog Whitepaper

{% hint style="danger" %}
**WORK IN PROGRESS**
{% endhint %}

## Introduction

BetDog is a protocol for betting and prediction. It is designed around ease of use, gas efficiency, and censorship resistance.

{% hint style="success" %}
Code: [https://github.com/lalawila/betdog](https://github.com/lalawila/betdog)

Website: still under development
{% endhint %}

## Odds

### Input Odds

Odds will be input by the oracle when creating a condition.

Examples of odds are`[2.5, 10, 2]`. This means there are three outcomes. You will get `2.5` a return if you bet `1` and win in 1st. You will get `10` a return if you bet `1` and win in 2nd. You will get `2` a return if you bet `1` and win in 3rd.&#x20;

We can get probability from the odds.

> Probability of the first outcome winning: `1 / 2.5 = 0.4`
>
> Probability of the second outcome winning:  `1 / 10 = 0.1`
>
> Probability of the third outcome winning: `1 / 2 = 0.5`



It's an error odds `[2, 4]` . Better will get stable profit just bet both of two outcomes. The probability of these odds are `[0.5, 0.25]`. Ensure the sum of probabilities must be greater than or equal to 1 can avoid this.

{% code title="libraries/Condition.sol" %}
```solidity
function createCondition(
    Condition.Info storage self,
    uint64[] calldata odds,
    uint256 reserve,
    uint64 startTime,
    uint64 endTime,
    bytes32 ipfsHash
) internal {
    require(endTime > startTime, "end time must be greater than start time");

    uint256 totalOdds = 0;
    for (uint256 i = 0; i < odds.length; i++) {
        totalOdds += multiplier ** 2 / odds[i];
    }

    // 1e4 is allowed tolerances
    require(totalOdds >= (multiplier - 1e4), "sum of probabilities must be greater than or equal to 1");

    self.state = Condition.ConditionState.CREATED;
    self.reserves = calcReserve(odds, reserve);
    self.startTime = startTime;
    self.endTime = endTime;
    self.reserve = reserve;
    self.ipfsHash = ipfsHash;
}
```
{% endcode %}

### Odds Change

The odds are not constant over time.  The gas fee will be too high if oracle updates the odds in real time. Thus introduced formula `x * y = k` to solve the problem.

### Example:

If there are two team game math the Apple team vs Banana team. Only two outcomes Apple win or Banana win. The prediction probability is `[0.2, 0.8]` . This means the Apple team has a `20%` chance to win and the Banana team is `80%` .  The odds get from probability are `[5, 1.25]` .



Ok, Let's provide total reserve `5000` as liquidity from the pool. The amount `5000` depends on us.  **The larger the number, the smaller the slippage.**



Then reserves are`[1000, 4000]` . This situation reflects the odds change in real-time. Because the more people who bet on an outcome, the greater the chance of winning, and the lower the odds.

> Apple team reserve:  5000 \* 0.2 = 1000
>
> Banana team reserve:  5000 \* 0.8 = 4000

{% code title="libraries/Condition.sol" %}
```solidity
function calcReserve(uint64[] calldata odds, uint256 totalReserve) internal pure returns (uint256[] memory reserves) {
    reserves = new uint256[](odds.length);

    for (uint64 i = 0; i < odds.length; i++) {
        reserves[i] = (totalReserve * multiplier) / odds[i];
    }
}
```
{% endcode %}



If bet 10 amount in Apple team.  The reward is `4000 - 1000 * 4000 / ( 1000 + 10)=` 39.603960396 `` .&#x20;

If the Apple team wins will get `reward + bet capital = 49.6` .  Actual odds is `49.6 / 10 = 4.96` . Actual odds are slightly lower than input odds.&#x20;



The reward and bet capital will be returned when gaming is over.

The latest reverses are`[1010, 3950.4]` so the latest odds are `[4.91128712871, 1.25567031187]` now.

> Apple team odds:  (1010 + 3950.4) / 1010 = 4.91128712871
>
> Banana team odds:  (1010 + 3950.4) / 3950.4 = 1.25567031187

#### What if the number of outcomes is greater than 2?

The bet reserve can be considered as x. Sum of others can be considered as y.



{% code title="libraries/Condition.sol" %}
```solidity


function addReserve(Condition.Info storage self, uint64 betIndex, uint256 amount) internal returns (uint256 reward) {
    uint256 total = totalReserves(self);
    uint256 anothersReserves = total - self.reserves[betIndex];

    uint256 k = self.reserves[betIndex] * anothersReserves;

    self.reserves[betIndex] += amount;

    uint256 afterAnothers = k / self.reserves[betIndex];
    reward = anothersReserves - afterAnothers;

    uint256 ratio = (multiplier * afterAnothers) / anothersReserves;

    for (uint64 i = 0; i < self.reserves.length; i++) {
        if (i != betIndex) {
            self.reserves[i] = (self.reserves[i] * ratio) / multiplier;
        }
    }
}
```
{% endcode %}

## Providing Liquidity

### Adding Liquidity

Providing the amount of liquidity you want to add. The smart contract will charge for equivalent value tokens and mint the number of liquidity tokens for you.



{% code title="LiquidityPoolERC20.sol" %}
```solidity
/// @inheritdoc ILiquidityPoolERC20
function addLiquidity(uint256 amount) external override {
    uint256 value = _addLiquidity(amount);

    IERC20(token).safeTransferFrom(msg.sender, address(this), value);
}

function _addLiquidity(uint256 amount) private returns (uint256 value) {
    uint256 currentSupply = totalSupply();

    if (currentSupply == 0) {
        value = amount;
    } else {
        value = (amount * totalValue()) / currentSupply;
    }

    _mint(msg.sender, amount);
}
```
{% endcode %}



### Removing Liquidity

Providing the amount of liquidity you want to remove. Then the smart contract will burn liquidity tokens and return equivalent value tokens for you.

{% code title="LiquidityPoolERC20.sol" %}
```solidity
/// @inheritdoc ILiquidityPoolERC20
function removeLiquidity(uint256 amount) external override {
    uint256 value = _removeLiquidity(amount);

    IERC20(token).safeTransfer(msg.sender, value);
}

function _removeLiquidity(uint256 amount) private returns (uint256 value) {
    require(amount <= balanceOf(msg.sender), "liquidity influences");

    value = (amount * totalValue()) / totalSupply();

    _burn(msg.sender, amount);
}
```
{% endcode %}



### Liquidity Fee

1% of the winner’s rewards will be charged for liquidity income.

{% code title="Core.sol" %}
```solidity
function resolveBet(uint256 tokenId) external {
    IBetNFT.Info memory betInfo = betNFT.getBet(tokenId);

    Condition.Info storage conditionInfo = conditions[betInfo.conditionId];

    require(conditionInfo.state == Condition.ConditionState.RESOLVED, "must be resolved first");

    if (conditionInfo.outcomeWinIndex == betInfo.outcomeIndex) {
        // There is a 1% of winer’s rewards will be charge for liquidity income.
        uint256 reward = (betInfo.reward * (multiplier - fee)) / multiplier;
        pool.pay(msg.sender, reward + betInfo.amount);
    }

    betNFT.resolveBet(tokenId);
}
```
{% endcode %}



## Manage Condition

### Creating Condition

* Lock values of total reserves in the pool
* Save condition info
* Calculate reserve for each outcome by odds

{% code title="Core.sol" %}
```solidity
/// @notice Oracle: Create new condition.
/// @param odds Odds for outcomes such as [4.27, 8.55, 1.42]
/// @param reserve The amount of reserve will be locked in the pool
/// @param startTime The start time of betting
/// @param endTime The end time of betting
/// @param ipfsHash detailed info about match stored in IPFS
function createCondition(
    uint64[] calldata odds,
    uint256 reserve,
    uint64 startTime,
    uint64 endTime,
    bytes32 ipfsHash
) external onlyOracle returns (uint256) {
    require(reserve >= minReserve, "must be at least min reserve");

    pool.lockValue(reserve);

    lastConditionId++;

    Condition.Info storage conditionInfo = conditions[lastConditionId];

    conditionInfo.createCondition(odds, reserve, startTime, endTime, ipfsHash);

    emit CreatedCondition(lastConditionId);
    return lastConditionId;
}
```
{% endcode %}

{% code title="libraries/Condition.sol" %}
```solidity
function createCondition(
    Condition.Info storage self,
    uint64[] calldata odds,
    uint256 reserve,
    uint64 startTime,
    uint64 endTime,
    bytes32 ipfsHash
) internal {
    require(endTime > startTime, "end time must be greater than start time");

    uint256 totalOdds = 0;
    for (uint256 i = 0; i < odds.length; i++) {
        totalOdds += multiplier ** 2 / odds[i];
    }

    // 1e4 is allowed tolerances
    require(totalOdds >= (multiplier - 1e4), "sum of probabilities must be greater than or equal to 1");

    self.state = Condition.ConditionState.CREATED;
    self.reserves = calcReserve(odds, reserve);
    self.startTime = startTime;
    self.endTime = endTime;
    self.reserve = reserve;
    self.ipfsHash = ipfsHash;
}
```
{% endcode %}

### Resolving Condition

* Save the index of the victory
* Change the state to RESOLVED
* Release the locked values in the pool

{% code title="Core.sol" %}
```solidity
function resolveCondition(uint256 conditionId, uint64 outcomeWinIndex) external onlyOracle {
    conditions[conditionId].resolveCondition(outcomeWinIndex);
    pool.releaseValue(conditions[conditionId].reserve);
}
```
{% endcode %}

{% code title="libraries/Condition.sol" %}
```solidity
function resolveCondition(Condition.Info storage self, uint64 outcomeWinIndex) internal {
    require(self.state == Condition.ConditionState.CREATED, "state must be CREATED");

    require(block.timestamp >= self.endTime, "now must be greater than endTime");

    self.state = ConditionState.RESOLVED;
    self.outcomeWinIndex = outcomeWinIndex;
}
```
{% endcode %}

## Betting

### Betting

{% code title="Core.sol" %}
```solidity
function bet(uint256 conditionId, uint64 betIndex, uint256 amount) public override returns (uint256 tokenId) {
    IERC20(pool.token()).safeTransferFrom(msg.sender, address(pool), amount);

    uint256 reward = conditions[conditionId].addReserve(betIndex, amount);

    tokenId = betNFT.mint(msg.sender, conditionId, betIndex, amount, reward);
}
```
{% endcode %}

{% code title="BetNFT.sol" %}
```solidity
function mint(
    address account,
    uint256 conditionId,
    uint256 outcomeIndex,
    uint256 amount,
    uint256 reward
) external override onlyCore returns (uint256) {
    lastTokenId++;

    _mint(account, lastTokenId);

    IBetNFT.Info storage betInfo = bets[lastTokenId];
    betInfo.state = BetState.CREATED;
    betInfo.conditionId = conditionId;
    betInfo.outcomeIndex = outcomeIndex;
    betInfo.amount = amount;
    betInfo.reward = reward;

    emit MintedBet(lastTokenId);

    return lastTokenId;
}
```
{% endcode %}

### Resolving Betting

{% code title="Core.sol" %}
```solidity
function resolveBet(uint256 tokenId) external {
    IBetNFT.Info memory betInfo = betNFT.getBet(tokenId);

    Condition.Info storage conditionInfo = conditions[betInfo.conditionId];

    require(conditionInfo.state == Condition.ConditionState.RESOLVED, "must be resolved first");

    if (conditionInfo.outcomeWinIndex == betInfo.outcomeIndex) {
        // There is a 1% of winer’s rewards will be charge for liquidity income.
        uint256 reward = (betInfo.reward * (multiplier - fee)) / multiplier;
        pool.pay(msg.sender, reward + betInfo.amount);
    }

    betNFT.resolveBet(tokenId);
}
```
{% endcode %}

{% code title="BetNFT.sol" %}
```solidity
function resolveBet(uint256 tokenId) external override onlyCore {
    IBetNFT.Info storage betInfo = bets[tokenId];

    require(betInfo.state == BetState.CREATED);

    betInfo.state = BetState.RESOLVED;
}
```
{% endcode %}
