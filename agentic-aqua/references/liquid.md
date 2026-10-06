# Liquid

L-BTC, USDt and every other Liquid asset. Backed by LWK against Blockstream's
Electrum/Esplora — no local node.

Liquid transactions are **confidential**: amounts and asset types are blinded.
Only parties with the blinding key see them. Confirmations take ~1 minute per
block, and 2 blocks is the usual settlement bar.

## Receive

```bash
aqua liquid address --wallet-name default
aqua liquid address --wallet-name default --index 5   # specific index
```

Returns a confidential address (`lq1...`). Give the user a fresh one per
payment; reusing an address links payments together.

To hand it over visually, see [qr.md](./qr.md).

## Balance

```bash
aqua --format json liquid balance --wallet-name default
```

Returns every asset the wallet holds, keyed by asset id. For a cross-chain view
use `aqua balance` instead.

## The asset registry

```bash
aqua liquid assets                      # mainnet
aqua liquid assets --network testnet
```

Returns `asset_id`, `ticker`, `name` and **`precision`** for each known asset.

**Read `precision` before sending an asset.** Amounts are in base units, so the
number you pass is `human_amount × 10^precision`. USDt on Liquid has precision
8: sending 1 USDt means `--amount 100000000`. Passing `--amount 1` sends
0.00000001 USDt; passing `--amount 100000000` when you meant 1 unit of an
8-decimal asset sends 100 million of them.

Get this wrong and the money is gone. There is no recall.

## Send L-BTC

```bash
aqua liquid send --wallet-name default \
  --address lq1... \
  --amount 50000 \
  --password-stdin
```

`--amount` is **satoshis**, integer, `>= 1`. `--wallet-name` and `--address` are
required — there is no default wallet for spending.

## Send an asset

```bash
aqua liquid send-asset --wallet-name default \
  --address lq1... \
  --amount 100000000 \
  --asset-ticker USDt \
  --password-stdin
```

Identify the asset by `--asset-ticker` (case-insensitive, resolved via the
registry) **or** `--asset-id` (hex). Ticker is friendlier; asset id is
unambiguous. When the user names an asset informally, resolve it with
`aqua liquid assets` first and read the resolved id back to them.

## Sweep

```bash
aqua liquid sweep --wallet-name default --address lq1... --password-stdin
aqua liquid sweep --wallet-name default --address lq1... --asset-ticker USDt --password-stdin
```

Empties the targeted balance entirely. The fee comes **out of the inputs**, so
0 sats of that balance remain after broadcast.

An asset sweep leaves L-BTC change behind (the fee is paid in L-BTC). To fully
empty the wallet: sweep each asset first, then run it once more without
`--asset-id`/`--asset-ticker` to take the remaining L-BTC.

Sweep is the single most destructive command here. Confirm the full destination
address and the exact asset with the user before running it.

## History and status

```bash
aqua liquid transactions --wallet-name default --limit 10
aqua liquid tx-status --tx <txid-or-explorer-url>
```

`--tx` accepts a hex txid or a Liquid explorer URL.

## Before every spend

1. State the exact amount **and its unit** (sats for L-BTC, base units for
   assets — name the asset and its precision).
2. Read back the **full** destination address, never truncated.
3. Name the wallet and the network.
4. Wait for an explicit yes.

A Liquid address is a long confidential string. An agent that abbreviates it
in the confirmation has not confirmed anything.

## Chain backend

By default AQUA uses a built-in list of Blockstream Electrum/Esplora endpoints
with fallback. Setting `electrum_url` in `~/.aqua/config.json` **replaces** that
list with a single endpoint and no fallback, for reads and broadcast alike. If
sends start failing after a config change, that is the first suspect — see
[troubleshooting.md](./troubleshooting.md).
