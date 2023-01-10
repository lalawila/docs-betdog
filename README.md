# 😁 BetDog Whitepaper

{% hint style="danger" %}
**WORK IN PROGRESS**
{% endhint %}

## Introduction

BetDog is a protocol for betting and prediction. It is designed around ease-of-use, gas efficiency, and censorship resistance.





{% hint style="success" %}
Code: [https://github.com/lalawila/betdog](https://github.com/lalawila/betdog)
{% endhint %}

## Odds

### Oracle

Odds will be input by oracle such as chainlink when create condition.

The odds list example is `[4.27, 8.55, 1.42]`. That mean there are three outcomes. You will get `427` return if bet in outcome `0` . You will get `855` return if bet in outcome `1` . You will get `142` return if bet in outcome `2` .

### Odds Change

Odds are not constant over time.  Gas fee will be too high if update odss in real time by oracle.Thus&#x20;

introduced formula `x * y = k` from uniswap.



### Example:

If there are two team game math that Apple team vs Banana team. Only two outcomes that Apple win or Banana win. The prediction probability is `[0.2, 0.8]` . Means Apple team has `0.2`

probability to win and Banana team is `0.8` . Odds is `[5, 1.25]` .

Apple team odds: 1 / 0.2 = 5, If you bet 1 Apple team win then you will get 5  return.

Banana team odds: 1 / 0.8 = 1.25, If you bet 1 Banana team win then you will get 1.25  return.



Ok, Let's provide total reserve `5000` as liquidity from pool. The amount `5000` depend how much in pool and what we want.  The larger the number, the smaller the sliding point.



Then reserves is `[1000, 4000]` . This situation reflects the odds change in real time. Because the more people who bet on a outcome, the greater the chance of winning, the lower the odds.

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

