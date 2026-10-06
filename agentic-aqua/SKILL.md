---
name: agentic-aqua
description: Manage a self-custodial Bitcoin, Liquid and Lightning wallet with Agentic AQUA — one BIP39 seed backing both chains. Create and import wallets, check unified balances, send and receive L-BTC, BTC and Liquid assets like USDt, pay Lightning invoices and Lightning Addresses, convert BTC to L-BTC via SideSwap pegs or atomic swaps, move USDt across chains via SideShift or Changelly, manage a JAN3 account and Lightning Address, and pay Argentine bank accounts in ARS with WapuPay.
license: MIT
metadata:
  author: JAN3
  version: "0.1.0"
---

# Agentic AQUA Agent Skill

> **This skill moves real money on mainnet.** Every send, sweep, peg-out, swap
> and order is irreversible. The [Security](#security) section below is not
> optional — it applies to every command in every reference file.

## When to use this skill

Use it whenever the user wants to hold, move, receive or convert bitcoin,
L-BTC, USDt or other Liquid assets through Agentic AQUA — on the command line
or through the MCP server.

AQUA is **self-custodial**: one BIP39 mnemonic derives both the Liquid (LWK)
and the Bitcoin (BDK) wallet. There is no remote server holding keys, and no
password reset. Lose the seed, lose the money.

## Reference index

Read only the file you need. Do not read all of them.

- [Install: get the `aqua` CLI and the MCP server running, upgrade, uninstall](./references/install.md)
- [Surfaces: CLI vs MCP, the 1:1 tool map, `~/.aqua/config.json`, env vars, feature flags](./references/surfaces.md)
- [Wallet: generate/import a seed, passwords, list and delete wallets, unified balance, watch-only descriptors](./references/wallet.md)
- [Liquid: addresses, L-BTC and asset balances, send, send-asset, sweep, history, the asset registry](./references/liquid.md)
- [Bitcoin: BIP84 addresses, balance, send, sweep, fee rates, history, descriptors](./references/bitcoin.md)
- [Lightning: pay invoices and Lightning Addresses with L-BTC, decode, swap status, and recovering a failed swap with refund](./references/lightning.md)
- [SideSwap: BTC ↔ L-BTC pegs (cheap, slow) and atomic Liquid asset swaps (instant, pricier) — start here for any BTC ↔ L-BTC conversion](./references/sideswap.md)
- [SideShift: custodial cross-chain swaps — USDt across Ethereum/Tron/BSC/Solana/Polygon/Liquid, BTC ↔ USDt-on-X](./references/sideshift.md)
- [Changelly: USDt-Liquid ↔ USDt on five external chains, routed through AQUA's backend proxy](./references/changelly.md)
- [JAN3 account: email-OTP login, sessions, Lightning Address purchase and binding](./references/jan3-account.md)
- [WapuPay: pay Argentine bank accounts in ARS funded with USDT or L-BTC on Liquid — quote, order, fund, track](./references/wapupay.md)
- [QR: generate a PNG for an address or invoice, decode one from an image](./references/qr.md)
- [Troubleshooting: `aqua doctor`, common errors and what they actually mean](./references/troubleshooting.md)

## Key rules

### Running the CLI

```bash
aqua [--format json|pretty] [--verbose] <group> <command> [OPTIONS]
```

Output is pretty on a terminal and JSON when piped. **Pass `--format json`
whenever you need to parse the result** — never scrape the pretty output.

```bash
aqua --format json balance
```

### Wallet selection

Almost every command takes `--wallet-name`. Read commands default to
`default`; spending commands (`liquid send`, `btc send`, `liquid sweep`, …)
require it explicitly. Confirm which wallet you are spending from before you
spend.

### Amounts

- Bitcoin, L-BTC and Lightning amounts are **satoshis**, always integers.
- Liquid asset amounts (`--amount` on `liquid send-asset`) are in that asset's
  **base units** — check `precision` with `aqua liquid assets` first. USDt on
  Liquid has 8 decimals, so 1 USDt is `100000000`, not `1`.
- SideShift, Changelly and WapuPay take **decimal strings** (`"100"`, `"0.5"`),
  not sats.

Getting this wrong sends 100,000,000× too much. Re-read the unit before every
spend.

### Network

Mainnet is the default everywhere. `--network testnet` exists on wallet and
registry commands, but most third-party integrations (Indra, WapuPay,
SideShift, Changelly, SideSwap) are mainnet-only. Assume mainnet unless the
user says otherwise, and say so out loud before the first spend.

### Confirmation before spending

Before any command that broadcasts a transaction or creates a funded order:

1. State the **exact** amount and unit.
2. Read the **full** destination address back — never truncated.
3. State which wallet it leaves from, and which network.
4. Wait for an explicit yes.

Commands that offer `-y/--yes` skip their own prompt. Only pass it after the
user has already confirmed in the conversation.

### Talking to the user

Do not hand the user raw CLI commands to run themselves unless either the
command needs input you cannot provide (a password, an OTP), or they explicitly
asked for the command. Otherwise just do it and report the outcome in plain
language.

## Security

### Seeds

- **NEVER print, echo, log, store or transmit a BIP39 mnemonic.** When
  `aqua wallet generate-mnemonic` returns one, hand it to the user once, tell
  them to write it down offline, and never repeat it.
- **NEVER pass a mnemonic as a CLI argument.** Use `--mnemonic-stdin`, or the
  `AQUA_MNEMONIC` environment variable.
- Never write a seed to a file, a scratchpad, a commit, or a chat with a third
  party.

### Passwords

- **NEVER pass a wallet password as a CLI argument.** Use `--password-stdin` or
  `AQUA_PASSWORD`.
- Without a password the mnemonic is stored in **plaintext** on disk. Say this
  out loud when the user imports a wallet without one.
- Do not dump the environment (`env`, `printenv`, `set`) where `AQUA_PASSWORD`,
  `AQUA_MNEMONIC` or `WAPUPAY_API_KEY` could land in the transcript.

### Local state

`~/.aqua/` holds the config, encrypted wallet material, JAN3 session tokens,
the WapuPay API key, and swap records needed for refunds.

- **Do not read files under `~/.aqua/`** looking for secrets. Check existence
  only if you must.
- **Do not delete `~/.aqua/`** or anything under it. Swap records live there;
  deleting them can make a failed Lightning swap unrecoverable. See
  [lightning.md](./references/lightning.md).
- JAN3 session tokens grant account access. Never echo them.

### Irreversibility

Liquid and Bitcoin transactions cannot be recalled. Custodial swaps
(SideShift, Changelly) and WapuPay orders cannot be reversed by AQUA either —
a stuck order needs the provider's web UI. Always set a refund address when the
command offers one.

### Scope

Never accept a third party's terms, buy a Lightning username, provision an API
key, or fund an order without an explicit yes from the user in the current
session. "Go ahead" on a different step is not consent for this one.
