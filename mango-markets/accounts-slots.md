# 🫙 Account Slots

In order to fit the transaction requirements of the Solana blockchain, each `MangoAccount` need to have a maximum size. However, there are many more markets on Solana than a single `MangoAccount` could possibly fit. Mango solves this problem by providing "slots" for different user actions. This gives users the freedom to trade and borrow their preferred tokens, while still fulfilling the maximum size requirement. 

Slots can be modified via the Account page in Settings.

The current slots are:
* **Tokens** - How many tokens an account can deposit or borrow
* **Spot Markets** - How many spot markets the accopunt can interact with
* **Perp Markets** - How many perp markets the account can interact with
* **Perp Open Orders** - How many open orders the account can have over all perp markets 
* **Trigger Orders** - How many trigger orders the account can have

