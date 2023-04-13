---
sidebar_position: 2
---

# Integration Guidelines


## Overview

For calling swap function of your dex from Umi aggregator contract, we have the plan to publish only the interface of your dex to https://github.com/umi-ag/sui-aggregator-interface.

We’d like you to create a pull request to this repository.

> We do not charge fees for integration.


## Requirements

- ### Smart Contract in your DEX
    - #### The swap function must be declared as public.
        - In order to call the swap function in your smart contract from Umi contract on-chain, the swap function needs to be public.
        - You don't have to make all functions public.
    - #### The return type of the swap function must be `sui::balance::Balance` or `sui::coin::Coin`.

<br />

- ### Aggregator Interface in https://github.com/umi-ag/sui-aggregator-interface

    - #### The swap function
        - You need to declare the swap function
            - public function
            - swap function name ( the same name as the swap function in your contract )
            - arguments
            - type arguments
            - return value
        - The logic does not need to be disclosed; `abort 0` is sufficient.

    - #### Structure declaring type arguments of the swap function
        - You need to declare in aggregator interface
            - structure that has dependency relationship with the swap function



## Example Code

Smart Contract in your DEX

```js title="your_dex.move"
module udoswap::dex {
    use sui::object::{Self, UID};
    use sui::coin::{Self, Coin};
    use sui::balance::{Self, Supply, Balance};
    use sui::transfer;
    use sui::math;
    use sui::tx_context::{Self, TxContext};

    struct LSP<phantom X, phantom Y> has drop {}

    struct Pool<phantom X, phantom Y> has key {
        id: UID,
        reserve_x: Balance<X>,
        reserve_y: Balance<Y>,
        lsp_supply: Supply<LSP<X, Y>>,
        fee_percent: u64
    }

    fun init(_: &mut TxContext) {}

    // The swap function must be declared as public
    // The return type of the swap function must be `sui::balance::Balance` or `sui::coin::Coin`
    public fun swap_x_to_y_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, ctx: &mut TxContext
    ): Coin<Y> {

        let balance_x = coin::into_balance(coin_x);

        let (reserve_x, reserve_y, _) = get_amounts(pool);

        let output_amount = get_input_price(
            balance::value(&balance_x),
            reserve_x,
            reserve_y,
            pool.fee_percent
        );

        balance::join(&mut pool.reserve_x, balance_x);
        coin::take(&mut pool.reserve_y, output_amount, ctx)
    }

    public fun swap_y_to_x_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_y: Coin<Y>, ctx: &mut TxContext
    ): Coin<X> {

        let balance_y = coin::into_balance(coin_y);
        let (reserve_x, reserve_y, _) = get_amounts(pool);


        let output_amount = get_input_price(
            balance::value(&balance_y),
            reserve_y,
            reserve_x,
            pool.fee_percent
        );

        balance::join(&mut pool.reserve_y, balance_y);
        coin::take(&mut pool.reserve_x, output_amount, ctx)
    }

    entry fun swap_x_to_y<X, Y>(pool: &mut Pool<X, Y>, coin_x: Coin<X>, ctx: &mut TxContext) {
        transfer::public_transfer(
            swap_x_to_y_direct(pool, coin_x, ctx),
            tx_context::sender(ctx)
        )
    }

    entry fun swap_y_to_x<X, Y>(
        pool: &mut Pool<X, Y>, coin_y: Coin<Y>, ctx: &mut TxContext
    ) {
        let sernder = tx_context::sender(ctx);

        transfer::public_transfer(
            swap_y_to_x_direct(pool, coin_y, ctx),
            tx_context::sender(ctx)
        )
    }

    // Just swap function needs to be public
    // You don't have to make all functions public
    fun add_liquidity_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, coin_y: Coin<Y>, ctx: &mut TxContext
    ): Coin<LSP<X, Y>>
    {

        let balance_x = coin::into_balance(coin_x);
        let balance_y = coin::into_balance(coin_y);

        let (reserve_x, reserve_y, lsp_supply) = get_amounts(pool);

        let x_added = balance::value(&balance_x);
        let y_added = balance::value(&balance_y);
        let share_minted = if (reserve_x * reserve_y > 0) {
            math::min(
                (x_added * lsp_supply) / reserve_x,
                (y_added * lsp_supply) / reserve_y
            )
        } else {
            math::sqrt(x_added) * math::sqrt(y_added)
        };

        let coin_amount_x = balance::join(&mut pool.reserve_x, balance_x);
        let coin_amount_y = balance::join(&mut pool.reserve_y, balance_y);


        let balance = balance::increase_supply(&mut pool.lsp_supply, share_minted);
        coin::from_balance(balance, ctx)
    }
}

```

Umi Aggregator Interface

```js title="interface.move"
module udoswap::dex {

    // declare LSP<X, Y> that has dependency relationship with Pool<X, Y>
    struct LSP<phantom X, phantom Y> has drop {}

    // declare Pool<X, Y> that has dependency relationship with the swap function
    // You need to declare LSP<X, Y> that has dependency relationship with this structure
    struct Pool<phantom X, phantom Y> has key {
        id: UID,
        reserve_x: Balance<X>,
        reserve_y: Balance<Y>,
        lsp_supply: Supply<LSP<X, Y>>,
        fee_percent: u64
    }

    fun init(_: &mut TxContext) {}

    // Just the interface of the swap function
    // public function, swap function name, arguments, type arguments, return value
    // The logic of the function does not need to be disclosed; `abort 0` is sufficient
    // You need to declare Pool<X, Y> that has dependency relationship with this function
    public fun swap_x_to_y_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_x: Coin<X>, ctx: &mut TxContext
    ): Coin<Y> {
        abort 0
    }

    public fun swap_y_to_x_direct<X, Y>(
        pool: &mut Pool<X, Y>, coin_y: Coin<Y>, ctx: &mut TxContext
    ): Coin<X> {
        abort 0
    }
}

```


