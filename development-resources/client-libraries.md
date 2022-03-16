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
{% endtabs %}

### Subscribe to an Order Book

To get the order book you need to retrieve the bids and asks independently. Each side of the order book has a max capacity of 512 orders.\
\
When the 512 capacity is reached, orders that are the furthest away from the top will be removed when a new order is added and the Out Event is fired when an order is removed.\
\
Orders can be set to expire using the Time-In-Force&#x20;

{% tabs %}
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
{% endtabs %}

### Get an Order Book snapshot

You can also retrieve a snapshot of the order book.

{% tabs %}
{% tab title="First Tab" %}
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
{% endtabs %}

### Subscribe to Market Fills

You can subscribe to a markets' event queue. When the event queue is updated, the entire queue is received.\
\
You can filter the event queue for fills and use the sequence number to filter new fills added to the queue.

{% tabs %}
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
{% endtabs %}

### Get a Market fills snapshot

You can also retrieve a snapshot of a markets' event queue.

{% tabs %}
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
{% endtabs %}

### Post Orders

Orders are placed by adding an instruction to a transaction. You can add multiple instructions to a single transaction.\
\
There is no additional fee per instruction, your account is only charged per transaction.\
\
You can have a max of 64 open orders across all perpetual markets.

{% tabs %}
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
{% endtabs %}

### Cancel an Order

You can cancel a specific order by OrderID, Side or ClientID.

{% tabs %}
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
{% endtabs %}

### Cancel All Orders

You can cancel all of your open orders by adding the CancelAllPerpOrders instruction to a transaction.\
\
You need to manually cancel all expired orders.

{% tabs %}
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
{% endtabs %}

### Refresh Order / Atomic Transaction for Market Makers

Currently, most market makers are sending cancel all followed by a bid and an ask in the same transaction.

{% tabs %}
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
{% endtabs %}

### Get Account Balance

You can query your Mango account assets and available equity.

{% tabs %}
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
{% endtabs %}
