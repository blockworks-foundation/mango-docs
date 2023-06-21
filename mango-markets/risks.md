# \* Risks

This is a terse, informal, incomplete list of risks that may apply to users of Mango Markets.

## General

### Legal risk

You are responsible for determining the legal consequences of your actions.

### Taxation risk

You are responsible for any and all tax consequences of your actions.

### Market risk

Market prices can change quickly and unexpectedly. Past trends don’t necessarily continue into the future.

### Unknown risk

There are likely risks that are not listed here.

## Solana

### Wallet loss risk

If you lose your wallet, you will be locked out of all funds on Mango.

There is absolutely no way to recover funds in this case, for technical reasons.

Malicious actors may use social engineering to convince users to leak private keys or sign transactions to steal tokens (including those deposited on Mango) that may appear innocuous in the simulation, or prevent the simulation from running at all.

### Solana infrastructure risk

#### Downtimes and Halts

Solana has temporarily halted several times in the past, making everything on-chain unusable until the blockchain has restarted. This could happen again and the downtime could last longer.

The downtime could be permanent, causing total loss of funds for everything on Solana.

#### Fluctuations in Quality of Service

The timeliness of reported data, speed of transaction confirmation and other Solana services could become significantly worse. There have been cases in the past where this happened temporarily.

### Oracle provider risk

Mango Markets relies on Switchboard and Pyth oracles for prices. These are their own entities, with their own Solana programs and update mechanisms.

Hacks, manipulation, bugs or other issues with oracle updating could cause the Mango Markets program to receive incorrect prices. This can have different consequences:

#### Denial of service from invalid oracles

If an oracle price is too stale, too unconfident or marked as invalid, the Mango Markets program will not use the price. As a consequence, all interactions with the affected token or market will become impossible until a valid oracle price is available again.

That means all user accounts that somehow depend on that price - because of deposits, borrows or positions - could be frozen in case of an invalid oracle.

Example: If the oracle for the SOL token price went stale, all users that had SOL deposits, borrows, open orders on a spot market involving SOL or positions in SOL-PERP would effectively be locked out of their accounts until the problem is resolved.

The DAO can make a proposal to switch to a different oracle, but the vote takes at least three days to complete.

#### Liquidation and denial of service from bad oracle prices

An oracle could report an incorrect price but look valid. The Mango Markets program would have no way of knowing that the reported value is incorrect and estimate the user’s account value incorrectly.

If a hack, manipulation, or problem on the oracle provider’s side produces incorrect oracle prices, this could cause liquidation or inability to trade for users.

A long-term manipulation of key oracle prices may allow an attacker to steal user funds by causing repeated liquidations.

### Token risk

Interacting with Solana tokens comes with various risks, many of which depend on what the token represents. They are not listed here.

#### Depegs

A common issue for bridged tokens (like wBTC (Portal), ETH (Portal), …) and stablecoin tokens (like USDC, USDT, DAI, …) is that they could depeg from the underlying.

If the value of wBTC (Portal) no longer corresponds to the value of BTC or the value of USDC no longer corresponds to the value of USD, user accounts that relied on this correspondence may be liquidated.

Additionally, bridged tokens are often configured with an oracle for the underlying in Mango Markets. That means a depeg would likely cause the Security Council to halt trading on the bridged asset until the oracle has been switched out.

## Mango Markets: General

### Mango program risk

The Solana program that Mango Markets is based on could have bugs or be exploitable somehow. This could lead to everything from incorrect behavior to total loss of funds.

### Mango DAO risk

Mango Markets is operated by the Mango DAO. That means the Mango DAO is in full control of the Mango Markets program, configuration and vaults.

The DAO could vote to

* change Mango Markets parameters for specific tokens or markets
* change the program itself, to behave in any way it likes
* that includes changing the program to steal all user funds

Furthermore, bugs in the Governance Program, which the Mango DAO uses to operate, could allow an attacker to perform these actions without a vote.

### Security council risk

The Mango DAO has delegated a limited set of powers to a Security Council, to allow for quick responses to newly discovered security issues.

The Security Council can

* halt the program
* disable Mango features (like withdrawals, trading, etc)

While the Mango DAO can vote to undo these changes, this vote takes at least three days. Until then Mango Markets would be (potentially partially) unusable.

The council has in the past used its powers when security issues became known to it. It’s very likely to happen again in the future.

### Liquidation risk

Holding margin positions on Mango creates the risk of being liquidated when market conditions change.

#### Socialized loss

If positions can’t be liquidated in a timely manner, either due to aforementioned Solana risks, Mango program bugs, liquidator bugs, or insufficient market or liquidator liquidity, accounts may end up in negative equity. If the total negative equity in the system exceeds the insurance fund balance then losses will occur on accounts that were not at risk of liquidation.

#### Cascades

Large liquidations could cause market movements that could cause further liquidations.

### Services risk

Mango Markets relies on several services to keep itself running:

* Scrapers read on-chain data and prepare it for consumption by user interfaces. If these are down or buggy, the ui may show no, bad, or old data.
* Cranks regularly send permissionless transactions to process events or trigger on-chain bookkeeping. If they stop working, trading may be impossible or values won’t update until someone sends these again.

### User interface risk

While all user data is stored on the Solana blockchain, users will most likely interact with Mango Markets through a user interface that is hosted somewhere.

This user interface could become unavailable, have bugs, be hacked, or be malicious, causing a variety of problems.

## Mango Markets: Token deposits and borrows

### Token withdrawal liquidity risk

Deposited tokens are lent out to other users. Consequently, withdrawing deposits may be impossible if there are not enough tokens available in the Mango Markets program vaults.

In this case, depositors can only wait until more tokens are available again before withdrawing.

Mango uses two main mechanisms to make it likely for there to be tokens available for withdrawal:

* borrow and deposit interest increases sharply as the utilization (the fraction of borrowed and deposited tokens) increases
* a fraction of deposited tokens is kept as a reserve and not available for borrowing

### Variable interest rate risk

The token borrow and deposit interest rates depend on the token utilization and can change significantly very quickly.

## Mango Markets: Swaps

### Routing

Mango Markets relies on an external router (such as Jupiter) to find a good way of satisfying the user’s trade request. The router may be unavailable or may not provide the best route. If the router is hacked, it might propose malicious routes.

### Programs

Swaps can interact with many external programs, all of which may carry some risk.

## Mango Markets: OpenBook trading

### OpenBook program risk

Trading on OpenBook spot markets via Mango Markets means interacting with the OpenBook program. This program is controlled by the OpenBook program authority multisig.

The OpenBook program authority could change the program to steal funds in open orders accounts.

There could be bugs in the OpenBook program that could be exploited by attackers.

## Mango Markets: Perp markets

### Liquidity risk

Exiting a position requires that someone else is willing to take the opposite side. It is possible for the orderbook to be empty or very sparse, making it impossible to immediately exit a position without significant losses.

### Funding rate risk

The perp market funding rate depends on the state of the orderbook and the oracle. If there is a large gap between the two, it can be severe (currently up to 5% per day).

The funding rate can also change quickly and significantly.
