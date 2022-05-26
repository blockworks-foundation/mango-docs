# TIF (Time in Force) Expiry

The time on the validator is used to determine if a TIF order has expired.

When retrieving the current cluster time, be sure to use the "Processed" commitment level.\
\*\*Solana Explorer reports the cluster time using the "Finalized" commitment level, which is slightly different.

### Retrieving the Cluster Time

You can retrieve the current cluster time by using the getBlockTime endpoint. [https://docs.solana.com/developing/clients/jsonrpc-api#getblocktime](https://docs.solana.com/developing/clients/jsonrpc-api#getblocktime)

