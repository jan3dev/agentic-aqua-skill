# SideShift

Cross-chain swaps: USDt across Ethereum/Tron/BSC/Solana/Polygon/Liquid, BTC ↔
USDt-on-X, and more.

> **SideShift is custodial.** They take the deposit into their hot wallet and
> send the converted asset from it. The trust model is "trust SideShift the
> company", not "trust an on-chain protocol". Say this to the user before the
> first shift — they are making a counterparty decision, not just a technical
> one.

**If both legs are on Bitcoin or Liquid, use [SideSwap](./sideswap.md)
instead** — atomic on Liquid, or a federation peg. Non-custodial beats custodial
when both are available.

## Which provider?

```bash
aqua sideshift recommend --from-coin USDT --from-network liquid \
  --to-coin USDT --to-network tron
```

Recommends SideSwap vs SideShift for a pair. Run it first.

For USDt-Liquid ↔ USDt on the five big chains, [Changelly](./changelly.md) is
the other option — compare quotes if the amount is large enough to matter.

## The pair allowlist

AQUA ships a curated allowlist mirroring the AQUA mobile app:

- **USDt** on `ethereum`, `tron`, `bsc`, `solana`, `polygon`, `liquid`
- **BTC** on `bitcoin`

Off-allowlist pairs raise an error from `send` and `receive` (`quote`,
`pair-info` and `coins` are read-only and unrestricted).

`SIDESHIFT_ALLOW_ALL_NETWORKS=1` bypasses the allowlist. **Do not set it on your
own.** It is there for users who know exactly which pair they want and why; an
agent reaching for it to make an error go away is routing someone's money onto
an untested path.

## Discovery

```bash
aqua sideshift coins
aqua sideshift pair-info --from-coin USDT --from-network liquid \
  --to-coin USDT --to-network tron --amount 100
```

`pair-info` gives rate, min and max. **Check min and max before quoting** —
below the minimum the shift is rejected, above the maximum it is capped.

## Quote

```bash
aqua sideshift quote --deposit-coin USDT --deposit-network liquid \
  --settle-coin USDT --settle-network tron --deposit-amount 100
```

Fixed-rate, **~15 minute TTL**. Give exactly one of `--deposit-amount` /
`--settle-amount`, as decimal strings (`"100"`, `"0.5"`) — **not satoshis**.

A quote that expires before the deposit lands means a different rate. For
`send`, AQUA quotes and broadcasts in one command, so the window is not usually
a problem — but do not quote, discuss for ten minutes, then send.

## Send

```bash
aqua sideshift send \
  --deposit-coin usdt --deposit-network liquid \
  --settle-coin usdt --settle-network tron \
  --settle-address T... \
  --deposit-amount 100 \
  --wallet-name default --password-stdin
```

Quotes, creates the shift, and **broadcasts the deposit from the local wallet**
in one go.

Deposit-side naming is the part that trips people up:

| What you are sending | `--deposit-coin` | `--deposit-network` |
|---|---|---|
| L-BTC | `btc` | `liquid` |
| BTC mainchain | `btc` | `bitcoin` |
| USDt on Liquid | `usdt` | `liquid` |

`--deposit-network` must be `bitcoin` or `liquid` — it is the chain AQUA signs
on. The settle side can be any SideShift-supported coin and network.

| Flag | Notes |
|---|---|
| `--settle-address` | Required. Where SideShift sends the converted asset. |
| `--deposit-amount` / `--settle-amount` | Exactly one, decimal string |
| `--liquid-asset-id` | Override only. Auto-resolved from `--deposit-coin` against the registry (USDt, DePix, JPYS, EURx, MEX). |
| `--settle-memo` | **Required for memo networks** (BNB and friends). Omitting it on a memo chain can lose the funds. |
| `-y, --yes` | Skips the quote-confirmation prompt |

A refund address is set automatically — the wallet's own address on the deposit
chain.

**Read `--settle-address` back to the user in full before sending.** It is on a
chain AQUA cannot validate, and a wrong address is unrecoverable.

## Receive

```bash
aqua sideshift receive \
  --deposit-coin usdt --deposit-network tron \
  --settle-coin usdt --settle-network liquid \
  --external-refund-address T... \
  --wallet-name default
```

Variable-rate. Returns a deposit address on the source chain for the user or an
external sender to pay from any wallet.

`--settle-network` must be `bitcoin` or `liquid` (AQUA's side).

**Always set `--external-refund-address`.** Without it, a stuck shift requires
manual intervention through SideShift's web UI — AQUA cannot recover it. The
flag is marked "strongly recommended" in the CLI; treat it as required and ask
the user for a refund address on the source chain before creating the order.

## Status

```bash
aqua sideshift status --shift-id <id>
```

Save the `shift_id` from `send`/`receive` and give it to the user. It is the
only handle on the order, and the only thing SideShift support can work from.

## Before every shift

1. Confirm the pair with `pair-info` — rate, min, max.
2. Read back the full `--settle-address` and its network.
3. State the amount **in decimal units**, naming the asset.
4. For memo chains, confirm the memo.
5. Say out loud that SideShift takes custody mid-flight.
6. Get an explicit yes.
