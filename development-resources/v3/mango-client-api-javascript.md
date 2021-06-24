# Mango Client API \(javascript\)

### Installation

```text
yarn add @blockworks-foundation/mango-client
```

### Usage Example

This example assumes that you have a wallet that is already setup with devnet tokens. The private key should be stored in `~/.config/solana/devnet.json`. Visit [https://v3.mango.markets/](https://v3.mango.markets/) and connect with the wallet to fund your margin account so that you can place orders. You can find the full source code in [example.ts](https://github.com/blockworks-foundation/mango-client-v3/blob/main/src/example.ts).

```javascript
// Fetch orderbooks
const bids = await perpMarket.loadBids(connection);
const asks = await perpMarket.loadAsks(connection);

// L2 orderbook data
for (const [price, size] of bids.getL2(20)) {
  console.log(price, size);
}

// L3 orderbook data
for (const order of asks) {
  console.log(
    order.owner.toBase58(),
    order.orderId.toString('hex'),
    order.price,
    order.size,
    order.side, // 'buy' or 'sell'
  );
}

// Place order
await client.placePerpOrder(
  mangoGroup,
  mangoAccount,
  mangoGroup.mangoCache,
  perpMarket,
  owner,
  'buy', // or 'sell'
  39000,
  0.0001,
  'limit', // or 'ioc' or 'postOnly'
);

// retrieve open orders for account
const openOrders = await perpMarket.loadOrdersForAccount(
  connection,
  mangoAccount,
);

// cancel orders
for (const order of openOrders) {
  await client.cancelPerpOrder(
    mangoGroup,
    mangoAccount,
    owner,
    perpMarket,
    order,
  );
}

// Retrieve fills
for (const fill of await perpMarket.loadFills(connection)) {
  console.log(
    fill.owner.toBase58(),
    fill.maker ? 'maker' : 'taker',
    fill.baseChange.toNumber(),
    fill.quoteChange.toNumber(),
    fill.longFunding.toFixed(3),
    fill.shortFunding.toFixed(3),
  );
}
```

