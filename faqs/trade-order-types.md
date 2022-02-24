# Trade Order Types

There are several different order types available to Mango traders; this section will cover their behaviors.&#x20;



**Market**: Buy / Sell an asset for current price offered on the order book. Great for aping.&#x20;

**Limit**: Buy / Sell an asset for a specific price or better.&#x20;

![](<../.gitbook/assets/Screen Shot 2021-10-11 at 4.49.12 PM.png>)

This will place a buy bid on the order book for 100 SOL at $140. If the SOL price never drops to $140 or below the order will not get executed.&#x20;

**Reduce Only**: This ensures an order will only be executed to reduce or close a position and will never increase or open a new position. Check the Reduce Only box above the buy/sell button to apply. \
\
**Immediate or cancel (IOC)**: If an order is placed and not filled immediately, it's cancelled.\
\
**POST**: These limit orders are added to the order-book and earn maker fees if filled. POST orders will always be the 'maker'.

**Stop Loss**: This order sets a trigger for a market order if price goes below a given threshold. The threshold price is set in the 'Trigger Price' box. You may, for example, place a stop loss order to exit a position if the price falls 25% below entry to reduce downside exposure.&#x20;

![](<../.gitbook/assets/Screen Shot 2021-10-11 at 3.08.04 PM.png>)

In the order above, if the price of SOL fell to $120, then 400 SOL would be market sold.&#x20;

**Stop Limit**: This order works similarly to a stop loss, although a limit order is triggered at a given price target rather than a market order. The price on the limit order is set in the 'Price' field.&#x20;

![](<../.gitbook/assets/Screen Shot 2021-10-11 at 4.21.44 PM.png>)

**Take Profit**: A take profit order will set a trigger for a market order if price goes above a given threshold. This can close a portion or all of a position for a profit.

&#x20;![](<../.gitbook/assets/Screen Shot 2021-10-11 at 4.09.45 PM.png>)

Here, a successful long 100 SOL position would be market closed if the price of SOL reached 220 or higher.&#x20;

**Take Profit Limit**: This order works similarly to take profit, although a limit order is placed with a provided price rather than a market order.&#x20;

![](<../.gitbook/assets/Screen Shot 2021-10-11 at 4.13.54 PM.png>)

Above, a limit sell order of 100 SOL at price 230 would be placed if the SOL price reached 220 or higher.&#x20;









