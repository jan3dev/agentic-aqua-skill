# WapuPay

Pay **Argentine bank accounts in ARS**, funded with USDT or L-BTC on Liquid.

The flow is deliberately split: AQUA creates an order and returns a Liquid
funding address. **AQUA never auto-funds it.** You send the funds in a separate,
explicit step, and WapuPay then settles the pesos.

That split is a safety feature. Do not try to collapse it into one action.

## Authentication

WapuPay needs an API key. Two ways to get one:

```bash
aqua wapupay provision-account --email user@example.com
```

Provisions the key through the user's JAN3 account and stores it under
`~/.aqua/wapupay/`. Requires an active JAN3 session for that email first — see
[jan3-account.md](./jan3-account.md).

Or set `WAPUPAY_API_KEY` in the environment. Never echo it, and never dump the
environment where it could land in the transcript.

## Orientation

```bash
aqua wapupay about
aqua wapupay rates
aqua wapupay spending-limit
```

`about` explains the service. `rates` shows current exchange rates (USDT/ARS).
`spending-limit` shows the **monthly** limit in USDT — check it before quoting a
large payment, because an order over the limit will fail after the user has
already committed mentally.

## Transfer speed

Every quote and order takes `--type`:

| Value | Shown to the user as | Timing | Fee |
|---|---|---|---|
| `fast_fiat_transfer` (default) | **Fast Transfer** | ~10 min – 1 h, daytime | Higher |
| `fiat_transfer` | **Normal Transfer** | 3 – 12 h | Lower |

Use the user-facing names when you talk about them. Offer the choice — the fee
difference is real and the user may not be in a hurry.

## Quote

```bash
aqua wapupay quote --amount-ars 50000 --type fast_fiat_transfer --alias juan.perez.mp
```

Previews USDT cost, fee and rate. **Creates no order.** Passing `--alias`
enables recipient validation — always pass it when you have it, so a bad alias
fails here instead of after the money moves.

`--amount-ars` is a decimal string (`"50000"`).

## Create the order

```bash
aqua wapupay create-order \
  --amount-ars 50000 \
  --alias juan.perez.mp \
  --type fast_fiat_transfer \
  --receiver-name "Juan Perez" \
  --funding-method USDT \
  --refund-address lq1... \
  --wallet-name default
```

| Flag | Required | Notes |
|---|---|---|
| `--amount-ars` | yes | Decimal string |
| `--alias` | yes | Recipient bank alias, CBU or CVU |
| `--type` | no | Default `fast_fiat_transfer` |
| `--receiver-name` | no | |
| `--refund-address` | no | Liquid mainnet address (`lq1…`/`ex1…`) used if funding cannot execute |
| `--wallet-name` | no | The wallet you intend to fund from |
| `--funding-method` | no | `USDT` (default) or `LBTC` |
| `-y, --yes` | no | Skips the quote-confirmation prompt |

Returns a **`tentative_id`** and a Liquid funding address. **This command never
broadcasts a payment.**

Always pass `--refund-address`. It is optional in the CLI, but without it a
failed funding has nowhere to go back to.

Only pass `-y/--yes` after the user has seen a quote and agreed to it.

## Fund the order

Separate, explicit step. Read the funding address and the exact amount from the
`create-order` output.

```bash
# USDT funding — amount in USDT base units
aqua liquid send-asset --wallet-name default \
  --address <funding_address> --amount <amount> --asset-ticker USDt --password-stdin

# L-BTC funding — amount in sats (total_amount_sats from the order)
aqua liquid send --wallet-name default \
  --address <funding_address> --amount <total_amount_sats> --password-stdin
```

**The unit depends on `--funding-method`.** `LBTC` orders are paid in L-BTC sats
(`total_amount_sats`); `USDT` orders in USDT base units. Mixing them up sends
wildly the wrong amount. Re-read [liquid.md](./liquid.md) on asset precision if
you are not certain.

Underpaying or overpaying an order can leave it stuck. Send the exact amount the
order asked for.

To re-issue funding instructions for an existing order:

```bash
aqua wapupay fund-order --tentative-id <id>
```

## Track

```bash
aqua wapupay order-status --tentative-id <id>    # re-read from WapuPay
aqua wapupay orders                              # locally tracked orders
aqua wapupay transactions
aqua wapupay transaction --id <uuid-or-numeric>
```

Save the `tentative_id` and give it to the user. It is the only handle on the
order.

## Before funding an order

This is the highest-consequence flow in AQUA — real pesos to a real bank
account, irreversible once funded.

1. `aqua wapupay quote` with `--alias` and relay the full quote.
2. Read back the **recipient alias/CBU/CVU** and the receiver name.
3. State the ARS amount **and** the USDT or L-BTC cost.
4. State the transfer type and its expected timing.
5. Confirm the funding amount and unit match the order exactly.
6. Get an explicit yes **for this specific order**.

A yes on the quote is not a yes on funding. Ask again.
