# unstake-stFLIP UI

Unstake stFLIP → FLIP through MetaMask. No private keys on disk.

---

## How to use

### 1. Save the file

Put `unstake-stflip.html` in a folder, like `~/Downloads`.

### 2. Open a terminal in that folder and run:

```bash
python3 -m http.server 8000
```

(or `npx serve .` if you prefer Node)

### 3. Open in your browser:

<http://localhost:8000/unstake-stflip.html>

> ⚠️ **Don't just double-click the HTML file.** Browsers block MetaMask on `file://` pages.

### 4. In the UI:

1. Click **Connect MetaMask**
2. Enter the stFLIP amount (or click "use max")
3. Click **1 · Approve stFLIP** → confirm in MetaMask
4. Click **2 · Unstake** → confirm in MetaMask
5. FLIP arrives in the same wallet

Done.

---

## Requirements

- Chrome, Brave, or Firefox
- [MetaMask](https://metamask.io/download/) installed and unlocked
- ETH for gas + stFLIP to unstake
- Python 3 or Node.js (to serve the file)

---

## Common problems

**MetaMask popup doesn't appear when I click connect.**
Check the environment panel at the top of the page. If `protocol: file:`, go back to step 2 above. If `window.ethereum: undefined`, MetaMask isn't installed or is disabled.

**"Exceeds instant liquidity" warning.**
The contract doesn't currently hold enough FLIP. Try a smaller amount or check again later.

**"Wrong network" warning.**
Switch MetaMask to Ethereum mainnet (chain id 1).

**Transaction reverts.**
Usually means liquidity dropped between checking and signing. Check the Etherscan link in the activity log for details.

---

## What it does

Two transactions, identical to the [reference script](https://github.com/kylezs/unstake-stFLIP):

1. `approve(AGGREGATOR, amount)` on the stFLIP token
2. `unstakeAggregate(amount, 0, 0, 0)` on the aggregator

MetaMask signs both. Your private key never leaves the extension.

### Contract addresses (mainnet)

| Contract | Address |
|----------|---------|
| stFLIP token | `0x961D4921e1718E633BAC8Ded88c4a1cAe44b785a` |
| Aggregator | `0x38d8d03dFA9554D2232D4249EB23c48c23a24fA4` |
| Burner | `0xb4078E779F4a982f27109522E2BA07dd9E133252` |
| Output | `0x6345A9F7e7069D478FFF3595f1522f28d8405151` |
| FLIP token | `0x826180541412D574cf1336d22c0C0a287822678A` |

Verify these against the source repo before signing.

---

## Security

- Private key stays in MetaMask. The page only builds unsigned transactions.
- No analytics, no server, no data leaves your browser.
- The page loads ethers.js from a CDN (`esm.sh`) and fonts from Google Fonts. That's the only external traffic.
- **Start with a small test amount.** Always.

---

## Disclaimer

Provided as-is, no warranty. Not affiliated with Chainflip or the author of the reference repo. You are responsible for verifying everything you sign.

Built on top of [kylezs/unstake-stFLIP](https://github.com/kylezs/unstake-stFLIP).
