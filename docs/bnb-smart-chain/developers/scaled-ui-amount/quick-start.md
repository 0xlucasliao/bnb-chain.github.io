---
title: Quick Start - Scaled UI Amount (ERC-8056)
---

# Quick Start: Integrate the UI Multiplier

This demo shows the four things almost every app needs: detect an ERC-8056 token, show the correct balance, convert a user-entered amount into a transfer, and react when the multiplier changes.

## 1. Set up the contract

```js
import { ethers } from "ethers";

const IScaledUIAmountABI = [
  "function uiMultiplier() view returns (uint256)",
  "function toUIAmount(uint256 rawAmount) view returns (uint256)",
  "function fromUIAmount(uint256 uiAmount) view returns (uint256)",
  "function balanceOfUI(address account) view returns (uint256)",
  "function balanceOf(address account) view returns (uint256)",
  "function decimals() view returns (uint8)",
  "function transfer(address to, uint256 rawAmount) returns (bool)",
  "function supportsInterface(bytes4 interfaceId) view returns (bool)",
  "event UIMultiplierUpdated(uint256 oldMultiplier, uint256 newMultiplier, uint256 setAtTimestamp, uint256 effectiveAtTimestamp)"
];

const provider = new ethers.BrowserProvider(window.ethereum);
const token = new ethers.Contract(tokenAddress, IScaledUIAmountABI, provider);

// Ask the token issuer for the official ERC-8056 interface ID, or maintain
// a curated allowlist of known ERC-8056 token addresses as a fallback.
const isScaledUIAmount = await token.supportsInterface(ISCALED_UI_AMOUNT_INTERFACE_ID);
```

## 2. Display the correct balance

Never show `balanceOf()` directly — call `balanceOfUI()` instead:

```js
const decimals = await token.decimals();
const rawUiBalance = await token.balanceOfUI(userAddress);
const uiBalance = Number(rawUiBalance) / 10 ** decimals;

console.log(`Balance: ${uiBalance} shares`); // e.g. "Balance: 200 shares"
```

## 3. Convert a user amount into a transfer

Users think in UI amounts ("send 200 shares"). Convert to raw before calling `transfer()`:

```js
async function sendShares(recipient, uiAmountString) {
  const decimals = await token.decimals();
  const uiAmountScaled = ethers.parseUnits(uiAmountString, decimals); // e.g. "200" -> 200_000_000
  const rawAmount = await token.fromUIAmount(uiAmountScaled);          // e.g. 100_000_000 raw (if multiplier = 2.0x)

  const signer = await provider.getSigner();
  const tx = await token.connect(signer).transfer(recipient, rawAmount);
  await tx.wait();
}

await sendShares("0xRecipient...", "200");
```

## 4. React to multiplier changes

Corporate actions (splits, dividends) fire `UIMultiplierUpdated`, sometimes with a future `effectiveAtTimestamp`. Refresh your cached balances and prices once that time is reached:

```js
token.on("UIMultiplierUpdated", (oldMultiplier, newMultiplier, setAtTimestamp, effectiveAtTimestamp) => {
  const now = Math.floor(Date.now() / 1000);

  if (Number(effectiveAtTimestamp) > now) {
    // Scheduled for later — show a notice, don't change anything yet.
    console.log(`Multiplier will change to ${Number(newMultiplier) / 1e18}x at ${effectiveAtTimestamp}`);
  } else {
    // Already active — refresh displayed balances/prices now.
    refreshBalances();
  }
});
```

## Putting it together

A minimal end-to-end demo: connect a wallet, print the UI-adjusted balance, and send a UI-denominated amount.

```js
async function demo(tokenAddress, userAddress) {
  const token = new ethers.Contract(tokenAddress, IScaledUIAmountABI, provider);
  const decimals = await token.decimals();

  const multiplier = Number(await token.uiMultiplier()) / 1e18;
  console.log(`Current UI multiplier: ${multiplier}x`);

  const uiBalance = Number(await token.balanceOfUI(userAddress)) / 10 ** decimals;
  console.log(`Balance: ${uiBalance} shares`);

  await sendShares("0xRecipient...", "10");
}
```

## Common mistakes to avoid

- Don't pass a UI amount straight into `transfer()` / `transferFrom()` / `approve()` — always convert with `fromUIAmount()` first.
- Don't apply a new multiplier before its `effectiveAtTimestamp` — the old one is still active until then.
- Don't hand-roll the `rawAmount × uiMultiplier / 1e18` math for display — call `toUIAmount()` / `balanceOfUI()` so rounding always matches the contract.

## Next steps

For per-audience integration details (wallets, explorers, DEXs, lending protocols) and how to handle splits vs. dividends safely, contact the token issuer for the full ERC-8056 Partner Integration Guide.
