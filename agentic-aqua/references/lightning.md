# Lightning

AQUA has no Lightning node. A Lightning **send** is a Boltz-v2 submarine swap:
AQUA locks L-BTC into a taproot output, the provider pays the invoice off that
lockup, then claims the locked coins.

That architecture is the whole reason this file is longer than the command list
— when the invoice fails to pay, **your L-BTC stays locked on chain** and you
have to refund it yourself.

## Providers

Set by `lightning_provider` in `~/.aqua/config.json`:

| Value | Service | Limits (sats) | Networks |
|---|---|---|---|
| `"indra"` (default) | `https://indra.aquabtc.com` (AQUA) | 1,000 – 100,000 | mainnet only |
| `"boltz"` | `https://api.boltz.exchange` | 100 – 25,000,000 | mainnet + testnet |

Those limits are client-side guards; the live pair is authoritative and
re-checked on every payment. **Testnet has no Indra endpoint** — AQUA raises
rather than falling back, so testnet needs `boltz`.

Override for one run:

```bash
AQUA_LIGHTNING_PROVIDER=boltz aqua lightning send --invoice lnbc...
```

A swap already on disk is always queried against the provider that created it,
so switching does not orphan an in-flight swap.

## Decode before paying

```bash
aqua lightning decode --invoice lnbc...
```

Shows amount, description and expiry without paying. **Always decode first** and
read the amount and description back to the user. A BOLT11 string is opaque —
neither you nor the user can tell what it is worth by looking at it.

## Send

```bash
# BOLT11 invoice
aqua lightning send --invoice lnbc... --wallet-name default --password-stdin

# zero-amount invoice: supply the amount
aqua lightning send --invoice lnbc... --amount-sats 25000 --wallet-name default --password-stdin

# Lightning Address
aqua lightning send --ln-address user@domain.com --amount-sats 25000 \
  --wallet-name default --password-stdin
```

| Flag | Notes |
|---|---|
| `--invoice` | BOLT11. Mutually exclusive with `--ln-address`. |
| `--ln-address` | `user@domain.com`. **Requires** `--amount-sats`. |
| `--amount-sats` | Required with `--ln-address`. With `--invoice` it is optional, and must match the encoded amount if supplied. |
| `--wallet-name` | Liquid wallet to pay from. Default `default`. |
| `--password-stdin` | Falls back to `AQUA_PASSWORD`, then no password |

The wallet pays in **L-BTC**, not BTC. A Bitcoin on-chain balance cannot fund a
Lightning send — peg it to L-BTC first, see [sideswap.md](./sideswap.md).

The command returns a **`swap_id`**. Record it and give it to the user. Without
it a failed swap is much harder to recover.

## Receive

`aqua lightning receive` exists but **ships disabled**. If the command is
missing from `aqua lightning --help`, that is why.

Enabling it means setting `enabled_tools.lightning_receive` to `true` in
`~/.aqua/config.json`. Explain what you are turning on before editing the
user's config, and do not edit it unasked. See [surfaces.md](./surfaces.md).

For inbound Lightning without that flag, a JAN3 Lightning Address delivers
straight to Liquid addresses — see [jan3-account.md](./jan3-account.md).

## Status

```bash
aqua lightning status --swap-id <id>
```

Works for both send and receive swaps. Watch for `refund_info.refundable` —
that means L-BTC is locked and waiting for you to claim it back.

## Refund a failed send

When the invoice cannot be paid (no route, destination rejects it), the swap
ends in `invoice.failedToPay` and **the L-BTC stays locked on chain**. It is not
lost, but nothing returns it automatically.

```bash
aqua lightning refund --swap-id <id>
aqua lightning refund --swap-id <id> --dry-run    # build and sign, do not broadcast
```

| Flag | Notes |
|---|---|
| `--swap-id` | Required |
| `--address` | Liquid destination. Default: a fresh address of the swap's own wallet. |
| `--dry-run` | Builds and signs without broadcasting |
| `--claim-public-key` | Legacy swaps only |
| `--blinding-key` | Legacy swaps only |

`--dry-run` still **consumes** an address from the wallet's pool when
`--address` is omitted — the wallet cannot hand out a preview address and take
it back. Harmless, but do not be surprised by the gap in indices.

### Why a refund can be refused today

The lockup output has two spend paths:

| Path | When it works | What signs |
|---|---|---|
| **Cooperative** (key path) | Immediately, while the provider cosigns | MuSig2 over claim + refund keys |
| **Unilateral** (script path) | Only after `timeoutBlockHeight` | The refund key alone |

AQUA tries cooperative first — it is cheap and needs no waiting. If the provider
has switched cooperative refunds off (HTTP 400), the only route left is
unilateral, and `OP_CHECKLOCKTIMEVERIFY` gates it until the chain reaches the
swap's timeout block. The error names that block and how far away it is; Liquid
produces roughly one block per minute, so convert it to minutes for the user and
tell them to retry then.

**This is a wait, not a loss.** Do not tell the user the funds are gone, and do
not retry in a loop — retry once the named block has passed.

### Swap records are the recovery material

A refund is rebuilt entirely from `~/.aqua/lightning_swaps/<id>.json`. There is
no server-side recovery. That file holds the locally generated refund private
key, the timeout block height, the provider's claim public key and blinding key,
and the lockup txid.

**Deleting `~/.aqua/lightning_swaps/` can make locked funds unrecoverable.**
Never clean up that directory, and never suggest it as a fix for anything.

### Legacy swaps

Records written before those fields were persisted hold only the refund key and
the timeout. For those, ask the provider for the swap's `claimPublicKey` and
blinding key and pass them explicitly:

```bash
aqua lightning refund --swap-id <id> \
  --claim-public-key <hex> \
  --blinding-key <hex>
```

## Before every send

1. `aqua lightning decode` and read the amount and description back.
2. Confirm the wallet has enough **L-BTC** (not BTC).
3. Confirm the amount is inside the provider's limits.
4. Get an explicit yes.
5. Save the returned `swap_id`.
