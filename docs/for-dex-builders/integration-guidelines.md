---
sidebar_position: 2
---

# Integration Guidelines


## Configure i18n

Modify `docusaurus.config.js` to add support for the `fr` locale:

```js title="your_dex.move"
module umaswap::dex {
    use sui::object::{Self, UID};
    use sui::coin::{Self, Coin};
    use sui::balance::{Self, Supply, Balance};

    /// The Pool token that will be used to mark the pool share
    /// of a liquidity provider. The first type parameter stands
    /// for the witness type of a pool. The seconds is for the
    /// coin held in the pool.
    struct LSP<phantom X, phantom Y> has drop {}

    /// The pool with exchange.
    ///
    /// - `fee_percent` should be in the range: [0-10000), meaning
    /// that 1000 is 100% and 1 is 0.1%
    struct Pool<phantom X, phantom Y> has key {
        id: UID,
        reserve_x: Balance<X>,
        reserve_y: Balance<Y>,
        lsp_supply: Supply<LSP<X, Y>>,
        /// Fee Percent is denominated in basis points.
        fee_percent: u64
    }

    /// Module initializer is empty - to publish a new Pool one has
    /// to create a type which will mark LSPs.
    fun init(_: &mut TxContext) {}

    public fun swap_x_to_y_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, ctx: &mut TxContext
    ): Coin<Y> {
        abort 0
    }

    /// Swap `Coin<T>` for the `Coin<SUI>`.
    /// Returns the swapped `Coin<SUI>`.
    public fun swap_y_to_x_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_y: Coin<Y>, ctx: &mut TxContext
    ): Coin<X> {
        abort 0
    }
}
```

## Must be

- 

