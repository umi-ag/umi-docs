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
    use sui::transfer;
    use sui::math;
    use sui::tx_context::{Self, TxContext};

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

    entry fun add_liquidity<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, coin_y: Coin<Y>, ctx: &mut TxContext
    ) {
        abort 0
    }

    public fun add_liquidity_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, coin_y: Coin<Y>, ctx: &mut TxContext
    ): Coin<LSP<X, Y>> {
        abort 0
    }

    /// Entrypoint for the `remove_liquidity` method. Transfers
    /// withdrawn assets to the sender.
    entry fun remove_liquidity_<X, Y>(
        pool: &mut Pool<X, Y>,
        lsp: Coin<LSP<X, Y>>,
        ctx: &mut TxContext
    ) {
        abort 0;
    }

    /// Remove liquidity from the `Pool` by burning `Coin<LSP>`.
    /// Returns `Coin<T>` and `Coin<SUI>`.
    public fun remove_liquidity<X, Y>(
        pool: &mut Pool<X, Y>,
        lsp: Coin<LSP<X, Y>>,
        ctx: &mut TxContext
    ): (Coin<X>, Coin<Y>) {
        abort 0
    }

    public fun price_x_to_y<X, Y>(pool: &Pool<X, Y>, delta_y: u64): u64 { 1 }

    public fun price_y_to_x<X, Y>(pool: &Pool<X, Y>, delta_x: u64): u64 { 1 }


    /// Get most used values in a handy way:
    /// - amount of CoinX
    /// - amount of CoinY
    /// - total supply of LSP
    public fun get_amounts<X, Y>(pool: &Pool<X, Y>): (u64, u64, u64) {
        (
            balance::value(&pool.reserve_x),
            balance::value(&pool.reserve_y),
            balance::supply_value(&pool.lsp_supply)
        )
    }

    /// Calculate the output amount minus the fee - 0.3%
    public fun get_input_price(
        input_amount: u64, input_reserve: u64, output_reserve: u64, fee_percent: u64
    ): u64 { 1 }
}
```

## Translate a doc

Copy the `docs/intro.md` file to the `i18n/fr` folder:

```bash
mkdir -p i18n/fr/docusaurus-plugin-content-docs/current/

cp docs/intro.md i18n/fr/docusaurus-plugin-content-docs/current/intro.md
```

Translate `i18n/fr/docusaurus-plugin-content-docs/current/intro.md` in French.

## Start your localized site

Start your site on the French locale:

```bash
npm run start -- --locale fr
```

Your localized site is accessible at [http://localhost:3000/fr/](http://localhost:3000/fr/) and the `Getting Started` page is translated.

:::caution

In development, you can only use one locale at a same time.

:::

## Add a Locale Dropdown

To navigate seamlessly across languages, add a locale dropdown.

Modify the `docusaurus.config.js` file:

```js title="docusaurus.config.js"
module.exports = {
  themeConfig: {
    navbar: {
      items: [
        // highlight-start
        {
          type: 'localeDropdown',
        },
        // highlight-end
      ],
    },
  },
};
```

The locale dropdown now appears in your navbar:

![Locale Dropdown](./img/localeDropdown.png)

## Build your localized site

Build your site for a specific locale:

```bash
npm run build -- --locale fr
```

Or build your site to include all the locales at once:

```bash
npm run build
```
