# 😁 BetDog Whitepaper

{% hint style="danger" %}
**WORK IN PROGRESS**
{% endhint %}

## Introduction

BetDog is a protocol for betting and prediction. It is designed around ease-of-use, gas efficiency, and censorship resistance.



{% hint style="success" %}
Github repository: [https://github.com/lalawila/betdog](https://github.com/lalawila/betdog)
{% endhint %}



## Odds

### Oracle

Odds list will be input by oracle such as chainlink when create condition.

The odds list example is `[4.27, 8.55, 1.42]`. That mean there are three outcomes. You will get `427` return if bet in outcome `0` . You will get `855` return if bet in outcome `1` . You will get `142` return if bet in outcome `2` .

### Odds Change

Odds are not constant over time.  Gas fee will be too high if update odss in real time by oracle.Thus&#x20;

introduced formula `x * y = k` from uniswap.



### Example Of Betting

If odds list is `[4, 1.25]`  then provide fixed ratio reserve like `[1000, 3000]`.&#x20;

Lilei bet 10 amount in outcome 0.  The reward is `3000 - 1000 * 3000 / ( 1000 + 10)= 29.7` . If outcome 0 is the right result, Lilei will get reword + bet amount = 39.7 amount.  Actual odds is `39.7 / 10 = 3.97` . Actual odds slightly lower than input odds. This is a bit like sliding point.



This situation reflects the odds change in real time. Because the more people who bet on a outcome, the greater the chance of winning, the lower the odds.







``





## Providing Liquidity

### Adding Liquidity

Providing the amount of liquidity you want to add. Then smart contract will charge for equivalent value token and mint the amount of liquidity tokens for you.

valueToken = amountLiquidit \* totalValue / totalSupply

### Removing Liquidity

Providing the amount of liquidity you want to remove. Then smart contract will pay for equivalent value token and burn the amount of liquidity token for you.

#### Liquidity income

The 5‰ of winer’s rewards will be charge for liquidity income.



##



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



## Bet







...

