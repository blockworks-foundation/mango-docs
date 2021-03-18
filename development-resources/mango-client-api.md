# Mango Client API

The source code for the Mango Client API is hosted on [github](https://github.com/blockworks-foundation/mango-client-ts).

To access the API you will first need to setup a [Connection](https://solana-labs.github.io/solana-web3.js/classes/connection.html) using the `@solana/web3` sdk. The easiest way for now is to connect to the \`mainnet-beta\` version of the Mango Program. You can easily access the relevant connection details exported as [a json constant ](https://github.com/blockworks-foundation/mango-client-ts/blob/main/src/ids.json)`IDS`.

## Mango Groups

A mango group is a basket of cross-margined tokens. We are launching with a single group, but we are planning to release more in the future.

```typescript
import { IDS, MangoGroup  } from '@blockworks-foundation/mango-client';
import { Connection, PublicKey } from `@solana/web.js`

const cluster = 'mainnet-beta';
const connection = new Connection(IDS.cluster_urls[cluster], 'singleGossip');

const group = 'BTC_ETH_USDT';
const mangoGroupPk = new PublicKey(IDS[cluster].mango_groups[group].mango_group_pk);
const srmVaultPk = new PublicKey(IDS[cluster].mango_groups[group].srm_vault_pk])
const client = new MangoClient();
const mangoGroup = await client.getMangoGroup(connection, mangoGroupPk, srmVaultPk);
```

### Oracles

Each group manages internally a set of oracles to have up to date price feeds for it's tokens. For the `BTC_ETH_USDT` group two oracles are used: one for the pair`BTC/USDT` and one for the pair`ETH/USDT`. The oracles are provided by the [solana-flux-aggregator](https://github.com/blockworks-foundation/solana-flux-aggregator) and updated based on the median price reported across multiple centralized exchanged. 

```typescript
// fetch current oracle prices
mangoGroup.getPrices(connection: Connection): Promise<number[]> 
```

In case you want to use these oracles for your own project, please reach out on discord. We are actively working with the solana foundation to get more oracle providers involved to improve their decentralization and can provide support for running your own oracle.

### Lending

Each group also manages it's own lending pool for each of the traded tokens. Users can deposit tokens into different mango groups, but each token can only be assigned to one group at a time. The cross-margin capabilities of mango are currently to individual groups, meaning that deposited funds are only counted as collateral inside a single group. Interest for deposits and borrows is calculated on group level as well.

```typescript
// look up token index by mint public key
getTokenIndex(token: PublicKey): number 

// fetch annual interest as a fraction of the value usually [0,1]
mangoGroup.getBorrowRate(tokenIndex: number): number 
mangoGroup.getDepositRate(tokenIndex: number): number
```



## Margin Accounts

Each user can have multiple accounts associated with a single group. They function as a balance sheet and keep track of the deposits, borrows and open positions on the serum dex order book. A margin account is fully cross-margined and can be liquidated once the collateral ratio drops below maintenance margin level. When interacting with the margin account you will often need to provide the serum dex program-id in order to access the open orders on it's order book.

```typescript
import { IDS, MangoGroup  } from '@blockworks-foundation/mango-client';
import { Connection, PublicKey } from `@solana/web.js`

const cluster = 'mainnet-beta';
const connection = new Connection(IDS.cluster_urls[cluster], 'singleGossip');

const serumProgramId = new PublicKey(clusterIds.dex_program_id);
const myMarginAccountPubKey = new PublicKey('5JpTGitZjwzmyaz8K4bFvhjMc4rvYhy157x2pPEtYYs5');
const client = new MangoClient();
const myMarginAccount = await client.getMarginAccount(connection, myPubKey, serumProgramId);
```

