---
description: >-
  Don't see your question answered here? Join our Discord to ask us!
  https://discord.gg/pV5mybZYY8
---

# General FAQ

### **How can I earn interest?**

To earn interest, simply deposit into your Mango Markets Account; all assets automatically earn interest. Annual rates are displayed in the Account page in green.

### **What are the different versions of Mango?**

V3 (trade.mango.markets) is the newest version of Mango Markets.

V2 (mango.markets) supports SOL, SRM, BTC, ETH, and USDC trading, lending, and borrowing.

V1 (usdt.mango.markets) supports BTC, ETH, and USDT trading, borrowing, and lending. SRM deposits do not count as collateral or earn interest.

### **What prevents someone from borrowing funds, withdrawing, and running away with it?**

Our system checks deposits, positions, and borrows accounts to ensure that an individual cannot withdraw more than their equity, calculated by val(deposits) +val(open orders) - val(borrows).

### **Does Mango Markets have an insurance fund on deposits?**

There is an insurance fund to cover socialised losses to lenders on v3.

### **Is Mango Markets' code audited?**

Mango Markets code has been informally reviewed by Serum and Solana devs as well as the German white hat hacker team “Alles!"

### **What does IOC mean?**

Immediate or cancel (IOC). If an order is placed and not filled within 10 seconds, it is cancelled.

### **What does POST mean?**

POST limit orders are added to the order-book and earn maker fees if filled. POST orders will always be the "maker."

### **How can I contribute MSRM?**

Connect wallet and click "Fee Discounts." There is an option to Deposit SRM.

### **How is equity calculated?**

equity = val(deposits) +val(open orders) - val(borrows)

### **How is interest calculated?**

The interest rate is a function of the utilisation ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching a utilisation ratio of 70% but will increase rapidly above that to encourage the market to deposit more funds.

![](../.gitbook/assets/work.png)

### Does it cost anything to open a Mango Markets Account?

Yes, to cover costs of creating necessary accounts on the Solana blockchain and storage.\
The costs is 0.032 SOL to create a Mango Markets Account and also 0.025 SOL for each new spot market they trade on.
