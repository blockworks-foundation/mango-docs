# 🫂 Account Delegation

Mango Markets Account Delegation is a feature that allows a separate Solana account limited access to the Mango Markets Account's features.

Delegation can provide some additional security for more sophisticated key management procedures.

A delegated account _can_, for example:

* Deposit funds
* Place orders
* Cancel orders

A delegated account _cannot_, for example:

* Withdraw funds
* Close the Mango Markets Account
* Change or remove existing account delegation

A Mango Markets Account can have, at most, one delegate. The owner of the Mango Markets Account continues to have full control of the Mango Markets Account irrespective of delegation.

{% hint style="warning" %}
Only delegate to an account that you trust. Although delegates cannot withdraw funds, there are some other mechanisms by which they could drain your account. For example, by using your account to make bad trades with their personal account.
{% endhint %}

### Usage

A delegate can be set from the Mango Markets UI via the accounts page, or programmatically using either the `mango-explorer` package for Python or the `mango-client` library for JS.

If a Mango Markets Account has been delegated to you, it will appear in the Accounts List in the UI, from where you can select it and perform limited operations as you would with any other account.

* [Mango Explorer Documentation](https://github.com/blockworks-foundation/mango-explorer/blob/main/docs/Delegation.md)
* [Mango Client Documentation](https://blockworks-foundation.github.io/mango-client-v3/classes/MangoClient.html#setDelegate)
