# PNL Calculation

The total PNL \(Profit & Loss\) calculation is a combination of realized and unrealized gains or losses from your trades.

Given these trades:

1. BUY 1 ETH at 1000 USDT
2. SELL 1 BTC at 10,000 USDT
3. BUY 0.5 BTC at 11,000 USDT

First, calculate net change for each token:

BTC:   `-1 + 0.5 = -0.5`  
ETH:   `+1`  
USDT: `-1000 + 10000 - 5500 = +3500`

If the current price of BTC is now at $9,900 and the current price of ETH is at $900 then:

Total PNL = `(-0.5 BTC * $9,900) + (1 ETH * $900) + $,3500 = -$550`

\*The total PNL is calculated based on trades placed on Mango Markets after March 15th, 2021. If you placed trades before this date and are seeing an inaccurate PNL, you can move your assets to a new wallet in Sollet.io and connect to Mango with the new Sollet wallet to reset your trade history.

