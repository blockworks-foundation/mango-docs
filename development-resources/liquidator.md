---
description: >-
  Liquidation, thresholds, the liquidation process, liquidators, and running the
  reference liquidator
---

# Liquidation

## Liquidation

**Note:** this guide applies only to liquidation in Mango Markets V2, not Mango Markets V3.

Liquidation is how Mango Markets protects against socialised losses from overextended accounts.

![](<../.gitbook/assets/4whyhx (1).jpg>)

Mango Markets margin accounts must hold more assets than liabilities. If margin accounts were allowed to hold more liabilities than assets, those excess liabilities would need to be paid for by all Mango Markets users – losses would be spread out - "socialised" - across all users.

To prevent this, Mango Markets requires users to hold significantly more assets than liabilities in their accounts. There are two thresholds:

* The "_Initial Collateral Ratio"_ (currently 1.2), and
* The "_Maintenance Collateral Ratio"_ (currently 1.1)

Accounts must exceed the Initial Collateral Ratio before they are allowed to trade. If an account has less than the Initial Collateral Ratio, then it cannot try to borrow more. Existing positions remain open, it’s just not possible to open new positions.

Open positions are given some "breathing space." If the market moves against your position, decreasing the value of your assets and/or increasing the value of your liabilities, the collateral ratio can safely dip just below the Initial Collateral Ratio.

However, if your asset values fall or your liability values grow to the extent that your collateral ratio falls below the Maintenance Collateral Ratio, then your account becomes open to "liquidation."

With the current Initial Collateral Ratio of 1.2 and Maintenance Collateral Ratio of 1.1:

* If a margin account doesn’t hold assets worth more than 120% of its liabilities, it cannot open new positions, and
* If a margin account doesn't hold assets worth more than 110% of its liabilities, it may be liquidated.

If an account is open to liquidation, anyone can "pay off" some of the liabilities in that account. In return, that payer receives 105% of the value of their tokens back, taken from the collateral of the liquidated account.

This liquidation process continues until the remaining assets are once again worth more than the Initial Collateral Ratio of the remaining liabilities, or until there are no more collateral tokens to return to liquidators.

## Liquidator

A "liquidator" is just a program that initiates the liquidation process. It is permissionless and decentralised – liquidators can be run by anyone, anywhere there is an internet connection.

A liquidator checks for liquidatable accounts, prepares those liquidatable accounts for liquidation, then provides funds to the liquidated account liabilities and in return receives funds from the liquidated account’s collateral.

