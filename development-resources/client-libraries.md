# Client Libraries

The following client libraries are currently independently maintained by the community:

* TypeScript [https://github.com/blockworks-foundation/mango-client-v3](https://github.com/blockworks-foundation/mango-client-v3)
* Python [https://github.com/blockworks-foundation/mango-explorer](https://github.com/blockworks-foundation/mango-explorer)
* C# [https://github.com/bmresearch/Solnet.Mango/](https://github.com/bmresearch/Solnet.Mango/)
* C++ [https://github.com/mschneider/solcpp](https://github.com/mschneider/solcpp)

In an attempt to create uniformity we have documented the major functions, which are available across all libraries.

### Package Install

Add your preferred library to your project.

{% tabs %}
{% tab title="TypeScript" %}
Using npm:

```
npm install @blockworks-foundation/mango-client
```

Using yarn:

```
yarn add @blockworks-foundation/mango-client
```
{% endtab %}

{% tab title="C#" %}
Solnet.Mango can be installed via the Nuget Package Manager or by using any of the following command lines.\


```
Install-Package Solnet.Mango -Version 5.0.1 // Package Manager
dotnet add package Solnet.Mango --version 5.0.1 // .NET CLI
<PackageReference Include="Solnet.Mango" Version="5.0.1" /> // PackageReference
paket add Solnet.Mango --version 5.0.1 // Paket CLI
#r "nuget: Solnet.Mango, 5.0.1" // Script & Interactive
```
{% endtab %}

{% tab title="Python" %}
```
pip install mango-explorer
```
{% endtab %}
{% endtabs %}

### Subscribe to an Order Book

To get the order book you need to retrieve the bids and asks independently. Each side of the order book has a max capacity of 512 orders.\
\
When the 512 capacity is reached, orders that are the furthest away from the top will be removed when a new order is added and the Out Event is fired when an order is removed.\
\
Orders can be set to expire using the Time-In-Force

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  BookSide,
  BookSideLayout,
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
} from '../src';
import { Commitment, Connection } from '@solana/web3.js';
import configFile from '../src/ids.json';

async function subscribeToOrderbook() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  // load group & market
  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  // subscribe to bids
  connection.onAccountChange(perpMarketConfig.bidsKey, (accountInfo) => {
    const bids = new BookSide(
      perpMarketConfig.bidsKey,
      perpMarket,
      BookSideLayout.decode(accountInfo.data),
    );

    // print L2 orderbook data
    for (const [price, size] of bids.getL2(20)) {
      console.log(price, size);
    }

    // print L3 orderbook data
    for (const order of bids) {
      console.log(
        order.owner.toBase58(),
        order.orderId.toString('hex'),
        order.price,
        order.size,
        order.side, // 'buy' or 'sell'
      );
    }
  });
}

subscribeToOrderbook();
```


{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Connect Streaming Client
await mangoClient.StreamingRpcClient.ConnectAsync();

//Market
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync("DtEcjPLyD4YtTBB4q8xwFZ9q49W89xZCZtJyrGebi5t8"); // BTC-PERP

//Subscribe to each side of the order book
await mangoClient.SubscribeOrderBookSideAsync((_, side, _) =>
{
    List<OpenOrder> bids = side.GetOrders();
}, perpMarket.ParsedResult.Bids, Commitment.Processed);

await mangoClient.SubscribeOrderBookSideAsync((_, side, _) =>
{
    List<OpenOrder> asks = side.GetOrders();
}, perpMarket.ParsedResult.Asks, Commitment.Processed);
```

OpenOrder model:

```csharp
public BigInteger OrderId; //The order id.    
public ulong ClientOrderId; // The client's order id.
public int OrderIndex; // The index of the order within the lists of order data in the Mango account
public long RawPrice; //The raw value for the price of the order.
public long RawQuantity; //The raw value for the quantity of the order.
public PublicKey Owner; //The owner of this order.
public ulong Timestamp; //The timestamp.
public ulong ExpiryTimestamp; //Timestamp of expiry.
public byte TimeInForce; //The time in force.
public PerpOrderType OrderType; //The order type.
```
{% endtab %}

{% tab title="Python" %}
This code will connect to the _devnet_ cluster and will print out the latest orderbook every time the orderbook changes, and will exit after 60 seconds.

```python
import datetime
import mango
import time

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    market = mango.market(context, "BTC-PERP")
    subscription = market.on_orderbook_change(context, lambda ob: print("\n", datetime.datetime.now(), "\n", ob))

    time.sleep(60)

    subscription.dispose()

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Get an Order Book snapshot

You can also retrieve a snapshot of the order book.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
} from '../src';
import { Commitment, Connection } from '@solana/web3.js';
import configFile from '../src/ids.json';

async function snapshotOrderbook() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  // load group & market
  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  // load bids
  const bids = await perpMarket.loadBids(connection);

  // print L2 orderbook data
  for (const [price, size] of bids.getL2(20)) {
    console.log(price, size);
  }

  // print L3 orderbook data
  for (const order of bids) {
    console.log(
      order.owner.toBase58(),
      order.orderId.toString('hex'),
      order.price,
      order.size,
      order.side,
    );
  }
}

snapshotOrderbook();
```


{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Market
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync("DtEcjPLyD4YtTBB4q8xwFZ9q49W89xZCZtJyrGebi5t8"); // BTC-PERP

//Request order book sides
AccountResultWrapper<OrderBookSide> bidsRequest = await mangoClient.GetOrderBookSideAsync(perpMarket.ParsedResult.Bids, Commitment.Processed); // Bids side of the orderbook
AccountResultWrapper<OrderBookSide> asksRequest = await mangoClient.GetOrderBookSideAsync(perpMarket.ParsedResult.Asks, Commitment.Processed); // Asks side of the orderbook

List<OpenOrder> bids = bidsRequest.ParsedResult.GetOrders();
List<OpenOrder> asks = asksRequest.ParsedResult.GetOrders();
```
{% endtab %}

{% tab title="Python" %}
This code will connect to the _devnet_ cluster, fetch the orderbook for BTC-PERP, and print out a summary of it:

```python
import mango

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    market = mango.market(context, "BTC-PERP")
    print(market.fetch_orderbook(context))
```
{% endtab %}
{% endtabs %}

### Subscribe to Market Fills

You can subscribe to a markets' event queue. When the event queue is updated, the entire queue is received.\
\
You can filter the event queue for fills and use the sequence number to filter new fills added to the queue.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
  PerpEventQueue,
  PerpEventQueueLayout,
  ZERO_BN,
} from '../src';
import { Commitment, Connection } from '@solana/web3.js';
import configFile from '../src/ids.json';
import { ParsedFillEvent } from '../src/PerpMarket';

async function subscribeToOrderbook() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  // load group & market
  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  // subscribe to event queue
  const lastSeenSeqNum = ZERO_BN;
  connection.onAccountChange(perpMarketConfig.eventsKey, (accountInfo) => {
    const queue = new PerpEventQueue(
      PerpEventQueueLayout.decode(accountInfo.data),
    );
    const fills = queue
      .eventsSince(lastSeenSeqNum)
      .map((e) => e.fill)
      .filter((e) => !!e)
      .map((e) => perpMarket.parseFillEvent(e) as ParsedFillEvent);

    for (const fill of fills) {
      console.log(`New fill for ${fill.quantity} at ${fill.price}`);
    }
  });
}

subscribeToOrderbook();
```
{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Connect Streaming Client
await mangoClient.StreamingRpcClient.ConnectAsync();

//Market and Event Queue
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync("DtEcjPLyD4YtTBB4q8xwFZ9q49W89xZCZtJyrGebi5t8"); // BTC-PERP

//Subscribe the event queue
await mangoClient.SubscribeEventQueueAsync((_, queue, _) =>
{
    foreach (Event evt in queue.Events)
    {
        switch (evt)
        {
            case FillEvent fill:                    
		 Console.Write(fill.Price);		   	           
	         break;
        }
    }
}, perpMarket.ParsedResult.EventQueue, Commitment.Processed);
```

FillEvent model:

```csharp
public Side TakerSide;//The side from the taker's point of view.
public byte MakerSlot;// The slot of the order in the maker's orders.
public bool MakerOut; // Whether the maker's order quantity has nothing left to fill.
public PublicKey Maker; // The maker's public key.
public BigInteger MakerOrderId; // The maker's order id.
public ulong MakerClientOrderId; // The maker's client order id.
public I80F48 MakerFee; // The maker fee.
public long BestInitial; // The best bid/ask at the time the maker order was placed.
public ulong MakerTimestamp; // Timestamp of when the maker order was placed.
public PublicKey Taker; // The taker's public key.
public BigInteger TakerOrderId; // The taker's order id.
public ulong TakerClientOrderId; // The taker's client order id.
public I80F48 TakerFee; // The taker fee.
public long Price; // The price of the fill.
public long Quantity; // The quantity that was filled.
```
{% endtab %}

{% tab title="Python" %}
This code will connect to the _devnet_ cluster and print out every fill that happens. It will exit after 60 seconds.

```python
import datetime
import mango
import time

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    market = mango.market(context, "BTC-PERP")
    subscription = market.on_fill(context, lambda ob: print("\n", datetime.datetime.now(), "\n", ob))

    time.sleep(60)

    subscription.dispose()

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Get a Market fills snapshot

You can also retrieve a snapshot of a markets' event queue.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
} from '../src';
import { Commitment, Connection } from '@solana/web3.js';
import configFile from '../src/ids.json';

async function snapshotFills() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  // load group & market
  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  const fills = await perpMarket.loadFills(connection);
  for (const fill of fills) {
    console.log(`Filled ${fill.quantity} at ${fill.price}`);
  }
}

snapshotFills();
```
{% endtab %}

{% tab title="C#" %}
```csharp
IRpcClient rpcClient = ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync("DtEcjPLyD4YtTBB4q8xwFZ9q49W89xZCZtJyrGebi5t8"); // BTC-PERP
AccountResultWrapper<EventQueue> eventQueue = await mangoClient.GetEventQueueAsync(perpMarket.ParsedResult.EventQueue, Commitment.Processed);

List<FillEvent> fills = eventQueue.ParsedResult
    .Events
    .Where(f => f.EventType == EventType.Fill)
    .Cast<FillEvent>()
    .ToList();

foreach (FillEvent fill in fills)
{
    Console.WriteLine($"Filled {fill.Quantity} @ {fill.Price}");
}
```
{% endtab %}

{% tab title="Python" %}
A 'fill' is when a maker order from the orderbook is matched with an incoming taker order. It can be useful to see these.

This code will connect to the _devnet_ cluster and fetch all recent events. It will then show all the fill events.

```python
import mango

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    market = mango.market(context, "BTC-PERP")
    event_queue = mango.PerpEventQueue.load(context, market.event_queue_address, market.lot_size_converter)
    print(event_queue.fills)

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Post Orders

Orders are placed by adding an instruction to a transaction. You can add multiple instructions to a single transaction.\
\
There is no additional fee per instruction, your account is only charged per transaction.\
\
You can have a max of 64 open orders across all perpetual markets.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
} from '../src';
import { Account, Commitment, Connection, PublicKey } from '@solana/web3.js';
import configFile from '../src/ids.json';
import * as os from 'os';
import * as fs from 'fs';

