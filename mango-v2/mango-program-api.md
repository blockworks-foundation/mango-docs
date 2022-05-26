# Mango Program API

This is a complete reference of all instructions processed by the Mango on-chain program. You might want to use the [Mango Client API ](../development-resources/mango-client-api.md)as a reference on how to call these instructions. You can find the typescript code creating the instructions on [github](https://github.com/blockworks-foundation/mango-client-ts/blob/main/src/client.ts).

```rust
#[repr(C)]
#[derive(Clone, Debug, PartialEq, Serialize, Deserialize)]
pub enum MangoInstruction {
    /// Initialize a group of lending pools that can be cross margined
    ///
    /// Accounts expected by this instruction (7 + 2 * NUM_TOKENS + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - the data account to store mango group state vars
    /// 1. `[]` rent_acc - Rent sysvar account
    /// 2. `[]` clock_acc - clock sysvar account
    /// 3. `[]` signer_acc - pubkey of program_id hashed with signer_nonce and mango_group_acc.key
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5. `[]` srm_vault_acc - vault for fee tier reductions
    /// 6. `[signer]` admin_acc - admin key who can change borrow limits
    /// 7..7+NUM_TOKENS `[]` token_mint_accs - mint of each token in the same order as the spot
    ///     markets. Quote currency mint should be last.
    ///     e.g. for spot markets BTC/USDC, ETH/USDC -> [BTC, ETH, USDC]
    ///
    /// 7+NUM_TOKENS..7+2*NUM_TOKENS `[]`
    ///     vault_accs - Vault owned by signer_acc.key for each of the mints
    ///
    /// 7+2*NUM_TOKENS..7+2*NUM_TOKENS+NUM_MARKETS `[]`
    ///     spot_market_accs - MarketState account from serum dex for each of the spot markets
    /// 7+2*NUM_TOKENS+NUM_MARKETS..7+2*NUM_TOKENS+2*NUM_MARKETS `[]`
    ///     oracle_accs - Solana Flux Aggregator accounts corresponding to each trading pair
    InitMangoGroup {
        signer_nonce: u64,
        maint_coll_ratio: U64F64,
        init_coll_ratio: U64F64,
        borrow_limits: [u64; NUM_TOKENS]
    },

    /// Initialize a margin account for a user
    ///
    /// Accounts expected by this instruction (4):
    ///
    /// 0. `[]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` margin_account_acc - the margin account data
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[]` rent_acc - Rent sysvar account
    InitMarginAccount,

    /// Deposit funds into margin account to be used as collateral and earn interest.
    ///
    /// Accounts expected by this instruction (7):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` margin_account_acc - the margin account for this user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[writable]` token_account_acc - TokenAccount owned by user which will be sending the funds
    /// 4. `[writable]` vault_acc - TokenAccount owned by MangoGroup
    /// 5. `[]` token_prog_acc - acc pointed to by SPL token program id
    /// 6. `[]` clock_acc - Clock sysvar account
    Deposit {
        quantity: u64
    },

    /// Withdraw funds that were deposited earlier.
    ///
    /// Accounts expected by this instruction (8 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` margin_account_acc - the margin account for this user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[writable]` token_account_acc - TokenAccount owned by user which will be receiving the funds
    /// 4. `[writable]` vault_acc - TokenAccount owned by MangoGroup which will be sending
    /// 5. `[]` signer_acc - acc pointed to by signer_key
    /// 6. `[]` token_prog_acc - acc pointed to by SPL token program id
    /// 7. `[]` clock_acc - Clock sysvar account
    /// 8..8+NUM_MARKETS `[]` open_orders_accs - open orders for each of the spot market
    /// 8+NUM_MARKETS..8+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    Withdraw {
        quantity: u64
    },

    /// Borrow by incrementing MarginAccount.borrows given collateral ratio is below init_coll_rat
    ///
    /// Accounts expected by this instruction (4 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` margin_account_acc - the margin account for this user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4..4+NUM_MARKETS `[]` open_orders_accs - open orders for each of the spot market
    /// 4+NUM_MARKETS..4+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    Borrow {
        token_index: usize,
        quantity: u64
    },

    /// Use this token's position and deposit to reduce borrows
    ///
    /// Accounts expected by this instruction (4):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` margin_account_acc - the margin account for this user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[]` clock_acc - Clock sysvar account
    SettleBorrow {
        token_index: usize,
        quantity: u64
    },

    /// Take over a MarginAccount that is below init_coll_ratio by depositing funds
    ///
    /// Accounts expected by this instruction (5 + 2 * NUM_MARKETS + 2 * NUM_TOKENS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` liqor_acc - liquidator's solana account
    /// 2. `[writable]` liqee_margin_account_acc - MarginAccount of liquidatee
    /// 3. `[]` token_prog_acc - SPL token program id
    /// 4. `[]` clock_acc - Clock sysvar account
    /// 5..5+NUM_MARKETS `[]` open_orders_accs - open orders for each of the spot market
    /// 5+NUM_MARKETS..5+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    /// 5+2*NUM_MARKETS..5+2*NUM_MARKETS+NUM_TOKENS `[writable]`
    ///     vault_accs - MangoGroup vaults
    /// 5+2*NUM_MARKETS+NUM_TOKENS..5+2*NUM_MARKETS+2*NUM_TOKENS `[writable]`
    ///     liqor_token_account_accs - Liquidator's token wallets
    Liquidate {
        /// Quantity of each token liquidator is depositing in order to bring account above maint
        deposit_quantities: [u64; NUM_TOKENS]
    },

    /// Deposit SRM into the SRM vault for MangoGroup
    /// These SRM are not at risk and are not counted towards collateral or any margin calculations
    /// Depositing SRM is a strictly altruistic act with no upside and no downside
    ///
    /// Accounts expected by this instruction (8):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` mango_srm_account_acc - the mango srm account for user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[writable]` srm_account_acc - TokenAccount owned by user which will be sending the funds
    /// 4. `[writable]` vault_acc - SRM vault of MangoGroup
    /// 5. `[]` token_prog_acc - acc pointed to by SPL token program id
    /// 6. `[]` clock_acc - Clock sysvar account
    /// 7. `[]` rent_acc - Rent sysvar account
    DepositSrm {
        quantity: u64
    },
    /// Withdraw SRM owed to this MarginAccount
    /// These SRM are not at risk and are not counted towards collateral or any margin calculations
    /// Depositing SRM is a strictly altruistic act with no upside and no downside
    ///
    /// Accounts expected by this instruction (8):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[writable]` mango_srm_account_acc - the mango srm account for user
    /// 2. `[signer]` owner_acc - Solana account of owner of the margin account
    /// 3. `[writable]` srm_account_acc - TokenAccount owned by user which will be sending the funds
    /// 4. `[writable]` vault_acc - SRM vault of MangoGroup
    /// 5. `[]` signer_acc - acc pointed to by signer_key
    /// 6. `[]` token_prog_acc - acc pointed to by SPL token program id
    /// 7. `[]` clock_acc - Clock sysvar account
    WithdrawSrm {
        quantity: u64
    },

    // Proxy instructions to Dex
    /// Place an order on the Serum Dex using Mango margin facilities
    ///
    /// Accounts expected by this instruction (17 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` owner_acc - MarginAccount owner
    /// 2. `[writable]` margin_account_acc - MarginAccount
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5. `[writable]` spot_market_acc - serum dex MarketState
    /// 6. `[writable]` dex_request_queue_acc - serum dex request queue for this market
    /// 7. `[writable]` dex_event_queue - serum dex event queue for this market
    /// 8. `[writable]` bids_acc - serum dex bids for this market
    /// 9. `[writable]` asks_acc - serum dex asks for this market
    /// 10. `[writable]` vault_acc - mango's vault for this currency (quote if buying, base if selling)
    /// 11. `[]` signer_acc - mango signer key
    /// 12. `[writable]` dex_base_acc - serum dex market's vault for base (coin) currency
    /// 13. `[writable]` dex_quote_acc - serum dex market's vault for quote (pc) currency
    /// 14. `[]` spl token program
    /// 15. `[]` the rent sysvar
    /// 16. `[writable]` srm_vault_acc - MangoGroup's srm_vault used for fee reduction
    /// 17..17+NUM_MARKETS `[writable]` open_orders_accs - open orders for each of the spot market
    /// 17+NUM_MARKETS..17+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    PlaceOrder {
        order: serum_dex::instruction::NewOrderInstructionV3
    },

    /// Settle all funds from serum dex open orders into MarginAccount positions
    ///
    /// Accounts expected by this instruction (14):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` owner_acc - MarginAccount owner
    /// 2. `[writable]` margin_account_acc - MarginAccount
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5  `[writable]` spot_market_acc - dex MarketState account
    /// 6  `[writable]` open_orders_acc - open orders for this market for this MarginAccount
    /// 7. `[]` signer_acc - MangoGroup signer key
    /// 8. `[writable]` dex_base_acc - base vault for dex MarketState
    /// 9. `[writable]` dex_quote_acc - quote vault for dex MarketState
    /// 10. `[writable]` base_vault_acc - MangoGroup base vault acc
    /// 11. `[writable]` quote_vault_acc - MangoGroup quote vault acc
    /// 12. `[]` dex_signer_acc - dex Market signer account
    /// 13. `[]` spl token program
    SettleFunds,

    /// Cancel an order using dex instruction
    ///
    /// Accounts expected by this instruction (11):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` owner_acc - MarginAccount owner
    /// 2. `[]` margin_account_acc - MarginAccount
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5. `[writable]` spot_market_acc - serum dex MarketState
    /// 6. `[writable]` bids_acc - serum dex bids
    /// 7. `[writable]` asks_acc - serum dex asks
    /// 8. `[writable]` open_orders_acc - OpenOrders for the market this order belongs to
    /// 9. `[]` signer_acc - MangoGroup signer key
    /// 10. `[writable]` dex_event_queue_acc - serum dex event queue for this market
    CancelOrder {
        order: serum_dex::instruction::CancelOrderInstructionV2
    },

    /// Cancel an order using client_id
    ///
    /// Accounts expected by this instruction (11):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` owner_acc - MarginAccount owner
    /// 2. `[]` margin_account_acc - MarginAccount
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5. `[writable]` spot_market_acc - serum dex MarketState
    /// 6. `[writable]` bids_acc - serum dex bids
    /// 7. `[writable]` asks_acc - serum dex asks
    /// 8. `[writable]` open_orders_acc - OpenOrders for the market this order belongs to
    /// 9. `[]` signer_acc - MangoGroup signer key
    /// 10. `[writable]` dex_event_queue_acc - serum dex event queue for this market
    CancelOrderByClientId {
        client_id: u64
    },

    /// Change the borrow limit using admin key. This will not affect any open positions on any MarginAccount
    /// This is intended to be an instruction only in alpha stage while liquidity is slowly improved
    ///
    /// Accounts expected by this instruction (2):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` admin_acc - admin of the MangoGroup
    ChangeBorrowLimit {
        token_index: usize,
        borrow_limit: u64
    },

    /// Place an order on the Serum Dex and settle funds from the open orders account
    ///
    /// Accounts expected by this instruction (19 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` owner_acc - MarginAccount owner
    /// 2. `[writable]` margin_account_acc - MarginAccount
    /// 3. `[]` clock_acc - Clock sysvar account
    /// 4. `[]` dex_prog_acc - program id of serum dex
    /// 5. `[writable]` spot_market_acc - serum dex MarketState
    /// 6. `[writable]` dex_request_queue_acc - serum dex request queue for this market
    /// 7. `[writable]` dex_event_queue - serum dex event queue for this market
    /// 8. `[writable]` bids_acc - serum dex bids for this market
    /// 9. `[writable]` asks_acc - serum dex asks for this market
    /// 10. `[writable]` base_vault_acc - mango vault for base currency
    /// 11. `[writable]` quote_vault_acc - mango vault for quote currency
    /// 12. `[]` signer_acc - mango signer key
    /// 13. `[writable]` dex_base_acc - serum dex market's vault for base (coin) currency
    /// 14. `[writable]` dex_quote_acc - serum dex market's vault for quote (pc) currency
    /// 15. `[]` spl token program
    /// 16. `[]` the rent sysvar
    /// 17. `[writable]` srm_vault_acc - MangoGroup's srm_vault used for fee reduction
    /// 18. `[]` dex_signer_acc - signer for serum dex MarketState
    /// 19..19+NUM_MARKETS `[writable]` open_orders_accs - open orders for each of the spot market
    /// 19+NUM_MARKETS..19+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    PlaceAndSettle {
        order: serum_dex::instruction::NewOrderInstructionV3
    },

    /// Allow a liquidator to cancel open orders and settle to recoup funds for partial liquidation
    ///
    /// Accounts expected by this instruction (16 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` liqor_acc - liquidator's solana account
    /// 2. `[writable]` liqee_margin_account_acc - MarginAccount of liquidatee
    /// 3. `[writable]` base_vault_acc - mango vault for base currency
    /// 4. `[writable]` quote_vault_acc - mango vault for quote currency
    /// 5. `[writable]` spot_market_acc - serum dex MarketState
    /// 6. `[writable]` bids_acc - serum dex bids for this market
    /// 7. `[writable]` asks_acc - serum dex asks for this market
    /// 8. `[]` signer_acc - mango signer key
    /// 9. `[writable]` dex_event_queue - serum dex event queue for this market
    /// 10. `[writable]` dex_base_acc - serum dex market's vault for base (coin) currency
    /// 11. `[writable]` dex_quote_acc - serum dex market's vault for quote (pc) currency
    /// 12. `[]` dex_signer_acc - signer for serum dex MarketState
    /// 13. `[]` token_prog_acc - SPL token program
    /// 14. `[]` dex_prog_acc - Serum dex program id
    /// 15. `[]` clock_acc - Clock sysvar account
    /// 16..16+NUM_MARKETS `[writable]` open_orders_accs - open orders for each of the spot market
    /// 16+NUM_MARKETS..16+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    ForceCancelOrders {
        /// Max orders to cancel -- could be useful to lower this if running into compute limits
        /// Recommended: 5
        limit: u8
    },

    /// Take over a MarginAccount that is below init_coll_ratio by depositing funds
    ///
    /// Accounts expected by this instruction (10 + 2 * NUM_MARKETS):
    ///
    /// 0. `[writable]` mango_group_acc - MangoGroup that this margin account is for
    /// 1. `[signer]` liqor_acc - liquidator's solana account
    /// 2. `[writable]` liqor_in_token_acc - liquidator's token account to deposit
    /// 3. `[writable]` liqor_out_token_acc - liquidator's token account to withdraw into
    /// 4. `[writable]` liqee_margin_account_acc - MarginAccount of liquidatee
    /// 5. `[writable]` in_vault_acc - Mango vault of in_token
    /// 6. `[writable]` out_vault_acc - Mango vault of out_token
    /// 7. `[]` signer_acc
    /// 8. `[]` token_prog_acc - Token program id
    /// 9. `[]` clock_acc - Clock sysvar account
    /// 10..10+NUM_MARKETS `[]` open_orders_accs - open orders for each of the spot market
    /// 10+NUM_MARKETS..10+2*NUM_MARKETS `[]`
    ///     oracle_accs - flux aggregator feed accounts
    PartialLiquidate {
        /// Quantity of the token being deposited to repay borrows
        max_deposit: u64
    }
}
```