Code for a [reference liquidator is available](https://github.com/blockworks-foundation/mango-explorer/blob/v2/Quickstart.md).

### Liquidation Process

The basic liquidation process is as follows:

1. The liquidator prepares the account to be liquidated by requesting a "force cancel" of all existing orders. This is necessary because orders can lock funds preventing them being sent to the liquidator. This is done by sending a "_ForceCancel"_ instruction to the Mango Markets programme.
2. The liquidator sends funds to cover some or all of the liabilities of the account to be liquidated. This is done by sending a "_PartialLiquidate"_ instruction to the Mango Markets programme.
3. Repeat step 2, if the account is still liquidatable and the liquidator has sufficient tokens.

In reality, a liquidator probably does the following:

1. Pick accounts that are liquidatable, or close to liquidatable.
2. Watch those accounts closely as prices change.
3. When it finds liquidatable accounts:
   1. Pick the one with the most collateral. (If there are multiple liquidatable accounts, and other liquidators are active, it makes sense to start with the accounts that will bring the most profit to the liquidator).
   2. Pick the market with the most value in that account’s openorders accounts.
   3. Build the "Force Cancel" instruction for that account’s outstanding orders in that market.
   4. Build the "Partial Liquidate" instruction for that account.
   5. Send the built instruction(s).
   6. Convert the received tokens so the liquidator wallet is ready to liquidate again.
   7. Repeat from step 2 until the account is above the Initial Collateral Ratio once more, or has fallen below the "worthwhile" threshold for the liquidator.
   8. Repeat from step 1 until there are no more worthwhile liquidations to perform.
4. Repeat from step 1.

### Running The Liquidator

The [reference liquidator](https://github.com/blockworks-foundation/mango-explorer/blob/v2/Quickstart.md) is written in Python, and the command to run the liquidator is simply:

`liquidator`

However, the mango-explorer code is usually deployed via a docker container so the command line becomes more complicated. For example:

`docker run --rm -it –name=mango-explorer /var/mango-explorer/id.json:/home/jovyan/work/id.json opinionatedgeek/mango-explorer:latest liquidator`

Further configuration of the liquidator is done via command-line parameters. For example, configuring the wallet to rebalance after significant liquidations is done by specifying "target" balances:

`docker run --rm -it –name=mango-explorer /var/mango-explorer/id.json:/home/jovyan/work/id.json opinionatedgeek/mango-explorer:latest liquidator --target "BTC:0.05" --target "ETH:1"`

### Creating A Shell Alias

Given the length and complexity of the docker command, it can be useful to create a shell "alias" so that doesn’t have to be typed each time.

Here’s an example command to create such an alias:

`alias mango-explorer="docker run --rm -it --name=mango-explorer -v /var/mango-explorer/id.json:/home/jovyan/work/id.json opinionatedgeek/mango-explorer:latest"`

That will create an alias called "mango-explorer," meaning the above docker command with targets could then be run using:

`mango-explorer liquidator --target "BTC:0.05" --target "ETH:1"`

That seems much more readable and less likely to have typos.

### Dry Runs

The commands that perform transactions generally take a `--dry-run` parameter. This should help show you what actions the command will take, without actually performing those tasks.

This can be useful for rebalancing a wallet, allowing you to see what buys and sells are required to achieve your target balance without actually performing those buys and sells.

### Balancing The Wallet

Liquidations will change the balances of tokens in your wallet. That’s to be expected.

But what should happen if you accumulate too much of one token? Or not enough of another?

And what happens if you perform a big liquidation that empties your wallet of one token?

A common step then is to "re-balance" the tokens in your wallet, buying or selling different token types until your wallet is back to a "balanced" state, ready to liquidate more accounts.

Picking appropriate targets for balancing is a tricky topic and beyond this discussion. However, the liquidator does allow for a few simple strategies.

You can target a fixed or percentage balance for each token in the group, allowing you to rebalance to a specific ratio of tokens (like thirds) or a specific fixed balance of some tokens, accumulating changes to a single token.

* A percentage target is specified by using the token name and the target with a "%" character after it, like:\
  \
  `--target "BTC:33%"` (Note that it’s best to put quote marks around targets since the "%" is a wildcard/substitution character in some shells).\
  \
  That says that rebalances should target the value of BTC in the wallet to be 33% of the total wallet value. This can cause unexpected decreases in other balances if the value of BTC rises, requiring additional funds from other tokens to maintain the 33% target.
* A fixed target is specified by using the token name and the target, like:\
  \
  `--target "BTC:0.1"`\
  \
  That says that rebalances should aim to keep 0.1BTC in the wallet, no more, no less. If a liquidation takes BTC, the next rebalance will try to buy BTC to make up the shortfall. If a liquidation returns BTC, the next rebalance will try to sell the excess BTC.

It’s probably not a good idea to rebalance on every liquidation though. It may be worth forgoing rebalancing after a 10 cent trade (which could lock funds), instead continuing trying to liquidate other accounts.

How much flexibility should there be before requiring a rebalance? Again, that aspect is beyond this discussion but you can choose how much tolerance you’re willing to accept by specifying the action threshold:

`--action-threshold 0.05`

The default action threshold is 0.01, which means only rebalance when the target balance is further away from the current balance than 1% of the wallet total balance.

So if you had:

* 35% ETH
* 35% BTC
* 30% USDT

And your targets were:

* 33% ETH
* 33% BTC
* 34% USDT

Then an action threshold of 0.05 would mean no attempt to re-balance, while an action threshold of 0.01 would trigger a re-balance once the liquidation was complete.

### Notifications

The liquidator supports sending notifications of successful, failed, or all liquidation attempts, using Telegram, Discord, Mailjet, or writing to a CSV file. Each notification target its own format for specifying it on the command line.

You can be notified of different events (all liquidation attempts, or just successful or failed ones), and you can specify multiple notification targets for each event. To add a notification target for an event, you use a command-line parameter. There are 4 different ones for the liquidator:

* `--notify-liquidations`
* `--notify-successful-liquidations`
* `--notify-failed-liquidations`
* `--notify-errors`

To specify multiple notifications for the same event, you just repeat the command line parameter. For example:

`--notify-successful-liquidations NOTIFICATION-TARGET-1 --notify-successful-liquidations NOTIFICATION-TARGET-2 --notify-successful-liquidations NOTIFICATION-TARGET-3`

#### Discord Notification Target

[Discord](https://discord.com) notifications are sent using Discord’s "webhook" URL. Once you get the webhook URL for the Discord channel, the format of the Discord notification target is:

1. The word "discord"
2. A colon ":"
3. The Discord webhook URL

For example:

`discord:https://discord.com/api/webhooks/012345678901234567/ABCDE_fghij-KLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMN`

#### Telegram Notification Target

[Telegram](https://telegram.org) notifications are a bit more complicated, since both a "bot ID" and "chat ID" are needed. The format for the telegram notification target is:

1. The word "telegram"
2. A colon ":"
3. The chat ID
4. An "@" symbol
5. The bot token

For example:

`telegram:012345678@9876543210:ABCDEFGHijklmnop-qrstuvwxyzABCDEFGH`

#### CSV File Notification Target

A CSV file can be useful for collecting the outcome of successful liquidations.

The format for CSV file target is:

1. The word "csvfile"
2. A colon ":"
3. The name of the CSV file (can be a relative or absolute path)

There are complications however.

If this is to be used via docker, the CSV file:

1. Must exist before the container is started,
2. Must be specified on the docker command line as a "volume,"
3. Must be writable by the user ID of the user in the docker container,
4. Must be specified as a filename in the context of the docker process, not the host.

For example, to use a file called /var/mango-explorer/report.csv:

1. Run touch /var/mango-explorer/report.csv to create the file if it doesn’t already exist.
2. Make the file writable by the liquidator process in the docker container with: `chown 1000:1000 /var/mango-explorer/report.csv`
3. Add the line: `-v /var/mango-explorer/report.csv:/home/jovyan/work/report.csv` to your mango-explorer alias or docker command line.

#### Mailjet Notification Target

[Mailjet](https://www.mailjet.com) allows you to send emails, but requires many more parameters and so is a lot more complicated to configure. You also need an API key and secret from Mailjet, and need to have Mailjet configured for the sender and receiver addresses.

The format for Mailjet target is:

1. The word "mailjet"
2. A colon ":"
3. Your Mailjet API key
4. A colon ":"
5. Your Mailjet API secret
6. A colon ":"
7. The "from" name you want to appear in your email
8. A colon ":"
9. The "from" address for the email being sent
10. A colon ":"
11. The "to" name you want to appear in your email
12. A colon ":"
13. The "to" address for the email being sent

It's important to properly escape the individual components. Colons are used as separators, and shells use spaces as separators, so the individual components should never have those characters. Components should instead be URL-encoded so, for example, spaces appear as "%20" and colons appear as "%3A."
