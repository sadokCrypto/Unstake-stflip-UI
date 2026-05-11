# unstake-stFLIP UI

A single-file browser UI for unstaking **stFLIP → FLIP** on Ethereum mainnet using MetaMask (or any EIP-6963 wallet). No server, no build step, no private keys on disk.

Wraps the on-chain `unstakeAggregate` flow from [kylezs/unstake-stFLIP](https://github.com/kylezs/unstake-stFLIP) — same contract calls, just signed through your browser wallet instead of a hot-key Node script.

---

## Why this exists

The reference repo expects you to put a `PRIVATE_KEY` in a `.env` file and run a Node script. That works, but it means:

- Your private key sits unencrypted on disk
- You need a separate "throwaway" address to keep your main wallet safe
- You have to send stFLIP into the throwaway, unstake, then send FLIP back out

This UI does the same on-chain operations but lets MetaMask hold the key. You can unstake directly from your main wallet (or hardware wallet via MetaMask) without ever exposing the private key.

---

## What it does

Two transactions, exactly matching `unstake.mjs` from the source repo:

1. **`approve(AGGREGATOR, amount)`** on the stFLIP token contract
2. **`unstakeAggregate(amount, 0, 0, 0)`** on the aggregator contract

The UI also reads, before you commit:

- Your stFLIP / FLIP / ETH balances
- The current allowance (skips approve if you've already approved enough)
- The on-chain instant liquidity (`output FLIP balance - totalPendingBurns`), so you know if your amount will revert

---

## Requirements

- A modern browser (Chrome, Brave, Firefox, Edge)
- [MetaMask](https://metamask.io/download/) extension installed and unlocked
- An Ethereum mainnet account with:
  - stFLIP you want to unstake
  - A small amount of ETH for gas
- Python 3 **or** Node.js (only to serve the file locally — see below)

---

## Setup

### Step 1 — Save the file

Save `unstake-stflip.html` anywhere on your computer (e.g., `~/Downloads`).

### Step 2 — Serve it over `http://localhost`

⚠️ **You cannot just double-click the HTML file.** Browsers block wallet extensions from injecting into `file://` pages — MetaMask will not respond to the connect button if you open it that way. The page detects this and shows a big red error if you try.

Open a terminal in the folder containing the file and run **one** of these:

```bash
# Option A — Python (already installed on macOS/Linux)
python3 -m http.server 8000

# Option B — Node.js
npx serve .

# Option C — Node.js, alternative
npx http-server -p 8000
```

### Step 3 — Open it in your browser

Go to **<http://localhost:8000/unstake-stflip.html>**

The environment panel at the top should show:

- `protocol: http:`
- `window.ethereum: present`
- `providers found: 1` (or more, if you have multiple wallets)

If any of those are wrong, see [Troubleshooting](#troubleshooting).

---

## How to unstake

1. **Connect wallet** — Click the "Connect MetaMask" button in the wallet panel. MetaMask pops up; approve the connection.
2. **Verify network** — Make sure MetaMask is on Ethereum mainnet (chain id 1). The UI shows a red warning if you're on the wrong network.
3. **Check liquidity** — The liquidity panel auto-loads after connect. Note the **max instant** value — your unstake amount cannot exceed it or the transaction will revert.
4. **Enter amount** — Type the stFLIP amount, or click "use max" to fill in your full balance.
5. **Approve** — Click "1 · Approve stFLIP". MetaMask pops up. Verify the spender address (`0x38d8d03dFA9554D2232D4249EB23c48c23a24fA4` — the aggregator) and confirm.
6. **Unstake** — Once approve is mined, click "2 · Unstake". A JS confirm shows the parameters, then MetaMask pops up. Verify and confirm.
7. **Done** — FLIP arrives in the same address. The activity log links the transaction hashes to Etherscan.

---

## Contract addresses (mainnet)

| Contract | Address |
|----------|---------|
| stFLIP token | `0x961D4921e1718E633BAC8Ded88c4a1cAe44b785a` |
| Aggregator | `0x38d8d03dFA9554D2232D4249EB23c48c23a24fA4` |
| Burner | `0xb4078E779F4a982f27109522E2BA07dd9E133252` |
| Output | `0x6345A9F7e7069D478FFF3595f1522f28d8405151` |
| FLIP token | `0x826180541412D574cf1336d22c0C0a287822678A` |

These are hardcoded in the HTML and match `unstake.mjs` from the reference repo. **Verify them against the source repo before signing transactions.**

---

## Troubleshooting

### "You opened this with file://" banner

Browsers don't inject wallet extensions into local file pages. Follow [Setup](#setup) to serve over `http://localhost`.

### MetaMask popup doesn't appear when clicking connect

Check the environment panel:

- **`window.ethereum: undefined`** — MetaMask isn't installed, or is disabled in your browser's extension list. Install/enable it and reload.
- **`providers found: 0` but `window.ethereum: present`** — older wallet without EIP-6963 support. The "Connect via window.ethereum" fallback button should appear.
- **Multiple wallets listed** (Rabby, Coinbase Wallet, Phantom, etc.) — pick the MetaMask button specifically. Other wallets can hijack the generic connect path.

Also try:

- Click the MetaMask extension icon and make sure it's **unlocked**
- Check the browser address bar for a popup-blocker icon
- If the log shows error code `-32002`, MetaMask has a pending request — open the extension to handle it
- If the log shows error code `4001`, you rejected the request in MetaMask

### "exceeds instant liquidity" warning

The output contract doesn't currently hold enough FLIP for your unstake amount. Options: try a smaller amount, or wait and check liquidity again later (the "Refresh balances" button re-reads it).

### "Wrong network" warning

MetaMask isn't on Ethereum mainnet. Open MetaMask → network dropdown → select Ethereum Mainnet. The page will auto-reload.

### Transaction reverts on-chain

Most common cause: liquidity dropped between when you read it and when your tx mined. The amount also has to be within `stflipBalance` and the `allowance` set in step 1. Check the Etherscan link from the log for the revert reason.

---

## Security notes

- **Your private key never leaves MetaMask.** This page only constructs unsigned transactions and asks MetaMask to sign them. The signing happens inside the extension.
- **Verify everything in MetaMask before signing.** MetaMask shows you the destination contract address and the decoded function call. If the contract address doesn't match the table above, **reject** and report it.
- **CDN dependency.** The page loads ethers.js from `esm.sh`. If you want zero external dependencies, download `ethers@6.13.4` and replace the `import` URL with a local path. For a one-off unstake this isn't worth the friction.
- **No data leaves your browser.** No analytics, no telemetry, no server. The only outbound requests are: ethers.js from the CDN, Google Fonts for the display font, and JSON-RPC calls through MetaMask (which uses MetaMask's own RPC, not one you configure here).
- **Start with a small amount.** Always test the flow with a tiny unstake first to confirm everything works as expected.

---

## How it differs from the reference script

| | `kylezs/unstake-stFLIP` Node script | This UI |
|---|---|---|
| Signer | Private key in `.env` | MetaMask |
| RPC | You provide one (Infura/Alchemy) | Whatever MetaMask uses |
| Where FLIP lands | The throwaway script address | The wallet you connected with |
| Setup | `npm install`, edit `.env` | Serve one HTML file |
| Safe to use main wallet? | No (use throwaway) | Yes |
| Liquidity preview | Logged to console | Live in UI |

---

## Disclaimer

Provided as-is, no warranty. You are responsible for verifying contract addresses, transaction parameters, and outcomes. This is not affiliated with the Chainflip team or the author of the reference repo. Always start with a small test amount.

---

## Credits

Built on top of [kylezs/unstake-stFLIP](https://github.com/kylezs/unstake-stFLIP). Contract addresses, ABI fragments, and the `unstakeAggregate(amount, 0, 0, 0)` call pattern come directly from `unstake.mjs` in that repo.
