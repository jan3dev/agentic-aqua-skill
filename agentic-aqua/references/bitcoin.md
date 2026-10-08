# Bitcoin

On-chain BIP84 (native segwit, `bc1...`) via BDK against Esplora. No local node.

Same seed as the Liquid wallet — see [wallet.md](./wallet.md). Different
derivation path, so the two balances are independent.

## Receive

```bash
aqua btc address --wallet-name default
aqua btc address --wallet-name default --index 5
```

Returns a `bc1...` address. Use a fresh one per payment.

## Balance

```bash
aqua --format json btc balance --wallet-name default
```

Satoshis. For the cross-chain view use `aqua balance`.

## Send

```bash
aqua btc send --wallet-name default \
  --address bc1... \
  --amount 50000 \
  --fee-rate 5 \
  --password-stdin
```

| Flag | Required | Notes |
|---|---|---|
| `--wallet-name` | yes | No default for spending |
| `--address` | yes | `bc1...` |
| `--amount` | yes | Satoshis, integer |
| `--fee-rate` | no | sat/vB. Omit to let BDK estimate. |
| `--password-stdin` | no | Falls back to `AQUA_PASSWORD`, then no password |

Set `--fee-rate` explicitly when the user cares about confirmation time. A low
fee rate on a congested mempool can leave the transaction unconfirmed for days,
and AQUA has no RBF/CPFP command to bump it afterwards. When in doubt, check a
fee estimator and tell the user the trade-off before broadcasting.

## Sweep

```bash
aqua btc sweep --wallet-name default --address bc1... --fee-rate 5 --password-stdin
```

Sends the **entire** balance to one address. The fee is deducted from the
inputs — 0 sats remain. Irreversible. Confirm the full address with the user
first.

## History

```bash
aqua btc transactions --wallet-name default --limit 10
```

## Watch-only descriptors

```bash
aqua btc export-descriptor --wallet-name default
aqua btc import-descriptor --descriptor "wpkh(...)" --wallet-name watch-btc
```

`--change-descriptor` is auto-derived from the external descriptor (`/0/*` →
`/1/*`) when omitted.

This exports **Bitcoin only**. The Liquid CT descriptor is separate and cannot
be derived from this xpub — run `aqua liquid export-descriptor` too. See
[wallet.md](./wallet.md).

## Converting to L-BTC

Do not send BTC to a Liquid address. They are different chains and the funds
are unrecoverable.

To move value between them use a SideSwap peg — see
[sideswap.md](./sideswap.md). `aqua sideswap recommend` tells you whether a peg
or an atomic swap is the better trade for a given amount and direction.

## Before every spend

1. Exact amount in sats.
2. Full `bc1...` address, read back untruncated.
3. Wallet name and fee rate.
4. Explicit yes from the user.

On-chain bitcoin cannot be recalled, and AQUA does not validate that the
address belongs to who the user thinks it does.
