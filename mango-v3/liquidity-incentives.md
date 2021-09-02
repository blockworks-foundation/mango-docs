# Liquidity Mining

## Overview

Mango Perps are the first perpetual futures contracts that are traded on a fully on chain order book. Providing liquidity on an order book is more of a skilled activity than providing on an AMM contract. In order to bootstrap the liquidity on Mango Perps, the Mango Protocol will disburse MNGO automatically to market makers according to an on-chain algorithm. This document describes the mechanism.

## Formula

```rust
/*
Definitions:
near touch - best bid if the order is a bid, best ask if order is an ask
best_initial - near touch when the order was placed
best_final - near touch when the order was canceled
time_initial - unix timestamp in seconds of when order was placed
time_final - unix timestamp of when order is being executed/canceled

max_depth_bps - the maximum depth behind near touch the program will reward
price - the price of the order

target_period_length - this is a param set by admin; It's how often we want
    rate adjustments. Analogous to Bitcoin difficulty adjustment period of 2 weeks
    
mngo_per_period - How much MNGO to award per period. This is the threshold 
    to determine when a period ends.
    
rate - this is the conversion MNGO per point. It's valid until mngo_per_period MNGO
    is paid out. Then the rate adjusts. 
*/

let best = if order.is_bid {
    max(best_initial, best_final)
} else {
    min(best_initial, best_final)
}

let dist_bps = abs(price - best) * 10000 / best;
let reverse_dist = max(max_depth_bps - dist_bps, 0);
let time_on_book = time_final - time_initial;

let points = reverse_dist * reverse_dist * time_on_book * quantity

let points_in_period = mngo_left_in_period * rate

// Check if this puts us over period threshold; If so rate adjust
if points >= points_in_period {
    user.mngo_accrued += mngo_left_in_period;
    points -= points_in_period;
    
    // There's no more MNGO for this period
    // Adjust rate (difficulty adjustment) and start new period
    let rate_adj = clamp((time_final - period_start) / target_period_length, 1/4, 4);
    rate = rate * rate_adj;
    period_start = time_final;
    mngo_left_in_period = mngo_per_period;
}

// Convert points to MNGO and award to user; limit to mngo_per_period
let mngo_earned = min(points * rate, mngo_per_period)
mngo_left_in_period -= mngo_earned
user.mngo_accrued += mngo_earned
```

## Explanation

This mechanism was inspired by the Bitcoin block rewards and difficulty mining adjustments that happen every 2016 blocks. In Bitcoin the target for 2016 blocks is 2 weeks. If it took longer than 2 weeks, then difficulty is adjusted down proportional to the time. If it took less than 2 weeks, difficulty is adjusted up. Similarly, we have a target time of 1 hour with a reward of 1,000 MNGO. When 1000 MNGO is distributed in a period, the time it took is compared against the target time of 1 hour. The rate is adjusted down if it took less than 1 hour and it's adjusted up if it took more than 1 hour.

## Motivation

This formula comes out of the intersection of our goals and technical limitations. Here are the goals:

* trustless and decentralized - should work even if initial devs disappear
* open and transparent - no special market maker agreements
* liquidity - tight spread, thick book that rivals centralized exchanges, even if volume is low
* easy to access - deploying the mango markets reference market maker should be profitable 



