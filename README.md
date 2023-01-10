# 😁 BetDog Whitepaper

{% hint style="danger" %}
**WORK IN PROGRESS**
{% endhint %}

## Introduction

BetDog is a protocol for betting and prediction. It is designed around ease of use, gas efficiency, and censorship resistance.

{% hint style="success" %}
Code: [https://github.com/lalawila/betdog](https://github.com/lalawila/betdog)
{% endhint %}

## Odds

### Oracle

Odds will be input by an oracle when creating a condition.

Examples of odds are`[2.5, 10, 2]`. This means there are three outcomes. You will get `2.5` a return if you bet `1` and win in 1st. You will get `10` a return if you bet `1` and win in 2nd. You will get `2` a return if you bet `1` and win in 3rd.&#x20;

We can get probability from the odds.

Probability of the first outcome winning: `1 / 2.5 = 0.4`

Probability of the second outcome winning:  `1 / 10 = 0.1`

Probability of the second outcome winning: `1 / 2 = 0.5`

It's an error odds `[2, 4]` . Better will get stable profit just bet both of two outcomes. The probability of this odds are `[0.5, 0.25]`. Ensure the sum of probabilities must be greater than or equal to 1 can avoid this.

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

If there are two team game math the Apple team vs Banana team. Only two outcomes Apple win or Banana win. The prediction probability is `[0.2, 0.8]` . This means the Apple team has a `20%`

chance to win and the Banana team is `80%` .  The odds get from probability are `[5, 1.25]` .



Ok, Let's provide total reserve `5000` as liquidity from the pool. The amount `5000` depends on us.  The larger the number, the smaller the sliding point.



Then reserves are`[1000, 4000]` . This situation reflects the odds change in real-time. Because the more people who bet on an outcome, the greater the chance of winning, and the lower the odds.

Apple team reserve:  5000 \* 0.2 = 1000

Banana team reserve:  5000 \* 0.8 = 4000



Lilei bet 10 amount in Apple team. &#x20;

The reward is `4000 - 1000 * 4000 / ( 1000 + 10)=` 39.603960396 `` .&#x20;

If Apple team win, Lilei will get `reward + bet` capital `= 49.6` amount.  Actual odds is `49.6 / 10 = 4.96` . Actual odds slightly lower than input odds. This is a bit like sliding point in uniswap.



The reward and bet capital will be return right game over.

The latest reverses is `[1010, 3950.4]` now.   The latest odds is `[4.91128712871, 1.25567031187]` now.

Apple team odds:  (1010 + 3950.4) / 1010 = 4.91128712871

Banana team odds:  (1010 + 3950.4) / 3950.4 = 1.25567031187

#### What if number of outcomes greater than 2 ?

There `x * y = z` seem can only deal with  2  outcomes.

## Providing Liquidity

### Adding Liquidity

Providing the amount of liquidity you want to add. Then smart contract will charge for equivalent value token and mint the amount of liquidity tokens for you.

valueToken = amountLiquidit \* totalValue / totalSupply

### Removing Liquidity

Providing the amount of liquidity you want to remove. Then smart contract will pay for equivalent value token and burn the amount of liquidity token for you.



### Liquidity Fee

There is a 1% of winer’s rewards will be charge for liquidity income.

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



## Manage Condition

```solidity
function createCondition(
    uint64[] calldata oddsList,
    uint256 reserve,
    uint64 startTime,
    uint64 endTime,
    bytes32 ipfsHash
) external onlyOracle returns (uint256) {
    require(reserve >= minReserve, "must be at least min reserve");

    pool.lockValue(reserve);

    lastConditionId++;

    Condition.Info storage conditionInfo = conditions[lastConditionId];

    conditionInfo.createCondition(oddsList, reserve, startTime, endTime, ipfsHash);

    emit CreatedCondition(lastConditionId);
    return lastConditionId;
}
```











...

