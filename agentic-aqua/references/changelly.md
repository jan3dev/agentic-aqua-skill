# Changelly

USDt cross-chain swaps: **USDt-Liquid ↔ USDt on Ethereum, Tron, BSC, Solana or
Polygon**. Routed through AQUA's Ankara backend proxy.

That is the entire scope. For BTC, L-BTC or any non-USDt asset, use
[sideswap.md](./sideswap.md) or [sideshift.md](./sideshift.md).

For Liquid-only swaps (L-BTC ↔ USDt-Liquid) prefer **SideSwap** — atomic on
Liquid, no counterparty holding the funds.

## Changelly vs SideShift

Both are custodial and both cover USDt across the same five chains. Compare
quotes when the amount is large enough for the spread to matter; otherwise
either is fine.

Say out loud that the provider takes custody mid-flight. The user is making a
counterparty decision.

## Discovery

```bash
aqua changelly currencies
```

Read-only.

## Quote

```bash
aqua changelly quote --external-network tron --direction send --amount-from 100
aqua changelly quote --external-network tron --direction receive --amount-to 100
```

| Flag | Notes |
|---|---|
| `--external-network` | `ethereum` \| `tron` \| `bsc` \| `solana` \| `polygon` |
| `--direction` | `send` = deposit USDt-Liquid. `receive` = deposit USDt on the external chain. Default `send`. |
| `--amount-from` / `--amount-to` | Decimal strings, **not satoshis** |

## Send — USDt-Liquid out to another chain

```bash
aqua changelly send --external-network tron \
  --settle-address T... \
  --amount-from 100 \
  --wallet-name default --password-stdin
```

Quotes, creates the order, and **broadcasts the USDt-Liquid deposit from the
local wallet**. The refund address is set automatically to the wallet's own
Liquid address.

### `--amount-from` vs `--amount-to`

This is the decision the user actually cares about, and they are mutually
exclusive:

| Flag | Meaning | Who pays the fee |
|---|---|---|
| `--amount-from 100` | You send exactly 100 USDt-Liquid | Deducted from what the recipient gets |
| `--amount-to 100` | The recipient receives exactly 100 USDt | Added on top, paid by you |

Ask which one the user means. "Send them 100 USDt" is ambiguous — if they are
paying an invoice for exactly 100, they want `--amount-to`.

`-y/--yes` skips the quote-confirmation prompt. Only pass it after the user has
confirmed in conversation.

**Read `--settle-address` back in full before sending.** It is on a chain AQUA
cannot validate, and USDt sent to a wrong address on Tron or Ethereum is gone.

## Receive — USDt into the Liquid wallet

```bash
aqua changelly receive --external-network tron \
  --external-refund-address T... \
  --amount-from 50 \
  --wallet-name default
```

Variable-rate. Returns a deposit address on the source chain for the external
sender to pay.

**Always set `--external-refund-address`.** Without it, a stuck order requires
manual intervention through Changelly's web UI — AQUA cannot recover it. Ask the
user for a refund address on the source chain before creating the order.

`--amount-from` and `--amount-to` are mutually exclusive here too.

## Status

```bash
aqua changelly status --order-id <id>
```

Save the `order_id` from `send`/`receive` and hand it to the user. It is the
only handle on the order.

## Before every swap

1. Confirm the external network.
2. Decide `--amount-from` vs `--amount-to` with the user — do not pick for them.
3. Read back the full settle address and its chain.
4. On `receive`, get a refund address.
5. Say that Changelly holds the funds mid-flight.
6. Get an explicit yes.
