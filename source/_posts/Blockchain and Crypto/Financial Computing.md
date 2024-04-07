---
title: Financial Computing
date: 2023-10-22 10:55:06
categories:
  - - 计算机
mathjax: true
tags:
  - B1
---
[[Order Books & AMM]]


<!-- more -->

# Introduction

## Money

**Purpose of Money**

- Medium of exchange
- Store of value
- Unit of account

**Forms of Money**

- Cash
- Coins
- Postage stamps
- Animals like cattle and sheep
- Sack of grains
- Feathers or shells
- Stones

## Markets

**Key Players of Financial Markets**

- Investors (lenders)
- Banks
- Borrowers (government and companies)

**Types of Financial Markets**

- Money markets (short-term)
- Capital markets (long-term)

**Types of Captal Markets**

- Equity markets (stocks / shares)
- Debt markets (bonds)

|               | Debt Financing                            | Equtiy Financing                                |
| ------------- | ----------------------------------------- | ----------------------------------------------- |
| Description   | Borrow money and pay interest             | Issue shares and pay dividend                   |
| Adavantages   | 1. Full control of business               | 1. Less risky                                   |
|               | 2. Tax benifit for interest payment       | 2. Reach investros and enhance creditability    |
|               | 3. Enhance creditworthniess               | 3. Assistance in company management             |
| Disadvantages | 1. Payment of interest and principal      | 1. Ownership by shareholders and profit sharing |
|               | 2. Need to pledge assets                  |                                                 |
|               | 3. Difficult for companies like start-ups |                                                 |
|               | 4. Affect company growth due to debt      |                                                 |

## Derivatives

**Main Purpose of Financial Derivatives**

- Hedge risk
- Speculating through derivatives

**Main Types of Financial Derivatives**

- Forward contract
- Future contract
- [[Options|Option contract]]
- Swap contract

# Time Value of Money

# Bonds

- **Face Value (Par Value):** Money received at maturity
- **Coupon Rate:** Interest rate of the bond
- **Coupon Date(s):** When to receive the interest payment (e.g., semi- annually)
- **Maturity (Date):** When the bond matures
- **Price:** Initial selling price

| Bond 1            |             |
| ----------------- | ----------- |
| **Face Value**    | $1,000      |
| **Coupon Rate**   | 5%          |
| **Coupon Dates**  | Annually    |
| **Matutiry Date** | 1 Jan. 2026 |

Today is 1 Jan. 2021.

**What interest will the owner receive after first year?**

\$50

**What will the owner receive on 1 Jan. 2026?**

\$1000 + \$1000 * 5% = \$1050

## Bond Price vs. Interest Rate

**The market interest goes up -> bond price decrease**

- The amount we receive in the future is a certain number.
- The present value (PV) of the amount decreases if the interest goes up.

**For a zero coupon bond with a face value of $5000 and a maturity of 2 years, what is the price if the interest rate is 5%?**
$$
{5000 \over {1.05^2}} = 4535
$$

## Examples

### Par Bond

| Par Bond                  |          |
| ------------------------- | -------- |
| **Face Value**            | $1,000   |
| **Coupon Rate**           | 10%      |
| **Coupon Dates**          | Annually |
| **Matutiry Date**         | 5 Years  |
| **Yield to Maturity (r)** | 10%      |

If the bond price is $999.9, then the **Yield To Maturity (YTM)** is 10%, which is equal to coupon rate.

YTM is a IRR, which makes its return balance cost.

| $n$  | $C$   | $C/(1+r)^n$ |
| ---- | ----- | ----------- |
| 1    | 100   | 90.9        |
| 2    | 100   | 82.6        |
| 3    | 100   | 75.1        |
| 4    | 100   | 68.3        |
| 5    | 1100  | 683.0       |
|      | Value | 999.9       |

### Discount Bond

| Discount Bond             |          |
| ------------------------- | -------- |
| **Face Value**            | $1,000   |
| **Coupon Rate**           | 10%      |
| **Coupon Dates**          | Annually |
| **Matutiry Date**         | 5 Years  |
| **Yield to Maturity (r)** | 15%      |

If the bond price is $832.5, then the YTM is 15%, which is greater than coupon rate.

It is called a discount bond.

| $n$  | $C$   | $C/(1+r)^n$ |
| ---- | ----- | ----------- |
| 1    | 100   | 87.0        |
| 2    | 100   | 75.6        |
| 3    | 100   | 65.8        |
| 4    | 100   | 57.2        |
| 5    | 1100  | 546.9       |
|      | Value | 832.5       |

### Premium Bond

| Premium Bond              |          |
| ------------------------- | -------- |
| **Face Value**            | $1,000   |
| **Coupon Rate**           | 10%      |
| **Coupon Dates**          | Annually |
| **Matutiry Date**         | 5 Years  |
| **Yield to Maturity (r)** | 5%       |

If the bond price is $1216.5, then the YTM is 5%, which is less than coupon rate.

It is called a premium bond.

| $n$  | $C$   | $C/(1+r)^n$ |
| ---- | ----- | ----------- |
| 1    | 100   | 95.2        |
| 2    | 100   | 90.7        |
| 3    | 100   | 86.4        |
| 4    | 100   | 82.3        |
| 5    | 1100  | 861.9       |
|      | Value | 1216.5      |

### Zero Coupon Bond

| Premium Bond              |          |
| ------------------------- | -------- |
| **Face Value**            | $1,000   |
| **Coupon Rate**           | 0%       |
| **Coupon Dates**          | Annually |
| **Matutiry Date**         | 5 Years  |
| **Yield to Maturity (r)** | 5%       |

If the coupon rate is 0, it is called a zero coupon bond.

| $n$  | $C$   | $C/(1+r)^n$ |
| ---- | ----- | ----------- |
| 1    | 0     | 0           |
| 2    | 0     | 0           |
| 3    | 0     | 0           |
| 4    | 0     | 0           |
| 5    | 1000  | 783.5       |
|      | Value | 783.5       |

# Stocks

**Equity = Total Assets − Total Liabilities**

## Captial Asset Priing Model (CAPM)

$$
R_a = R_f + \beta_a(R_m-R_f)
$$

$R_a$: Return of an asset

$R_f$: Risk-free return/rate

$\beta_a$: Risk. Larger than 1 means more risky than the market

$R_m - R_f$: Market risk premium.

Example: Suppose that the risk-free rate is 3%. A stock has a beta value of 1.2 (more risky than the market). The market is expected to rise by 10% per year. The expected return of the stock is:
$$
3\%+1.2(10\%-3\%)=11.4\%
$$
Here 11.4% is so-called $r$ in the calculation of stock price. For example, if a stock is held for one year, and the divident $D_1$ in year 1 is \$10 per share, and it is expected to be sold at $P_1 = 100$. The price should be (per share):
$$
V = {(D_1+P_1)}/{(1+r)}
$$

$$
{(10+100)}/{(1+0.114)}=98.7
$$
