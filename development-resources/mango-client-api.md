# Mango Client API

[API Reference](https://blockworks-foundation.github.io/mango-client-v3/)

The source code for the v3 Mango Markets Client API is hosted on [github](https://github.com/blockworks-foundation/mango-client-v3).

To access the API you will first need to setup a [Connection](https://solana-labs.github.io/solana-web3.js/classes/Connection.html) using the `@solana/web3` sdk. The easiest way for now is to connect to the "mainnet" version of the Mango Markets Programme. You can easily access the relevant connection details exported as [a json constant ](https://github.com/blockworks-foundation/mango-client-v3/blob/main/src/ids.json)`IDS`.

## Mango Groups

A Mango Group is a basket of cross-margined tokens. Each version of Mango Markets uses a different Mango Group containing different tokens. The current v3 group is "mainnet.1."

```typescript
// get v3 Mango Group
import { IDS, MangoClient, Config, I80F48  } from '@blockworks-foundation/mango-client-v3';
import { Connection, PublicKey } from `@solana/web.js`

const cluster = 'mainnet';
const group = 'mainnet.1';

const config = new Config(IDS);
const groupConfig = config.getGroup(cluster, group);
if (!groupConfig) {
    throw new Error("unable to get mango group config");
  }
const mangoGroupKey = groupConfig.publicKey;

const clusterData = IDS.groups.find((g) => {
    return g.name == group && g.cluster == cluster;
  });
const mangoProgramIdPk = new PublicKey(clusterData.mangoProgramId);

const clusterUrl = IDS.cluster_urls[cluster];
const connection = new Connection(clusterUrl, 'singleGossip');
const client = new MangoClient(connection, mangoProgramIdPk);
const mangoGroup = await client.getMangoGroup(mangoGroupKey);
```

### Oracles

Each group uses a set of oracles that provides up-to-date price feeds for it's tokens. For the `mainnet.1` group Pyth and Switchboard oracles are used.

```typescript
// fetch current oracle price of a single token
mangoGroup.getPrice(tokenIndex: number, mangoCache: MangoCache): I80F48
// then use I80F48 .add/.sub/.mul/.div functions to perform calculations 
// or convert to human-friendly number with .toNumber() or .toFixed(2)
```

### Lending

Each group also manages it's own lending pool for each of the traded tokens. Users can deposit tokens into different mango groups, but each token can only be assigned to one group at a time. The cross-margin capabilities of mango are currently to individual groups, meaning that deposited funds are only counted as collateral inside a single group. Interest for deposits and borrows is calculated on group level as well.

```typescript
// look up token index by mint public key
mangoGroup.getTokenIndex(token: PublicKey): number 

// fetch total deposits/borrows, deposit and borrow interest rates, as well as percent utilization of each token in the group
const rootBanks = await mangoGroup.loadRootBanks(connection);
    const tokensInfo = groupConfig.tokens.map((token) => {
      const rootBank = rootBanks.find((bank) => {
        if (!bank) {
          return false;
        }
        return bank.publicKey.toBase58() == token.rootKey.toBase58();
      });

      if (!rootBank) {
        throw new Error("rootBanks is undefined");
      }
      return {
        name: token.symbol,
        totalDeposits: totalDeposits.toFixed(
           tokenPrecision[token.symbol] || 2
         ),
         totalBorrows: totalBorrows.toFixed(
           tokenPrecision[token.symbol] || 2
         ),
        depositRate: rootBank
          .getDepositRate(mangoGroup)
          .mul(I80F48.fromNumber(100)),
        borrowRate: rootBank
          .getBorrowRate(mangoGroup)
          .mul(I80F48.fromNumber(100)),
         utilization: totalDeposits.gt(I80F48.fromNumber(0))
           ? totalBorrows.div(totalDeposits)
           : I80F48.fromNumber(0),
      };
    });
```

## Mango Accounts

Each user can have multiple accounts associated with a single group. They function as a balance sheet and keep track of the deposits, borrows and open positions on the serum dex order book. A Mango Markets margin account is fully cross-margined and can be liquidated once the collateral ratio drops below maintenance margin level. When interacting with the margin account you will often need to provide the serum dex program-id in order to access the open orders on it's order book.

```typescript
import { IDS, MangoClient  } from '@blockworks-foundation/mango-client-v3';
import { Connection, PublicKey } from `@solana/web.js`

const cluster = 'mainnet';
const group = 'mainnet.1';
const myMangoAccountAddress = 'YOUR MANGO ACCOUNT ADDRESS GOES HERE';

const clusterData = IDS_v3.groups.find((g) => {
    return g.name == group && g.cluster == cluster;
  });
const mangoProgramIdPk = new PublicKey(clusterData.mangoProgramId);
const serumProgramIdPk = new PublicKey(clusterData.serumProgramId);

const clusterUrl = IDS_v3.cluster_urls[cluster];
const connection = new Connection(clusterUrl, 'singleGossip');

const myMangoAccountPubKey = new PublicKey(myMangoAccountAddress);
const client = new MangoClient(connection, mangoProgramIdPk);
const myMangoAccount = await client.getMangoAccount(myMangoAccountPubKey, serumProgramIdPk);
```
