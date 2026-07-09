---
title: Overview - Scaled UI Amount (ERC-8056)
---

# Scaled UI Amount (ERC-8056) Developer Kit

Some tokens on BNB Smart Chain — such as tokenized stocks — implement ERC-8056 (Scaled UI Amount). This standard adds a **UI multiplier** to a token contract, so corporate actions like stock splits, reverse splits, and dividends can be reflected on-chain without minting, burning, or migrating any tokens.

This kit gives you the minimum you need to correctly display balances, transfers, and prices for these tokens. If you skip it, your users will see wrong balances and prices whenever a multiplier changes.

## What is the UI Multiplier?

The UI multiplier is a single scalar value stored on the token contract that converts a raw on-chain balance into the amount you should show a user.

```
UI Amount = rawAmount × uiMultiplier / 1e18
```

It's an 18-decimal fixed-point number: `1e18` means "no change" (1.0×), `2e18` means "double" (2.0×), `5e17` means "half" (0.5×).

### Example — 2-for-1 stock split

| | Before Split | After Split |
|---|---|---|
| Raw on-chain balance | 100 tokens | 100 tokens (unchanged) |
| UI Multiplier | 1.0× | 2.0× |
| UI display amount | 100 shares | 200 shares |
| Total value | $5,000 | $5,000 (unchanged) |

No tokens move. Only the multiplier changes — and every balance, transfer amount, and price you display should scale with it.

## Why it matters

If your app reads `balanceOf()` directly and shows that number, it will be wrong the moment a multiplier changes. Always go through the multiplier-aware helpers described in the [Quick Start guide](./quick-start.md).

## Where to go next

- [Quick Start & Demo](./quick-start.md) — a runnable example that detects an ERC-8056 token, shows the correct balance, and converts a user-entered amount for a transfer.
