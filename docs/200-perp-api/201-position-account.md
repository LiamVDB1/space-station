---
sidebar_label: "Position Account"
description: "Understand and integrate the Jupiter Perp Program."
title: "Position Account"
---

<head>
    <title>Position Account</title>
    <meta name="twitter:card" content="summary" />
</head>

This page contains an overview of the **Solana account types** used in the Jupiter Perpetuals Program, and specifically the `Position` account.

The `Position` account is a struct which represents a set of parameters and states associated to trade position data for a given token.

:::note `Position` account derivation
The `Position` account's address is derived from the trader's wallet address / public key, the custody account, the collateral custody account, and a few other constant seeds. This means traders will always have the same Position account address for their open positions.

This also means that traders only have nine positions available at one time:

- Long SOL
- Long wETH
- Long wBTC
- Short SOL (USDC as collateral)
- Short SOL (USDT as collateral)
- Short wETH (USDC as collateral)
- Short wETH (USDT as collateral)
- Short wBTC (USDC as collateral)
- Short wBTC (USDT as collateral)

This is an example [`Position` account](https://solscan.io/account/FBLzd5VM67MEKkoWerXu7Nu1ksbLXQvJDx63y5aeLEvt).
:::

:::tip Example Typescript Repository
This [repository](https://github.com/julianfssen/jupiter-perps-anchor-idl-parsing) contains Typescript code samples on interacting with the Jupiter Perpetuals program IDL with `anchor` and `@solana/web3.js`

You can also find the [Custody Account fields in the repository](https://github.com/julianfssen/jupiter-perps-anchor-idl-parsing/blob/1a0b5dc71081958895691047a9aa8ba51d2a8765/src/idl/jupiter-perpetuals-idl.ts#L2699) or on a [blockchain explorer](https://solscan.io/account/PERPHjGBqRHArX4DySjwM6UJHiR3sWAatqfdBS2qQJu#anchorProgramIdl).
:::

## Account Details

Each `Position` account contains the following data:

| Field | Description |
| --- | --- |
| `owner`                       | **Type:** `publicKey`<br /><br />The public key of the trader's account. |
| `pool`                        | **Type:** `publicKey`<br /><br />The public key of the [JLP pool account](./pool-account). |
| `custody`                     | **Type:** `publicKey`<br /><br />The public key of the position's [`custody` account](./custody-account). |
| `collateralCustody`           | **Type:** `publicKey`<br /><br />The public key of the position's collateral custody account.<br /><br />Like the `custody` account, a `collateralCustody` account contains information for the token that's used as collateral for the position (SOL / wETH / wBTC for long positions, USDC / USDT for short positions). The borrow rates for the position will also be calculated based on the position's `collateralCustody`. |
| `openTime`                    | **Type:** `i64`<br /><br />The open time of the position in UNIX timestamp format. |
| `updateTime`                  | **Type:** `i64`<br /><br />The last updated time of the position in UNIX timestamp format. |
| `side`                        | **Type:** `Side`<br /><br />The position's side, either `long` or `short`. |
| `price`                       | **Type:** `u64`<br /><br />The entry price of the position when it was opened. The entry price is an integer in the atomic value (before decimals), a USDC (6 decimals) value of `158225872` is equivalent to $158.22. |
| `sizeUsd`                     | **Type:** `u64`<br /><br />The position size after leverage in USD in the atomic value (before decimals). A position with `sizeUsd = 0` is treated as a closed position. |
| `collateralUsd`               | **Type:** `u64`<br /><br />The position's collateral size after fees in USD in the atomic value (before decimals). |
| `realisedPnlUsd`              | **Type:** `i64`<br /><br />The position's realized PNL when closing the position partially.<br /><br />When a position is closed completely, the position's `realisedPnlUsd` will be `0` as the position is considered closed (as described in `sizeUsd`). |
| `cumulativeInterestSnapshot`  | **Type:** `u128`<br /><br />Stores the position's interest rate snapshot when it was last updated.<br /><br />- The `collateralCustody` account for the respective collateral token stores a monotonically increasing counter in `collateralCustody .fundingRateState .cumulativeInterestRate`.<br /><br />The difference between the `collateralCustody .fundingRateState .cumulativeInterestRate` and the position's `cumulativeInterestSnapshot` is used to calculate the borrow fees for the position. |
| `lockedAmount`                | **Type:** `u64`<br /><br />The amount of tokens (SOL / wETH / wBTC for long positions, USDC / USDT for short positions) locked to pay off the position's max potential profit. It acts as a cap on the maximum potential profit of the position. This amount is locked in the collateral custody to ensure the platform has sufficient tokens to pay out profitable trades. |
| `bump`                        | **Type:** `u8`<br /><br />The bump seed used to derive the PDA for the `Position` account. |
