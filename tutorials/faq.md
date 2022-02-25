---
description: >-
  Don't see your question answered here? Join our Discord to ask us!
  https://discord.gg/pV5mybZYY8
---

# General FAQ

### **How can I earn interest?**

To earn interest, simply deposit into your margin account; all assets automatically earn interest. Annual rates are displayed in the margin account panel in green.&#x20;

### **What are the different versions of Mango?**&#x20;

V2( mango.markets ) is the newest version of Mango and supports SOL, SRM, BTC, ETH, and USDC trading, lending, and borrowing.

V1( usdt.mango.markets ) supports BTC, ETH, and USDT trading, borrowing, and lending. SRM deposits do not count as collateral or earn interest. &#x20;

### **What prevents someone from borrowing funds, withdrawing, and running away with it?**

Our system checks deposits, positions, and borrows accounts to ensure that an individual can not withdraw more than their equity, calculated by val(deposits) +val(open orders) - val(borrows).

### **Does Mango have an insurance fund on deposits?**

There is an insurance fund to cover socialized losses to lenders on v3.&#x20;

### **Is Mango’s code audited?**

Mango code has been informally reviewed by Serum and Solana devs as well as the German white hat hacker team “Alles!’.

### **What does IOC mean?**

&#x20;Immediate or cancel (IOC). If an order is placed and not filled immediately, it's cancelled.

### **What does POST mean?**

POST limit orders are added to the order-book and earn maker fees if filled. POST orders will always be the 'maker'.

### **How can I contribute MSRM?**

Connect wallet and click 'Fee Discounts'. There is an option to Deposit SRM.

### **How is equity calculated?**

equity = val(deposits) +val(open orders) - val(borrows)

### **How is interest calculated?**

The interest rate is a function of the utilization ratio: total borrowed by all users divided by total deposits of all users. The interest rate will increase slowly approaching a utilization ratio of 70% but will increase rapidly above that to encourage the market to deposit more funds.

![](../.gitbook/assets/work.png)

### Does it cost anything to open a Mango account?&#x20;

Yes, to cover costs of creating necessary accounts on Solana blockchain and storage. \
The costs is 0.032 SOL to create MangoAccount and also 0.025 SOL for each new spot market they trade on.\
