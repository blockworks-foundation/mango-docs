# ⛏ Liquidity Mining

* Liquidity Mining rewards MNGO tokens to limit orders on the perpetual futures order book within a specified depth of mark price.
* Orders closer to mark price are rewarded more favourably.
* Liquidity Mining is open to everyone.

## Overview

Mango Markets Perpetual Futures are the first perpetual futures contracts that are traded on a fully on-chain order book. Providing liquidity on an order book is more of a skilled activity than providing on an AMM contract. In order to bootstrap the liquidity on Mango Markets Perpetual Futures, the Mango Protocol will distribute MNGO tokens automatically to market makers according to an on-chain algorithm. This document describes the mechanism. But broadly:

* Quoting closer to the top of book is rewarded much more generously.
* You do not have to be filled to get the rewards (you get points every time you cancel OR get filled).
* There are open source market making bots developed by and used by Mango Markets devs.

## Formula

The liquidity mining formula rewards the top N contracts on the bids and asks weighted quadratically to favour those closer to top of book. Your depth on the book is assessed when you place an order on the book and when you get filled/cancel the order and the worse of the two is used in calculating your liquidity mining points.

The full source code is here: [https://github.com/blockworks-foundation/mango-v3/blob/3d59136bb025252c5b20e06064526ff115471f2d/program/src/state.rs#L1621](https://github.com/blockworks-foundation/mango-v3/blob/3d59136bb025252c5b20e06064526ff115471f2d/program/src/state.rs#L1621)

Here's the abridged formula:

```
// Definitions:
// max_depth - the max number of contracts rewarded in this PerpMarket
// depth - how many contracts are ahead your order on the bids or asks
// quantity - quantity of the order or the 

let depth = max(depth_initial, depth_final)
let depth_factor = max_depth - depth

if depth_factor <= 0 {
    return;
}

// time the order was on the book in seconds
let time_factor = time_final - time_initial

// limit to rewarding the amount of your order that remains in the window
let quantity_factor = min(quantity, depth_factor)

let points = depth_factor.pow(2) * time_factor * quantity_factor

// Convert the points into MNGO; adjust the rate of disbursement
return convert_points(points)
```

## Example

Suppose `max_depth = 20000, mngo_per_hour=250` and the bids look like this:

```
1000 MNGO @ 0.30
5000 MNGO @ 0.28
10000 MNGO @ 0.26
```

Suppose I place a bid for 8,000 MNGO at price 0.27 at time t=100. Then I get filled at time t=110. Then:

```
depth_initial = 6000
depth_final = 0  // since there are no bids ahead of me if I got filled
time_intial = 100
time_final = 110
quantity = 8000

depth = max(6000, 0) = 6000
depth_factor = 20000 - 6000 = 14000
time_factor = 110 - 100 = 10
quantity_factor = min(8000, 14000) = 8000

points = 14,000 ^ 2 * 10 * 8000 = 1.568e13
```

The points earned are 1.568e13 which is a totally meaningless number until it is normalized using a rate parameter and adjusted constantly such that a total of 250 MNGO per hour is distributed.

## Rate Adjustments

This mechanism was inspired by the Bitcoin block rewards and difficulty mining adjustments that happen every 2016 blocks. In Bitcoin the target for 2016 blocks is 2 weeks. If it took longer than 2 weeks, then difficulty is adjusted down proportional to the time. If it took less than 2 weeks, difficulty is adjusted up. Similarly, we have a target time of 1 hour with a reward of 1,000 MNGO. When 1000 MNGO is distributed in a period, the time it took is compared against the target time of 1 hour. The rate is adjusted down if it took less than 1 hour and it's adjusted up if it took more than 1 hour.

## Motivation

This formula comes out of the intersection of our goals and technical limitations. Here are the goals:

* Trustless and decentralized - should work even if initial devs disappear.
* Open and transparent - no special market maker agreements.
* Liquidity - tight spread, thick book that rivals centralized exchanges, even if volume is low.
* Easy to access - deploying the Mango Markets reference market maker should be profitable.

## Liquidity Mining Specs

Max Depth is the amount of liquidity on \*each\* side of the book that is rewarded.

| Perp Contract |  Max Depth  | MNGO per hour |
| :-----------: | :---------: | :-----------: |
|      MNGO     | 50,000 MNGO |       5       |
|      BTC      |    5 BTC    |       20      |
|      ETH      |    50 ETH   |       15      |
|      SOL      |  3,000 SOL  |       50      |
|      SRM      |  20,000 SRM |       5       |
|      RAY      |  12,000 RAY |       5       |
|      FTT      |  1,000 FTT  |       5       |
|      ADA      |  50,000 ADA |       5       |
|      BNB      |   100 BNB   |       5       |
|      AVAX     |  1,500 AVAX |       10      |
|      GMT      |  50,000 GMT |       50      |
