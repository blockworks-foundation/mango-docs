---
description: >-
  Don't see your question answered here? Join our Discord to ask us!
  https://discord.gg/pV5mybZYY8
---

# FAQ

### **How can I earn interest?**

To earn interest, simply deposit into your margin account; all assets automatically earn interest. Annual rates are displayed in the margin account panel in green. 

### **What are the different versions of Mango?** 

V2\( mango.markets \) is the newest version of Mango and supports SOL, SRM, BTC, ETH, and USDC trading, lending, and borrowing.

V1\( usdt.mango.markets \) supports BTC, ETH, and USDT trading, borrowing, and lending. SRM deposits do not count as collateral or earn interest. 

### **Do SRM deposits earn interest?**

SRM deposits on V2 earn interest, while deposits in V1\(USDT\) do not because they are exempt from liquidation and not counted toward margin collateral.

Serum’s tier structure determines fee rates based on the amount of SRM held in an account; the more SRM in an account, the less you pay in fees and the more you earn as a maker. The highest tier is 1 MegaSerum\(MSRM\), worth 1 million SRM. Mango margin accounts are structured such that SRM deposits are combined with all users to collectively reach a higher tier and benefit together when trading on the platform. 

### **What prevents someone from borrowing funds, withdrawing, and running away with it?**

Our system checks deposits, positions, and borrows accounts to ensure that an individual can not withdraw more than their equity, calculated by val\(deposits\) +val\(open orders\) - val\(borrows\).

### **Does Mango have an insurance fund on deposits?**

Mango protocol charges 0 fees, which is nice, but as a result the protocol does not have an insurance fund. When a margin account has negative equity, the losses are socialized among the lenders. There is obviously the risk that a user loses his margin account to a liquidator when his account goes below the required maintenance collateral ratio \(MCR\) of 110%. But there is an even bigger risk to the ecosystem when a margin account falls below 100% collateral ratio. At this point, the value of the account's liabilities are larger than the value of assets. In this situation, the liquidator is provided a 1% incentive to liquidate and lenders collectively take the negative equity of the account. In rare circumstances, this could even trigger a liquidation cascade where the socialized losses trigger more accounts to fall below 100% collateral ratio.

We believe users should be aware of all the risks. That being said, we think socialized losses and especially liquidation cascades are extremely unlikely events. Given Solana's speed, a liquidator will most likely be able to liquidate an account as soon as it falls below MCR.

### **Is Mango’s code audited?**

Mango's code is unaudited. However, our contracts have been reviewed and we are working with the Solana team to get other experienced devs to critically read the smart contract before we open source it. It is possible bugs exist.

### **What does IOC mean?**

 Immediate or cancel \(IOC\). If an order is placed and not filled within 10 seconds, it's cancelled.

### **What does POST mean?**

POST limit orders are added to the order-book and earn maker fees if filled. POST orders will always be the 'maker'.

### **How can I contribute SRM?**

Connect wallet and click 'Fee Discounts'. There is an option to Deposit SRM.

### **What does the settle borrows button do?**

 The settle borrows button automatically closes your leverage positions \(add more, placeholder\).

### **Can we use auto approve transactions on** [**sollet.io**](http://sollet.io/)**?** 

Soon, we are working on implementing. Even if you select auto-approve, it will no work properly.

### **How is equity calculated?**

equity = val\(deposits\) +val\(open orders\) - val\(borrows\)

### **How is interest calculated?**

The interest rate is a function of the utilization ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching a utilization ratio of 70% but will increase rapidly above that to encourage the market to deposit more funds.

![](.gitbook/assets/work.png)

### Does it cost anything to open a Mango account? 

Yes, to cover costs of creating necessary accounts on Solana blockchain and storage.   
The costs is 0.032 SOL to create MangoAccount and also 0.025 SOL for each new spot market they trade on.  


## How to convert USDT to USDC?

The most efficient way to convert USDT to USDC is through Serum DEX:   
[https://dex.projectserum.com/\#/market/77quYg4MGneUdjgXCunt9GgM1usmrxKY31twEy3WHwcS](https://dex.projectserum.com/#/market/77quYg4MGneUdjgXCunt9GgM1usmrxKY31twEy3WHwcS)  
  
Alternatively, the tokens can be swapped on on Orca\([https://www.orca.so/](https://www.orca.so/)\) or StableSwap\([https://stableswap.pro/](https://stableswap.pro/)\). Please check the current exchange rates.   

![](.gitbook/assets/screen-shot-2021-06-07-at-6.34.09-pm%20%281%29%20%281%29.png)

