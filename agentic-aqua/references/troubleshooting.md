# Troubleshooting

## Start here

```bash
aqua doctor
aqua doctor --fix
```

Diagnoses `~/.aqua/config.json` and, with `--fix`, repairs it. Exit code 0 means
healthy or fully repaired, 1 means issues remain.

`--fix` only touches the config file. It does not touch wallets, seeds or swap
records.

## A documented command does not exist

`aqua <group> --help` does not list it.

Three causes, in order of likelihood:

1. **Feature flag off.** `enabled_tools` in `~/.aqua/config.json` gates both the
   MCP tool and the CLI command. `lightning_receive` ships disabled — that is by
   design, not a bug. See [surfaces.md](./surfaces.md).
2. **Old version.** `aqua --version`, then `uv tool upgrade agentic-aqua`.
3. **Wrong binary on PATH.** `which aqua`.

## MCP host shows no AQUA tools

- The config needs the **absolute** path. MCP hosts do not read your shell PATH.
  Get it with `which agentic-aqua`.
- Restart the host after editing its config.
- Fewer tools than expected → `enabled_tools`, same as above.

See [install.md](./install.md).

## Password errors

A wallet imported with a password needs it on every spend.

- `--password-stdin`, or `AQUA_PASSWORD`, or the interactive prompt.
- **Never** as a CLI argument.
- There is no password reset. The 12-word mnemonic is the only recovery path —
  re-import with `aqua wallet import-mnemonic` under a new wallet name.

If a command fails with no password set, the wallet was imported encrypted.

## Liquid sends failing or balances stale

`electrum_url` in `~/.aqua/config.json` **replaces** the built-in endpoint list
with a single backend and **no fallback**, for reads and broadcast alike. If
things broke right after a config change, set it back to `null` and retry.

`auto_sync: false` means balances and addresses are not refreshed on each call.

Liquid only — Bitcoin keeps its own endpoint list.

## Lightning send failed and the L-BTC is gone

It is not gone, it is locked. A failed submarine swap leaves L-BTC in an on-chain
lockup that **nothing returns automatically**.

```bash
aqua lightning status --swap-id <id>     # look for refund_info.refundable
aqua lightning refund --swap-id <id>
```

If the refund is refused, the provider has disabled cooperative refunds and the
unilateral path is time-locked until the swap's `timeoutBlockHeight`. The error
names that block. Liquid is roughly one block per minute — convert it, tell the
user when to retry, and retry once. Do not loop.

Full detail in [lightning.md](./lightning.md).

**Never delete `~/.aqua/lightning_swaps/`.** Those records are the only recovery
material for a locked swap.

## Lightning limits rejected the payment

| Provider | Limits (sats) | Networks |
|---|---|---|
| `indra` (default) | 1,000 – 100,000 | mainnet only |
| `boltz` | 100 – 25,000,000 | mainnet + testnet |

Above 100,000 sats, switch provider for the run:

```bash
AQUA_LIGHTNING_PROVIDER=boltz aqua lightning send --invoice lnbc...
```

Testnet has no Indra endpoint — AQUA raises rather than falling back, so testnet
requires `boltz`.

## SideShift rejects the pair

AQUA ships a curated allowlist: USDt on ethereum/tron/bsc/solana/polygon/liquid,
and BTC on bitcoin. Off-allowlist pairs are refused by `send` and `receive`.

`SIDESHIFT_ALLOW_ALL_NETWORKS=1` bypasses it. **Do not set it yourself** — ask
the user, and only if they understand they are leaving the tested path. See
[sideshift.md](./sideshift.md).

## SideSwap refuses to sign a swap

That is the safety check working: AQUA verifies the PSET locally and refuses if
the wallet's net balance change does not match the agreed quote.

For small swaps (under ~25k sats) the cause is usually SideSwap's dealer
rounding — pass `--flexible` to accept adjustments up to ±3000 sats.

For anything larger, **do not** reach for `--flexible`. Re-quote and look at why
the numbers moved.

## A peg is taking hours

Peg-in falls back to the cold-wallet path when the amount exceeds SideSwap's
hot-wallet liquidity: 102 Bitcoin confirmations, roughly 17 hours, instead of
~20 minutes.

```bash
aqua sideswap recommend --amount <sats> --direction btc_to_lbtc   # before committing
aqua sideswap peg-status --order-id <id>                          # after
```

## A custodial swap is stuck

SideShift and Changelly orders cannot be recovered by AQUA. A stuck order needs
the provider's web UI, and the refund address set at creation time.

```bash
aqua sideshift status --shift-id <id>
aqua changelly status --order-id <id>
```

This is why `--external-refund-address` is effectively mandatory. See
[sideshift.md](./sideshift.md) and [changelly.md](./changelly.md).

## Parsing output is unreliable

Use `--format json`:

```bash
aqua --format json balance
```

Pretty output is for humans and its layout is not a contract.

## What never to do as a fix

- Delete `~/.aqua/` or anything under it.
- Read files under `~/.aqua/` looking for keys or tokens.
- Set `SIDESHIFT_ALLOW_ALL_NETWORKS=1` on your own judgment.
- Pass `--flexible` to make a PSET verification failure go away.
- Retry a spending command in a loop after an ambiguous failure — check status
  first. A command that failed may still have broadcast.
