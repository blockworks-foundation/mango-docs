# 🖥 API Endpoints

Mango Markets does not have a traditional trading REST or WebSocket API.

The best way to connect programmatically is to use one of the [community-maintained libraries](../developer-resources/code-examples.md).

Listed below are some third-party, community-maintained public-facing services that we are aware of.&#x20;

If you have created something cool and would like us to add it to the list below please let us know by joining our [Discord](../../discord-and-community.md) server.

Transaction logs and PNL - [https://mango-transaction-log.herokuapp.com/redoc](https://mango-transaction-log.herokuapp.com/redoc)

* Account funding: [https://mango-transaction-log.herokuapp.com/v3/stats/total-funding?mango-account=](https://mango-transaction-log.herokuapp.com/v3/stats/total-funding?mango-account=\_\_mango\_account\_\_)\_\__your\_mango\_account\_\__
* Account PNL: [https://mango-transaction-log.herokuapp.com/v3/stats/settle\_perp?mango-account=](https://mango-transaction-log.herokuapp.com/v3/stats/settle\_perp?mango-account=)_\_\_your\_mango\_account\_\__
* Account rewards: [https://mango-transaction-log.herokuapp.com/v3/stats/redeem\_mngo\_rewards?mango-account=](https://mango-transaction-log.herokuapp.com/v3/stats/redeem\_mngo\_rewards?mango-account=)_\_\_your\_mango\_account\_\__

Candles - [https://event-history-api-candles.herokuapp.com/ ](https://event-history-api-candles.herokuapp.com/)

Event history - [https://event-history-api.herokuapp.com/](https://event-history-api.herokuapp.com/)

Stats - [https://mango-stats-v3.herokuapp.com/](https://mango-stats-v3.herokuapp.com/)

Mangolorians - [https://mangolorians.com/](https://mangolorians.com)

* Historical funding rates: [https://mangolorians.com/historical\_data/funding\_rates.csv?instrument=](https://mangolorians.com/historical\_data/funding\_rates.csv?instrument=SOL-PERP)_\_\_contract\_symbol\_name\_\__
* Historical trades: [https://mangolorians.com/historical\_data/trades.csv?instrument=](https://mangolorians.com/historical\_data/trades.csv?instrument=SOL-PERP)_\_\_contract\_symbol\_name\_\__
* Historical liquidations: [https://mangolorians.com/historical\_data/liquidations.csv?instrument=](https://mangolorians.com/historical\_data/liquidations.csv?instrument=SOL-PERP)_\_\_contract\_symbol\_name\_\__

For other specific data or endpoints, please contact us on [Discord](../../discord-and-community.md).
