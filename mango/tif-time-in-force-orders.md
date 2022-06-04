---
description: Time In Force orders
---

# ⏱ Time In Force (TIF)

Time In Force orders expire after a set amount of seconds. This provides security and confidence to market makers that stale or miss-priced orders will not get left on the order book in event of network degradation or any other issue.\
\
"placePerpOrder2" accepts an "expiryTimestamp" input which cancels an order after "n" seconds if it has not been executed.\
\
Time In Force orders are available in client, Mango Bowl, Python Explorer, and REST/WebSocket APIs. More information on these services is in Developer Resources.

The time on the validator is used to determine if a TIF order has expired.

When retrieving the current cluster time, be sure to use the "Processed" commitment level.\
\
\*\*Solana Explorer reports the cluster time using the "Finalised" commitment level, which is slightly different.

### Retrieving the Cluster Time

You can retrieve the current cluster time by using the getBlockTime endpoint. [https://docs.solana.com/developing/clients/jsonrpc-api#getblocktime](https://docs.solana.com/developing/clients/jsonrpc-api#getblocktime)
