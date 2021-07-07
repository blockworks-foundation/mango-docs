# Liquidity Incentives

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
time_final - unix timestamp in seconds of when order was placed

max_depth_bps - the maximum depth behind near touch the program will reward
scaler - some scaling constant
price - the price of the order
*/


let best = if order.is_bid {
    max(best_initial, best_final)
} else {
    min(best_initial, best_final)
}

let dist_bps = abs(price - best) * 10000 / best;
let reverse_dist = max(max_depth_bps - dist_bps, 0);
let time_on_book = time_final - time_initial;

let points = reverse_dist * reverse_dist * time_on_book * quantity * scaler

```

## Motivation

This formula comes out of the intersection of our goals and technical limitations. Here are the goals:

* trustless and decentralized - should work even if the Mango team disappears
* open and transparent - no special market maker agreements
* liquidity - tight spread, thick book that rivals centralized exchanges, even if volume is low
* easy to access - deploying the mango markets reference market maker should be profitable 