async function postOrders() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  const payer = new Account(
    JSON.parse(
      fs.readFileSync(
        os.homedir() + '/.config/solana/my-mainnet.json',
        'utf-8',
      ),
    ),
  );

  // load group, cache, account, market
  const mangoAccountPk = new PublicKey('YOUR_MANGOACCOUNT_KEY');
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const mangoAccount = await client.getMangoAccount(
    mangoAccountPk,
    mangoGroup.dexProgramId,
  );

  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  // post an ask for 1btc at 50000
  await client.placePerpOrder2(
    mangoGroup,
    mangoAccount,
    perpMarket,
    payer,
    'sell',
    50000,
    1,
    {
      orderType: 'postOnly',
    },
  );
}

postOrders();
```
{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Token
PublicKey perpMarketPublicKey = new PublicKey("4GkJj2znAr2pE2PBbak66E12zjCs2jkmeafiJwDVM9Au"); // SRM-PERP
AccountResultWrapper<MangoGroup> mangoGroup = await mangoClient.GetMangoGroupAsync(Constants.MangoGroup);
IPythClient pythClient = Solnet.Pyth.ClientFactory.GetClient(rpcClient, streamingRpcClient);
AccountResultWrapper<MappingAccount> mappingAccount = await pythClient.GetMappingAccountAsync(Solnet.Pyth.Constants.MappingAccount);
MultipleAccountsResultWrapper<List<ProductAccount>> productAccounts = await pythClient.GetProductAccountsAsync(mappingAccount.ParsedResult);
ProductAccount productAccount = productAccounts.ParsedResult.First(x => x.Product.Symbol.Contains("SRM"));
PublicKey oracle = mangoGroup.ParsedResult.Oracles.First(x => x.Key == productAccount.PriceAccount.Key);
int perpetualMarketIndex = mangoGroup.ParsedResult.Oracles.IndexOf(oracle);
int marketIndex = mangoGroup.ParsedResult.GetPerpMarketIndex(perpMarketPublicKey);
TokenInfo token = mangoGroup.ParsedResult.Tokens[marketIndex];
TokenInfo quoteToken = mangoGroup.ParsedResult.GetQuoteTokenInfo();

//Market and Pricing
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync(perpMarketPublicKey);
(long bidPrice, long bidQuantity) = perpMarket.ParsedResult.UiToNativePriceQuantity(1.5, 14156.28539071348, token.Decimals, quoteToken.Decimals);
(long askPrice, long askQuantity) = perpMarket.ParsedResult.UiToNativePriceQuantity(2.1, 14156.28539071348, token.Decimals, quoteToken.Decimals);

//Solana Wallet
PublicKey solanaWalletPublicKey = new PublicKey("__your_solana_wallet_public_key__");

//Mango Account
PublicKey mangoAccountPublicKey = new PublicKey("__your_mango_account_public_key__");
AccountResultWrapper<MangoAccount> mangoAccount = await mangoClient.GetMangoAccountAsync(mangoAccountPublicKey, Commitment.Processed);

//Expiry
ulong expiryTimestamp = (ulong)new DateTimeOffset(DateTime.UtcNow.AddSeconds(250)).ToUnixTimeSeconds();

MangoProgram mangoProgram = MangoProgram.CreateMainNet();

TransactionInstruction bid = mangoProgram.PlacePerpOrder2(
                mangoGroup: Constants.MangoGroup,
                mangoAccount: mangoAccountPublicKey,
                owner: solanaWalletPublicKey,
                mangoCache: Constants.MangoCache,
                perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
                bids: perpMarket.ParsedResult.Bids,
                asks: perpMarket.ParsedResult.Asks,
                eventQueue: perpMarket.ParsedResult.EventQueue,
                openOrdersAccounts: mangoAccount.ParsedResult.SpotOpenOrders,
                side: Side.Buy,
                orderType: PerpOrderType.PostOnly,
                price: bidPrice, 
                maxBaseQuantity: bidQuantity,
                clientOrderId: 1_000_001ul,
                expiryTimestamp: expiryTimestamp,
                reduceOnly: false
            );

TransactionInstruction ask = mangoProgram.PlacePerpOrder2(
	            mangoGroup: Constants.MangoGroup,
	            mangoAccount: mangoAccountPublicKey,
	            owner: solanaWalletPublicKey,
	            mangoCache: Constants.MangoCache,
	            perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
	            bids: perpMarket.ParsedResult.Bids,
	            asks: perpMarket.ParsedResult.Asks,
	            eventQueue: perpMarket.ParsedResult.EventQueue,
	            openOrdersAccounts: mangoAccount.ParsedResult.SpotOpenOrders,
	            side: Side.Sell,
	            orderType: PerpOrderType.PostOnly,
	            price: askPrice,
	            maxBaseQuantity: askQuantity,
	            clientOrderId: 1_000_002ul,
	            expiryTimestamp: expiryTimestamp,
	            reduceOnly: false
            );

//Build Transaction
RequestResult<ResponseValue<BlockHash>> blockhashRequest = await rpcClient.GetRecentBlockHashAsync();
TransactionBuilder txBuilder = new TransactionBuilder().SetFeePayer(solanaWalletPublicKey).SetRecentBlockHash(blockhashRequest.Result.Value.Blockhash);

txBuilder.AddInstruction(bid);
txBuilder.AddInstruction(ask);

Wallet solanaWallet = new SolanaKeyStoreService().RestoreKeystoreFromFile("__location_of_your_json_wallet_file__");
byte[] txBytes = txBuilder.Build(new List<Account>() { solanaWallet.Account });

//Send Transaction
RequestResult<string> transaction = await mangoClient.RpcClient.SendTransactionAsync(txBytes, true, Commitment.Processed);
```
{% endtab %}

