# 😁 BetDog

{% hint style="danger" %}
**WORK IN PROGRESS**
{% endhint %}

## Introduction

BetDog is a protocol for betting and prediction. It is designed around ease-of-use, gas efficiency, and censorship resistance.



{% hint style="success" %}
Github repository: [https://github.com/lalawila/betdog](https://github.com/lalawila/betdog)
{% endhint %}

## oracle

Odds list will be input by oracle such as chainlink when create condition.

The odds list example is `[4.27, 8.55, 1.42]`. That mean there are three outcomes. You will get `427` return if bet in outcome `0` . You will get `855` return if bet in outcome `1` . You will get `142` return if bet in outcome `2` .



## Providing Liquidity

### Adding Liquidity

Providing the amount of liquidity you want to add. Then smart contract will charge for equivalent value token and mint the amount of liquidity tokens for you.

valueToken = amountLiquidit \* totalValue / totalSupply

### Removing Liquidity

Providing the amount of liquidity you want to remove. Then smart contract will pay for equivalent value token and burn the amount of liquidity token for you.

#### Liquidity income

The 5‰ of winer’s rewards will be charge for liquidity income.



...

