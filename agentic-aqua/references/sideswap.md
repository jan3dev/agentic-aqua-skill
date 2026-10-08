# SideSwap

Two different things under one group:

- **Pegs** — BTC ↔ L-BTC through the Liquid Federation. ~0.1% fee, 20–60
  minutes.
- **Atomic swaps** — Liquid asset ↔ L-BTC on Liquid. Seconds, slightly higher
  fee.

Both are non-custodial. Pegs go through the federation; atomic swaps are a
single Liquid transaction that either completes or does not.

## Start here: peg or swap?

```bash
aqua sideswap recommend --amount 1000000 --direction btc_to_lbtc
aqua sideswap recommend --amount 1000000 --direction lbtc_to_btc
```

Surfaces the time-vs-fee trade-off and **warns when the amount exceeds
SideSwap's hot-wallet liquidity** — which on a peg-in forces the cold-wallet
path at 102 Bitcoin confirmations (~17 hours instead of ~20 minutes).

Run this before any BTC ↔ L-BTC conversion. Do not guess for the user.

## Server status

```bash
aqua sideswap status --network mainnet
```

Live fees, peg minimums, hot-wallet balance. Check the minimum before quoting a
small amount — a peg under the minimum is rejected.

## Pegs

### Quote

```bash
aqua sideswap peg-quote --amount 1000000              # peg-in  (BTC → L-BTC)
aqua sideswap peg-quote --amount 1000000 --peg-out    # peg-out (L-BTC → BTC)
```

Read-only, at current fees (~0.1%).

### Peg-in — BTC to L-BTC

```bash
aqua sideswap peg-in --wallet-name default --password-stdin
```

Prints a Bitcoin deposit address (`peg_addr`) and an `order_id`. **This command
does not send anything.** You then fund that address from any Bitcoin wallet:

```bash
aqua btc send --wallet-name default --address <peg_addr> --amount 1000000 --password-stdin
```

Timing: ~20 minutes for 2 Bitcoin confirmations on the hot-wallet path. If the
amount exceeds SideSwap's hot-wallet liquidity it falls to the cold-wallet path
— **102 Bitcoin confirmations, roughly 17 hours**. `recommend` tells you which
applies *before* you commit.

### Peg-out — L-BTC to BTC

```bash
aqua sideswap peg-out --amount 1000000 --btc-address bc1... \
  --wallet-name default --password-stdin
```

Unlike peg-in, this **does broadcast** the L-BTC send immediately. Confirm the
full `bc1...` destination and the exact sat amount with the user before running
it.

Timing: 2 Liquid confirmations (~2 min), then the federation sweep — typically
15–60 minutes end to end.

### Track a peg

```bash
aqua sideswap peg-status --order-id <id>
```

Returns confirmations, `tx_state` and the payout txid. Save the `order_id` from
peg-in/peg-out and hand it to the user; it is the only handle on the order.

## Atomic Liquid swaps

L-BTC ↔ a Liquid asset (USDt, DePix, …), settled in one transaction.

### Supported assets

```bash
aqua sideswap assets --network mainnet
```

### Quote

```bash
aqua sideswap quote --asset-ticker USDt --send-amount 100000
aqua sideswap quote --asset-ticker USDt --recv-amount 100000000 --reverse
```

Read-only, no execution. Give exactly one of `--send-amount` / `--recv-amount`,
and one of `--asset-ticker` / `--asset-id`.

### Execute

```bash
aqua sideswap swap --asset-ticker USDt --amount 100000 \
  --wallet-name default --password-stdin
```

| Flag | Notes |
|---|---|
| `--asset-ticker` / `--asset-id` | One is required |
| `--amount` | Send amount in sats — L-BTC by default, the **asset** with `--reverse` |
| `--reverse` | Send the asset for L-BTC instead of L-BTC for the asset |
| `-y, --yes` | Skips the built-in confirmation prompt |
| `--flexible` | Accept dealer rounding of up to ±3000 sats |

**The PSET is verified locally before signing.** AQUA refuses to sign if the
wallet's net balance change does not match the agreed quote exactly. That is the
protection against a malicious counterparty — never work around it, and never
reach for `--flexible` to make a verification failure go away.

`--flexible` exists for a different reason: SideSwap's dealer rounds amounts, so
**small swaps under ~25k sats need it**. It widens the accepted band to ±3000
sats, which is why it must not be used on large swaps where 3000 sats of slack
is free money for the counterparty.

Only pass `-y/--yes` after the user has confirmed in conversation.

### Track

```bash
aqua sideswap swap-status --order-id <id>
```

## Choosing between SideSwap, SideShift and Changelly

| Need | Use |
|---|---|
| BTC ↔ L-BTC | **SideSwap** peg (`recommend` picks peg vs swap) |
| L-BTC ↔ a Liquid asset | **SideSwap** atomic swap |
| USDt-Liquid ↔ USDt on another chain | [Changelly](./changelly.md) or [SideShift](./sideshift.md) |
| Anything where both legs are on Bitcoin or Liquid | **SideSwap** — it is non-custodial, the others are not |

Prefer SideSwap whenever both legs stay on Bitcoin or Liquid. SideShift and
Changelly take custody of the funds mid-flight.