{% tab title="Python" %}
This code will load the 'example' wallet, connect to the _devnet_ cluster, place an order and wait for the order's transaction signature to be confirmed.

The order is an IOC ('Immediate Or Cancel') order to buy 2 SOL-PERP at $1. This order is unlikely to be filled (SOL currently costs substantially more than $1), but that saves having to show order cancellation code here.

Possible `order_type` values are:

* mango.OrderType.LIMIT
* mango.OrderType.IOC
* mango.OrderType.POST\_ONLY
* mango.OrderType.MARKET (it's nearly always better to use IOC and a price with acceptable slippage)
* mango.OrderType.POST\_ONLY\_SLIDE (only available on perp markets)



```python
import decimal
import mango

from solana.publickey import PublicKey

# Use our hard-coded devnet wallet for DeekipCw5jz7UgQbtUbHQckTYGKXWaPQV4xY93DaiM6h.
# For real-world use you'd load the bytes from the environment or a file. Later we use
# its Mango Account at HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL.
wallet = mango.Wallet(bytes([67,218,68,118,140,171,228,222,8,29,48,61,255,114,49,226,239,89,151,110,29,136,149,118,97,189,163,8,23,88,246,35,187,241,107,226,47,155,40,162,3,222,98,203,176,230,34,49,45,8,253,77,136,241,34,4,80,227,234,174,103,11,124,146]))

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    group = mango.Group.load(context)
    account = mango.Account.load(context, PublicKey("HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL"), group)
    market_operations = mango.operations(context, wallet, account, "SOL-PERP", dry_run=False)

    # Try to buy 2 SOL for $1.
    order = mango.Order.from_values(side=mango.Side.BUY,
                                    price=decimal.Decimal(1),
                                    quantity=decimal.Decimal(2),
                                    order_type=mango.OrderType.IOC)
    print("Placing order:", order)
    placed_order_signatures = market_operations.place_order(order)

    print("Waiting for place order transaction to confirm...\n", placed_order_signatures)
    mango.WebSocketTransactionMonitor.wait_for_all(
            context.client.cluster_ws_url, placed_order_signatures, commitment="processed"
        )

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Cancel an Order

You can cancel a specific order by OrderID, Side or ClientID.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  MangoClient,
} from '../src';
import { Account, Commitment, Connection, PublicKey } from '@solana/web3.js';
import configFile from '../src/ids.json';
import * as os from 'os';
import * as fs from 'fs';
import { BN } from 'bn.js';

async function cancelOrder() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  const payer = new Account(
    JSON.parse(
      fs.readFileSync(
        os.homedir() + '/.config/solana/my-mainnet.json',
        'utf-8',
      ),
    ),
  );

  // load group, account, market
  const mangoAccountPk = new PublicKey('YOUR_MANGOACCOUNT_KEY');
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const mangoAccount = await client.getMangoAccount(
    mangoAccountPk,
    mangoGroup.dexProgramId,
  );

  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );
  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  const clientOrderId = 123;
  // post an ask for 1btc at 50000
  await client.placePerpOrder2(
    mangoGroup,
    mangoAccount,
    perpMarket,
    payer,
    'sell',
    50000,
    1,
    {
      orderType: 'postOnly',
      clientOrderId: clientOrderId,
    },
  );

  // load open perp orders
  const orders = await perpMarket.loadOrdersForAccount(
    connection,
    mangoAccount,
  );

  // cancel by client id
  const order = orders.find((order) => order.clientId == new BN(clientOrderId));
  await client.cancelPerpOrder(
    mangoGroup,
    mangoAccount,
    payer,
    perpMarket,
    order,
  );
}

cancelOrder();
```


{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Solana Wallet
PublicKey solanaWalletPublicKey = new PublicKey("__your_solana_wallet_public_key__");

//Mango Account
PublicKey mangoAccountPublicKey = new PublicKey("__your_mango_account_public_key__");
AccountResultWrapper<MangoAccount> mangoAccount = await mangoClient.GetMangoAccountAsync(mangoAccountPublicKey, Commitment.Processed);

//Market
PublicKey perpMarketPublicKey = new PublicKey("4GkJj2znAr2pE2PBbak66E12zjCs2jkmeafiJwDVM9Au"); // SRM-PERP
AccountResultWrapper<MangoGroup> mangoGroup = await mangoClient.GetMangoGroupAsync(Constants.MangoGroup);
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync(perpMarketPublicKey);

//Market Index
IPythClient pythClient = Solnet.Pyth.ClientFactory.GetClient(rpcClient, streamingRpcClient);
AccountResultWrapper<MappingAccount> mappingAccount = await pythClient.GetMappingAccountAsync(Solnet.Pyth.Constants.MappingAccount);
MultipleAccountsResultWrapper<List<ProductAccount>> productAccounts = await pythClient.GetProductAccountsAsync(mappingAccount.ParsedResult);
ProductAccount productAccount = productAccounts.ParsedResult.First(x => x.Product.Symbol.Contains("SRM"));
PublicKey oracle = mangoGroup.ParsedResult.Oracles.First(x => x.Key == productAccount.PriceAccount.Key);
int perpetualMarketIndex = mangoGroup.ParsedResult.Oracles.IndexOf(oracle);

MangoProgram mangoProgram = MangoProgram.CreateMainNet();

TransactionInstruction cancelById = mangoProgram.CancelPerpOrder(
    mangoGroup: Constants.MangoGroup,
    mangoAccount: mangoAccountPublicKey,
    owner: solanaWalletPublicKey,
    perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
    bids: perpMarket.ParsedResult.Bids,
    asks: perpMarket.ParsedResult.Asks,
    orderId: 1_000_001ul,
    invalidIdOk: false
);

//Build Transaction
RequestResult<ResponseValue<BlockHash>> blockhashRequest = await rpcClient.GetRecentBlockHashAsync();
TransactionBuilder txBuilder = new TransactionBuilder().SetFeePayer(solanaWalletPublicKey).SetRecentBlockHash(blockhashRequest.Result.Value.Blockhash);

txBuilder.AddInstruction(cancelById);

Wallet solanaWallet = new SolanaKeyStoreService().RestoreKeystoreFromFile("__location_of_your_json_wallet_file__");
byte[] txBytes = txBuilder.Build(new List<Account>() { solanaWallet.Account });

//Send Transaction
RequestResult<string> transaction = await rpcClient.SendTransactionAsync(txBytes, true, Commitment.Processed);
```
{% endtab %}

{% tab title="Python" %}
This code will load the 'example' wallet, connect to the _devnet_ cluster, place an order and then cancel it.

This is a bit longer than the Place Order example but only because it performs most of the Place Order code as a setup to create the order so there's an order to cancel.

The key point is that `cancel_order()` takes an `Order` as a parameter, and that `Order` needs either the `id` or `client_id` property to be set so the Mango program can find the equivalent order data on-chain. The sample code specified 1001 as the `client_id` when the order was instantiated so it's fine for use here.

```python
import decimal
import mango

from solana.publickey import PublicKey

# Use our hard-coded devnet wallet for DeekipCw5jz7UgQbtUbHQckTYGKXWaPQV4xY93DaiM6h.
# For real-world use you'd load the bytes from the environment or a file. Later we use
# its Mango Account at HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL.
wallet = mango.Wallet(bytes([67,218,68,118,140,171,228,222,8,29,48,61,255,114,49,226,239,89,151,110,29,136,149,118,97,189,163,8,23,88,246,35,187,241,107,226,47,155,40,162,3,222,98,203,176,230,34,49,45,8,253,77,136,241,34,4,80,227,234,174,103,11,124,146]))

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    group = mango.Group.load(context)
    account = mango.Account.load(context, PublicKey("HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL"), group)
    market_operations = mango.operations(context, wallet, account, "SOL-PERP", dry_run=False)

    print("Orders (initial):")
    print(market_operations.load_orderbook())

    order = mango.Order.from_values(side=mango.Side.BUY,
                                    price=decimal.Decimal(10),
                                    quantity=decimal.Decimal(1),
                                    order_type=mango.OrderType.POST_ONLY)
    print("Placing order:", order)
    placed_order_signatures = market_operations.place_order(order)

    print("Waiting for place order transaction to confirm...\n", placed_order_signatures)
    mango.WebSocketTransactionMonitor.wait_for_all(
            context.client.cluster_ws_url, placed_order_signatures, commitment="processed"
        )

    print("\n\nOrders (including our new order):")
    orderbook = market_operations.load_orderbook()
    print(orderbook)

    # Order has the client ID 1001 so we can use that Order object as a parameter here.
    cancellaton_signatures = market_operations.cancel_order(order)

    print("Waiting for cancel order transaction to confirm...\n", cancellaton_signatures)
    mango.WebSocketTransactionMonitor.wait_for_all(
            context.client.cluster_ws_url, cancellaton_signatures, commitment="processed"
        )

    print("\n\nOrders (without our order):")
    print(market_operations.load_orderbook())
print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Cancel All Orders

You can cancel all of your open orders by adding the CancelAllPerpOrders instruction to a transaction.\
\
You need to manually cancel all expired orders.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import { Config, GroupConfig, MangoClient } from '../src';
import { Account, Commitment, Connection, PublicKey } from '@solana/web3.js';
import configFile from '../src/ids.json';
import * as os from 'os';
import * as fs from 'fs';

async function cancelOrders() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  const payer = new Account(
    JSON.parse(
      fs.readFileSync(
        os.homedir() + '/.config/solana/my-mainnet.json',
        'utf-8',
      ),
    ),
  );

  // load group, account, market
  const mangoAccountPk = new PublicKey('YOUR_MANGOACCOUNT_KEY');
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const mangoAccount = await client.getMangoAccount(
    mangoAccountPk,
    mangoGroup.dexProgramId,
  );

  const perpMarkets = await Promise.all(
    groupConfig.perpMarkets.map((perpMarket) => {
      return mangoGroup.loadPerpMarket(
        connection,
        perpMarket.marketIndex,
        perpMarket.baseDecimals,
        perpMarket.quoteDecimals,
      );
    }),
  );

  // cancel all perp orders on all markets
  await client.cancelAllPerpOrders(
    mangoGroup,
    perpMarkets,
    mangoAccount,
    payer,
  );
}

cancelOrders();
```
{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Solana Wallet
PublicKey solanaWalletPublicKey = new PublicKey("__your_solana_wallet_public_key__");

//Mango Account
PublicKey mangoAccountPublicKey = new PublicKey("__your_mango_account_public_key__");
AccountResultWrapper<MangoAccount> mangoAccount = await mangoClient.GetMangoAccountAsync(mangoAccountPublicKey, Commitment.Processed);

//Market
PublicKey perpMarketPublicKey = new PublicKey("4GkJj2znAr2pE2PBbak66E12zjCs2jkmeafiJwDVM9Au"); // SRM-PERP
AccountResultWrapper<MangoGroup> mangoGroup = await mangoClient.GetMangoGroupAsync(Constants.MangoGroup);
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync(perpMarketPublicKey);

//Market Index
IPythClient pythClient = Solnet.Pyth.ClientFactory.GetClient(rpcClient, streamingRpcClient);
AccountResultWrapper<MappingAccount> mappingAccount = await pythClient.GetMappingAccountAsync(Solnet.Pyth.Constants.MappingAccount);
MultipleAccountsResultWrapper<List<ProductAccount>> productAccounts = await pythClient.GetProductAccountsAsync(mappingAccount.ParsedResult);
ProductAccount productAccount = productAccounts.ParsedResult.First(x => x.Product.Symbol.Contains("SRM"));
PublicKey oracle = mangoGroup.ParsedResult.Oracles.First(x => x.Key == productAccount.PriceAccount.Key);
int perpetualMarketIndex = mangoGroup.ParsedResult.Oracles.IndexOf(oracle);

MangoProgram mangoProgram = MangoProgram.CreateMainNet();

TransactionInstruction cancelAll = mangoProgram.CancelAllPerpOrders(
    mangoGroup: Constants.MangoGroup,
    mangoAccount: mangoAccountPublicKey,
    owner: solanaWalletPublicKey,
    perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
    bids: perpMarket.ParsedResult.Bids,
    asks: perpMarket.ParsedResult.Asks,
    limit: 20 //Maximum number of orders to cancel
);

//Build Transaction
RequestResult<ResponseValue<BlockHash>> blockhashRequest = await rpcClient.GetRecentBlockHashAsync();
TransactionBuilder txBuilder = new TransactionBuilder().SetFeePayer(solanaWalletPublicKey).SetRecentBlockHash(blockhashRequest.Result.Value.Blockhash);

txBuilder.AddInstruction(cancelAll);

Wallet solanaWallet = new SolanaKeyStoreService().RestoreKeystoreFromFile("__location_of_your_json_wallet_file__");
byte[] txBytes = txBuilder.Build(new List<Account>() { solanaWallet.Account });

//Send Transaction
RequestResult<string> transaction = await rpcClient.SendTransactionAsync(txBytes, true, Commitment.Processed);
```
{% endtab %}

{% tab title="Python" %}
Mango's Perp markets allow cancelling all open perp orders in a single instruction - no need to look up `id`s or `client_id`s. This can be useful in fast-moving situations, for example if an order might be filled between you fetching order data and sending the cancellation.

It's particularly important when using Time In Force as these must be explicitly cancelled when they have expired.

This code will load the "SOL-PERP" `MarketInstructionsBuilder` to build the `CancelAllPerpOrders` instruction, and then `execute()` it.

```python
import mango

from solana.publickey import PublicKey

# Use our hard-coded devnet wallet for DeekipCw5jz7UgQbtUbHQckTYGKXWaPQV4xY93DaiM6h.
# For real-world use you'd load the bytes from the environment or a file. Later we use
# its Mango Account at HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL.
wallet = mango.Wallet(bytes([67,218,68,118,140,171,228,222,8,29,48,61,255,114,49,226,239,89,151,110,29,136,149,118,97,189,163,8,23,88,246,35,187,241,107,226,47,155,40,162,3,222,98,203,176,230,34,49,45,8,253,77,136,241,34,4,80,227,234,174,103,11,124,146]))

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    group = mango.Group.load(context)
    account = mango.Account.load(context, PublicKey("HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL"), group)
    market_instructions: mango.PerpMarketInstructionBuilder = mango.instruction_builder(context, wallet, account, "SOL-PERP", dry_run=False)

    max_to_cancel = 20 # Maximum number of orders to cancel
    cancel_all = market_instructions.build_cancel_all_orders_instructions(limit=max_to_cancel)

    signers = mango.CombinableInstructions.from_wallet(wallet)
    cancel_signatures = (signers + cancel_all).execute(context)

    print("Waiting for CancelAll transaction to confirm...\n", cancel_signatures)
    mango.WebSocketTransactionMonitor.wait_for_all(
            context.client.cluster_ws_url, cancel_signatures, commitment="processed"
        )

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Refresh Order / Atomic Transaction for Market Makers

Currently, most market makers are sending cancel all followed by a bid and an ask in the same transaction.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import {
  Config,
  getMarketByBaseSymbolAndKind,
  GroupConfig,
  I64_MAX_BN,
  makeCancelAllPerpOrdersInstruction,
  makePlacePerpOrder2Instruction,
  MangoClient,
} from '../src';
import {
  Account,
  Commitment,
  Connection,
  PublicKey,
  Transaction,
} from '@solana/web3.js';
import configFile from '../src/ids.json';
import * as os from 'os';
import * as fs from 'fs';
import { BN } from 'bn.js';

async function atomicCancelReplace() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  const payer = new Account(
    JSON.parse(
      fs.readFileSync(
        os.homedir() + '/.config/solana/my-mainnet.json',
        'utf-8',
      ),
    ),
  );

  // load group, account, market
  const mangoAccountPk = new PublicKey('YOUR_MANGOACCOUNT_KEY');
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const mangoAccount = await client.getMangoAccount(
    mangoAccountPk,
    mangoGroup.dexProgramId,
  );

  const perpMarketConfig = getMarketByBaseSymbolAndKind(
    groupConfig,
    'BTC',
    'perp',
  );

  const perpMarket = await mangoGroup.loadPerpMarket(
    connection,
    perpMarketConfig.marketIndex,
    perpMarketConfig.baseDecimals,
    perpMarketConfig.quoteDecimals,
  );

  const tx = new Transaction();

  // add cancel all instruction
  tx.add(
    makeCancelAllPerpOrdersInstruction(
      groupConfig.mangoProgramId,
      mangoGroup.publicKey,
      mangoAccount.publicKey,
      payer.publicKey,
      perpMarket.publicKey,
      perpMarket.bids,
      perpMarket.asks,
      new BN(20),
    ),
  );

  // add place bid instruction
  // use best price
  const bids = await perpMarket.loadBids(connection);
  const bidPrice = bids.getBest().price;

  const [nativeBidPrice, nativeBidQuantity] =
    perpMarket.uiToNativePriceQuantity(bidPrice, 0.1);

  tx.add(
    makePlacePerpOrder2Instruction(
      groupConfig.mangoProgramId,
      mangoGroup.publicKey,
      mangoAccount.publicKey,
      payer.publicKey,
      mangoGroup.mangoCache,
      perpMarket.publicKey,
      perpMarket.bids,
      perpMarket.asks,
      perpMarket.eventQueue,
      mangoAccount.getOpenOrdersKeysInBasketPacked(),
      nativeBidPrice,
      nativeBidQuantity,
      I64_MAX_BN,
      new BN(0),
      'buy',
      new BN(20),
      'postOnly',
      false,
    ),
  );

  // add place ask instruction
  // use best price
  const asks = await perpMarket.loadAsks(connection);
  const askPrice = asks.getBest().price;

  const [nativeAskPrice, nativeAskQuantity] =
    perpMarket.uiToNativePriceQuantity(askPrice, 0.1);

  tx.add(
    makePlacePerpOrder2Instruction(
      groupConfig.mangoProgramId,
      mangoGroup.publicKey,
      mangoAccount.publicKey,
      payer.publicKey,
      mangoGroup.mangoCache,
      perpMarket.publicKey,
      perpMarket.bids,
      perpMarket.asks,
      perpMarket.eventQueue,
      mangoAccount.getOpenOrdersKeysInBasketPacked(),
      nativeAskPrice,
      nativeAskQuantity,
      I64_MAX_BN,
      new BN(0),
      'sell',
      new BN(20),
      'postOnly',
      false,
    ),
  );

  await client.sendTransaction(tx, payer, []);
}

atomicCancelReplace();
```
{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

//Token
PublicKey perpMarketPublicKey = new PublicKey("4GkJj2znAr2pE2PBbak66E12zjCs2jkmeafiJwDVM9Au"); // SRM-PERP
AccountResultWrapper<MangoGroup> mangoGroup = await mangoClient.GetMangoGroupAsync(Constants.MangoGroup);
IPythClient pythClient = Solnet.Pyth.ClientFactory.GetClient(rpcClient, streamingRpcClient);
AccountResultWrapper<MappingAccount> mappingAccount = await pythClient.GetMappingAccountAsync(Solnet.Pyth.Constants.MappingAccount);
MultipleAccountsResultWrapper<List<ProductAccount>> productAccounts = await pythClient.GetProductAccountsAsync(mappingAccount.ParsedResult);
ProductAccount productAccount = productAccounts.ParsedResult.First(x => x.Product.Symbol.Contains("SRM"));
PublicKey oracle = mangoGroup.ParsedResult.Oracles.First(x => x.Key == productAccount.PriceAccount.Key);
int perpetualMarketIndex = mangoGroup.ParsedResult.Oracles.IndexOf(oracle);
int marketIndex = mangoGroup.ParsedResult.GetPerpMarketIndex(perpMarketPublicKey);
TokenInfo token = mangoGroup.ParsedResult.Tokens[marketIndex];
TokenInfo quoteToken = mangoGroup.ParsedResult.GetQuoteTokenInfo();

//Market and Pricing
AccountResultWrapper<PerpMarket> perpMarket = await mangoClient.GetPerpMarketAsync(perpMarketPublicKey);
(long bidPrice, long bidQuantity) = perpMarket.ParsedResult.UiToNativePriceQuantity(1.5, 14156.28539071348, token.Decimals, quoteToken.Decimals);
(long askPrice, long askQuantity) = perpMarket.ParsedResult.UiToNativePriceQuantity(2.1, 14156.28539071348, token.Decimals, quoteToken.Decimals);

//Solana Wallet
PublicKey solanaWalletPublicKey = new PublicKey("__your_solana_wallet_public_key__");

//Mango Account
PublicKey mangoAccountPublicKey = new PublicKey("__your_mango_account_public_key__");
AccountResultWrapper<MangoAccount> mangoAccount = await mangoClient.GetMangoAccountAsync(mangoAccountPublicKey, Commitment.Processed);

//Expiry
ulong expiryTimestamp = (ulong)new DateTimeOffset(DateTime.UtcNow.AddSeconds(250)).ToUnixTimeSeconds();

MangoProgram mangoProgram = MangoProgram.CreateMainNet();

TransactionInstruction cancelAll = mangoProgram.CancelAllPerpOrders(
    mangoGroup: Constants.MangoGroup,
    mangoAccount: mangoAccountPublicKey,
    owner: solanaWalletPublicKey,
    perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
    bids: perpMarket.ParsedResult.Bids,
    asks: perpMarket.ParsedResult.Asks,
    limit: 2
);

TransactionInstruction bid = mangoProgram.PlacePerpOrder2(
    mangoGroup: Constants.MangoGroup,
    mangoAccount: mangoAccountPublicKey,
    owner: solanaWalletPublicKey,
    mangoCache: Constants.MangoCache,
    perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
    bids: perpMarket.ParsedResult.Bids,
    asks: perpMarket.ParsedResult.Asks,
    eventQueue: perpMarket.ParsedResult.EventQueue,
    openOrdersAccounts: mangoAccount.ParsedResult.SpotOpenOrders,
    side: Side.Buy,
    orderType: PerpOrderType.PostOnly,
    price: bidPrice, 
    maxBaseQuantity: bidQuantity,
    clientOrderId: 1_000_001ul,
    expiryTimestamp: expiryTimestamp,
    reduceOnly: false
);

TransactionInstruction ask = mangoProgram.PlacePerpOrder2(
	mangoGroup: Constants.MangoGroup,
	mangoAccount: mangoAccountPublicKey,
	owner: solanaWalletPublicKey,
	mangoCache: Constants.MangoCache,
	perpetualMarket: mangoGroup.ParsedResult.PerpetualMarkets[perpetualMarketIndex].Market,
	bids: perpMarket.ParsedResult.Bids,
	asks: perpMarket.ParsedResult.Asks,
	eventQueue: perpMarket.ParsedResult.EventQueue,
	openOrdersAccounts: mangoAccount.ParsedResult.SpotOpenOrders,
	side: Side.Sell,
	orderType: PerpOrderType.PostOnly,
	price: askPrice,
	maxBaseQuantity: askQuantity,
	clientOrderId: 1_000_002ul,
	expiryTimestamp: expiryTimestamp,
	reduceOnly: false
);

//Build Transaction
RequestResult<ResponseValue<BlockHash>> blockhashRequest = await rpcClient.GetRecentBlockHashAsync();
TransactionBuilder txBuilder = new TransactionBuilder().SetFeePayer(solanaWalletPublicKey).SetRecentBlockHash(blockhashRequest.Result.Value.Blockhash);

txBuilder.AddInstruction(cancelAll);
txBuilder.AddInstruction(bid);
txBuilder.AddInstruction(ask);

Wallet solanaWallet = new SolanaKeyStoreService().RestoreKeystoreFromFile("__location_of_your_json_wallet_file__");
byte[] txBytes = txBuilder.Build(new List<Account>() { solanaWallet.Account });

//Send Transaction
RequestResult<string> transaction = await mangoClient.RpcClient.SendTransactionAsync(txBytes, false, Commitment.Processed);
```
{% endtab %}

{% tab title="Python" %}
Solana's transaction mechanism allows for atomic cancel-and-replace of orders - either the entire transaction succeeds (old orders are cancelled, new orders are placed), or the entire transaction fails (no orders are cancelled, no orders are placed).

Neither Serum nor Mango supports 'editing' or changing an order - to change the price or quantity for an order you must cancel it and replace it with an order with updated values.

This code will loop 3 times around:

* in one transaction: cancelling all perp orders and placing bid and ask perp orders on SOL-PERP
* wait for that transaction to confirm
* pause for 5 seconds

You can verify the transaction signatures in [Solana Explorer](https://explorer.solana.com/?cluster=devnet) to see there is a single transaction containing a `CancelAllPerpOrders` instruction followed by two `PlacePerpOrder2` instructions. Since they're all in the same transaction, they will all succeed or all fail - if any instruction fails, the previous instructions are not committed to the chain, as if they never happened.

```python
import decimal
import mango
import time

from solana.publickey import PublicKey

# Use our hard-coded devnet wallet for DeekipCw5jz7UgQbtUbHQckTYGKXWaPQV4xY93DaiM6h.
# For real-world use you'd load the bytes from the environment or a file. Later we use
# its Mango Account at HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL.
wallet = mango.Wallet(bytes([67,218,68,118,140,171,228,222,8,29,48,61,255,114,49,226,239,89,151,110,29,136,149,118,97,189,163,8,23,88,246,35,187,241,107,226,47,155,40,162,3,222,98,203,176,230,34,49,45,8,253,77,136,241,34,4,80,227,234,174,103,11,124,146]))

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    group = mango.Group.load(context)
    account = mango.Account.load(context, PublicKey("HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL"), group)
    market_operations = mango.operations(context, wallet, account, "SOL-PERP", dry_run=False)
    market_instructions: mango.PerpMarketInstructionBuilder = mango.instruction_builder(context, wallet, account, "SOL-PERP", dry_run=False)

    signers: mango.CombinableInstructions = mango.CombinableInstructions.from_wallet(wallet)

    for counter in range(3):
        print("\n\nOrders:")
        orderbook = market_operations.load_orderbook()
        print(orderbook)

        instructions = signers
        instructions += market_instructions.build_cancel_all_orders_instructions()
        buy = mango.Order.from_values(side=mango.Side.BUY,
                                      price=orderbook.top_bid.price,
                                      quantity=decimal.Decimal(1),
                                      order_type=mango.OrderType.POST_ONLY,
                                      client_id=counter+10)
        print(buy)
        instructions += market_instructions.build_place_order_instructions(buy)
        sell = mango.Order.from_values(side=mango.Side.SELL,
                                       price=orderbook.top_ask.price,
                                       quantity=decimal.Decimal(1),
                                       order_type=mango.OrderType.POST_ONLY,
                                       client_id=counter+20)
        print(sell)
        instructions += market_instructions.build_place_order_instructions(sell)

        signatures = instructions.execute(context)

        print("Waiting for refresh order transaction to confirm...\n", signatures)
        mango.WebSocketTransactionMonitor.wait_for_all(
                context.client.cluster_ws_url, signatures, commitment="processed"
            )

        print("Sleeping for 5 seconds...")
        time.sleep(5)

print("Cleaning up...")
instructions = signers
instructions += market_instructions.build_cancel_all_orders_instructions()

cleanup_signatures = instructions.execute(context)

print("Waiting for cleanup transaction to confirm...\n", cleanup_signatures)
mango.WebSocketTransactionMonitor.wait_for_all(
        context.client.cluster_ws_url, cleanup_signatures, commitment="processed"
    )

print("Example complete.")
```
{% endtab %}
{% endtabs %}

### Get Account Balance

You can query your Mango account assets and available equity.

{% tabs %}
{% tab title="TypeScript" %}
```typescript
import { Config, GroupConfig, MangoClient } from '../src';
import { Commitment, Connection, PublicKey } from '@solana/web3.js';
import configFile from '../src/ids.json';

async function getAccountBalance() {
  // setup client
  const config = new Config(configFile);
  const groupConfig = config.getGroupWithName('mainnet.1') as GroupConfig;
  const connection = new Connection(
    config.cluster_urls[groupConfig.cluster],
    'processed' as Commitment,
  );
  const client = new MangoClient(connection, groupConfig.mangoProgramId);

  // load group, cache, account
  const mangoAccountPk = new PublicKey('YOUR_MANGOACCOUNT_KEY');
  const mangoGroup = await client.getMangoGroup(groupConfig.publicKey);
  const cache = await mangoGroup.loadCache(connection);
  const mangoAccount = await client.getMangoAccount(
    mangoAccountPk,
    mangoGroup.dexProgramId,
  );

  // print account balances
  console.log(mangoAccount.toPrettyString(groupConfig, mangoGroup, cache));
}

getAccountBalance();
```


{% endtab %}

{% tab title="C#" %}
```csharp
//Mango Client
IRpcClient rpcClient = Solnet.Rpc.ClientFactory.GetClient(Cluster.MainNet);
IStreamingRpcClient streamingRpcClient = ClientFactory.GetStreamingClient(Cluster.MainNet);
IMangoClient mangoClient = Solnet.Mango.ClientFactory.GetClient(rpcClient, streamingRpcClient);

PublicKey mangoAccountPublicKey = new PublicKey("__your_mango_account_public_key__");

//Mango Accounts
AccountResultWrapper<MangoGroup> mangoGroup = await mangoClient.GetMangoGroupAsync(Constants.MangoGroup);
AccountResultWrapper<MangoAccount> mangoAccounts = await mangoClient.GetMangoAccountAsync(mangoAccountPublicKey, Commitment.Processed);
AccountResultWrapper<MangoCache> mangoCache = await mangoClient.GetMangoCacheAsync(Constants.MangoCache);

I80F48 liabilities = mangoAccounts.ParsedResult.GetLiabilitiesValue(mangoGroup.ParsedResult, mangoCache.ParsedResult);
I80F48 assets = mangoAccounts.ParsedResult.GetAssetsValue(mangoGroup.ParsedResult, mangoCache.ParsedResult);
I80F48 equity = assets - liabilities;

Console.WriteLine($"Liabilities: {liabilities}, Assets: {assets}, Available Equity: {equity}");
```
{% endtab %}

{% tab title="Python" %}
This code will connect to the _devnet_ cluster and show important data from a Mango Account.

Data on deposits, borrows and perp positions are all available in a `pandas` `DataFrame` for you to perform your own calculations upon. The `Account` class also has some methods to take this `DataFrame` and run common calculations on it, such as calculating the total value of the `Account` (using `total_value()`), the health of the account (using `init_health()` and `maint_health()`), or the account's leverage (using `leverage()`).

```python
import mango

from solana.publickey import PublicKey

with mango.ContextBuilder.build(cluster_name="devnet") as context:
    group = mango.Group.load(context)
    cache: mango.Cache = mango.Cache.load(context, group.cache)

    account = mango.Account.load(context, PublicKey("HhepjyhSzvVP7kivdgJH9bj32tZFncqKUwWidS1ja4xL"), group)
    open_orders = account.load_all_spot_open_orders(context)
    frame = account.to_dataframe(group, open_orders, cache)
    print(frame["Spot"])
    print(frame["SpotValue"])

print("Example complete.")
```
{% endtab %}
{% endtabs %}
