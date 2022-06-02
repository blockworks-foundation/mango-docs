# ⏱ TIF (Time In Force) orders

Time In Force orders expire after a set amount of seconds. This provides security and confidence to market makers that stale or miss-priced orders will not get left on the orderbook in events of network degradation or any other issue.\
\
"placePerpOrder2" accepts an "expiryTimestamp" input which cancels an order after "n" seconds if it has not been executed.\
\
TIF orders are available in client, Mango Bowl, Python Explorer, and REST/websocket APIs. More information on these services in Developer Resources.
