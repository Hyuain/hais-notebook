---
title: Order Books & AMM
date: 2023-04-05 11:38:30
categories:
  - - 计算机
tags:
  - BX
---

# Order Book

> Order Book 不是为链上交易设计的。他的价格设置不够灵活、交易不够快速，Gas 消耗过大。
## Price Discovery

买东西的时候，向不同的卖家（Sellers）询价（Asking for prices）。作为买家（Buyer），自然会去寻找最低的价格（Lowest Price）。

**交易所（Exchanges）** 的目的是 **将这些活动都集中在一个地方**，并高效促进：

- **流动性（Liquidity）**：资产、抵押品等在不影响市场价格的情况下转变为现金的便利程度；
- **价格发现（Price Discovery）**：资产现货价格的设定过程。

## Order Book and Price

- Order Book 包含了买方和卖方所需的所有信息；
- 每对交易对应一个 Order Book；
- **Bid 指的是买入价**，包含了所有买入订单；**Ask 指的是卖出价**，包含了所有卖出信息。

**Ask Price（卖出价）通常会高于买入价（Bid Price）**

## Price Time Priority

**Order Price**：最高买入价被称为 Best Bid，最低卖出价被称为 Best Ask。

**Order Time**：当价格相同时，较早提交的 Order 的优先级更高。

## Bid Ask Spread

Best Bid 和 Best Ask 之间的差值叫做 **买卖差价（Bid Ask Spread）**。

Best Bid 和 Best Ask 的平均值称为 **中间价（Mid Price）**。

## Order States: Passive & Aggressive

**Passive Orders** 不会立即成交，他们的限价（Limit Price）处于他们应该在的区域（Bid Price 或 Ask Price），而没有跨到对面去。**Passive Orders 为 Order Book 增加了流动性。**

反之，**Aggressive Orders** 会立即成交，他们会**减少 Order Book 的流动性。**

## Order Types: Limit & Market

**限价单（Limit Order）** 会指定成交的 **最坏价格**，比如对于买方来说，如果他提交一个订单：

```
BUY ETH @ 3200 USDT per ETH
```

那么价格在高于 3200 USDT 时，该订单不会成交。

**市价单（Market Order）** 则立即以市场上的最佳价格成交，因此 Market Order 永远都是 Aggressive Order，比如：

```
BUY ETH @ Market Price
```

> 最好只使用 Limit Order，不要使用 Market Order，避免 **滑点（Slippage）**。

## Exchange Fees

交易所从交易手续费中获利，交易手续费分为两种：

- **挂单成交手续费（Maker Fees）**：提供一个 Passive Order，他人成交；
- **吃单成交手续费（Taker Fees）**：提交一个 Aggressive Order 并成交。

有时候交易所为了鼓励用户提供流动性，会提供 **Negative Maker Fees** 或称 **Maker Rebate**。

**Market Makers** 提供大量流动性，从 Market Rebate 中获利；他们也会同时提供 Bid 和 Ask，低买高卖赚取差价。

# Automated Market Makers (AMMs)

> 一种线上的合约，自动为两种资产确定合适的交易价格

## Constant Product AMM (CPAMM)

> `x * y = K`，其中 `x` 和 `y` 是两种资产的数量。资产的价值仅由其数量（供需）决定。

比如有两组人，一组有 50,000 个包，一组有 50,000 个 iPhone，他们需要找到一个公平的方式来交换彼此的资产。

在 CPAMM 中，应用公式 `x * y = K`，其中 `x` 是包的数量 50,000，`y` 是 iPhone 的数量 50,000，`K` 为一个常数 2.5 bn。

那么按照公式，每个包和每个 iPhone 的价值均为 1。

现在开始进行交换，当我们想要获得 7,000 部 iPhones 时，我们剩下的包的数量 `x` 为：

```
x = 2.5bn / y
  = 2.5bn / 57,000
  = 50,000 - 6,140
```

因此，我们需要 6,140 个包进行交换。同时可以计算出新的包的价值为 1.14，iPhone 的价值为 0.88。

### Liquidity & Slippage

**流动性池（Liquidity Pool）** 是一个 **智能合约（Smart Contract）**，他持有一定数量的两种代币，并尝试得到公平的价格。

由于流动性池大小的不同，资产交换的价格也会随之波动，产生价格滑点。流动性越高，滑点越小。

### Impermanent Loss

用户在复杂的交换的过程中可能产生损失，由于池子流动性的变化，他的资产价值可能发生折损。如果用户持续持有该池中的代币，而不撤出他的流动性，那么这种损失就是 **非永久性损失（Impermanent Loss）**。

