# Surfaces — CLI vs MCP

Agentic AQUA exposes the same capabilities twice: the `aqua` CLI and the MCP
server. They are **1:1** — same functions, same arguments, same gating.

## Which one to use

**Prefer the CLI**, and use the MCP tools when the host already has them loaded.

- The CLI works in any agent that can run a shell. No host config, no restart.
- The MCP tools are self-describing: if they are loaded, their schemas are
  already in your context and you do not need this skill to call them.
- Every command in the other reference files is written as CLI. Translate with
  the table below when you are on MCP.

Do not mix surfaces mid-flow for a single operation. Pick one and stay on it —
both read and write the same `~/.aqua/` state, but interleaving makes the
transcript impossible to audit after a bad spend.

## Tool map

| MCP tool | CLI |
|---|---|
| `unified_balance` | `aqua balance` |
| `doctor` | `aqua doctor` |
| `lw_generate_mnemonic` | `aqua wallet generate-mnemonic` |
| `lw_import_mnemonic` | `aqua wallet import-mnemonic` |
| `lw_list_wallets` | `aqua wallet list` |
| `delete_wallet` | `aqua wallet delete` |
| `lw_balance` | `aqua liquid balance` |
| `lw_address` | `aqua liquid address` |
| `lw_transactions` | `aqua liquid transactions` |
| `lw_tx_status` | `aqua liquid tx-status` |
| `lw_send` | `aqua liquid send` |
| `lw_send_asset` | `aqua liquid send-asset` |
| `lw_sweep` | `aqua liquid sweep` |
| `lw_list_assets` | `aqua liquid assets` |
| `lw_export_descriptor` | `aqua liquid export-descriptor` |
| `lw_import_descriptor` | `aqua liquid import-descriptor` |
| `btc_balance` | `aqua btc balance` |
| `btc_address` | `aqua btc address` |
| `btc_transactions` | `aqua btc transactions` |
| `btc_send` | `aqua btc send` |
| `btc_sweep` | `aqua btc sweep` |
| `btc_export_descriptor` | `aqua btc export-descriptor` |
| `btc_import_descriptor` | `aqua btc import-descriptor` |
| `lightning_send` | `aqua lightning send` |
| `lightning_decode` | `aqua lightning decode` |
| `lightning_transaction_status` | `aqua lightning status` |
| `lightning_refund` | `aqua lightning refund` |
| `lightning_receive` | `aqua lightning receive` — **ships disabled** |
| `sideswap_server_status` | `aqua sideswap status` |
| `sideswap_recommend` | `aqua sideswap recommend` |
| `sideswap_peg_quote` | `aqua sideswap peg-quote` |
| `sideswap_peg_in` | `aqua sideswap peg-in` |
| `sideswap_peg_out` | `aqua sideswap peg-out` |
| `sideswap_peg_status` | `aqua sideswap peg-status` |
| `sideswap_list_assets` | `aqua sideswap assets` |
| `sideswap_quote` | `aqua sideswap quote` |
| `sideswap_execute_swap` | `aqua sideswap swap` |
| `sideswap_swap_status` | `aqua sideswap swap-status` |
| `sideshift_list_coins` | `aqua sideshift coins` |
| `sideshift_pair_info` | `aqua sideshift pair-info` |
| `sideshift_recommend` | `aqua sideshift recommend` |
| `sideshift_quote` | `aqua sideshift quote` |
| `sideshift_send` | `aqua sideshift send` |
| `sideshift_receive` | `aqua sideshift receive` |
| `sideshift_status` | `aqua sideshift status` |
| `changelly_list_currencies` | `aqua changelly currencies` |
| `changelly_quote` | `aqua changelly quote` |
| `changelly_send` | `aqua changelly send` |
| `changelly_receive` | `aqua changelly receive` |
| `changelly_status` | `aqua changelly status` |
| `jan3_login` | `aqua jan3 login` |
| `jan3_verify` | `aqua jan3 verify` |
| `jan3_login_start` | `aqua jan3 login-start` |
| `jan3_login_complete` | `aqua jan3 login-complete` |
| `jan3_user_info` | `aqua jan3 user-info` |
| `jan3_ln_check_username` | `aqua jan3 ln-check-username` |
| `jan3_purchase_ln_username` | `aqua jan3 purchase-ln-username` |
| `jan3_enable_lightning_address` | `aqua jan3 enable-lightning-address` |
| `jan3_rebind_wallet` | `aqua jan3 rebind-wallet` |
| `jan3_list_sessions` | `aqua jan3 list-sessions` |
| `jan3_session_info` | `aqua jan3 session-info` |
| `jan3_logout` | `aqua jan3 logout` |
| `wapupay_provision_account` | `aqua wapupay provision-account` |
| `wapupay_exchange_rates` | `aqua wapupay rates` |
| `wapupay_quote` | `aqua wapupay quote` |
| `wapupay_create_order` | `aqua wapupay create-order` |
| `wapupay_fund_order` | `aqua wapupay fund-order` |
| `wapupay_order_status` | `aqua wapupay order-status` |
| `wapupay_orders` | `aqua wapupay orders` |
| `wapupay_transactions` | `aqua wapupay transactions` |
| `wapupay_transaction` | `aqua wapupay transaction` |
| `wapupay_spending_limit` | `aqua wapupay spending-limit` |
| `qr_generate` | `aqua qr generate` |
| `qr_decode` | `aqua qr decode` |

CLI flags are kebab-case (`--wallet-name`); MCP arguments are the snake_case
equivalent (`wallet_name`).

## Config file

`~/.aqua/config.json`, created on first run.

```json
{
  "network": "mainnet",
  "default_wallet": "default",
  "electrum_url": null,
  "auto_sync": true,
  "lightning_provider": "indra",
  "enabled_tools": {
    "unified_balance": true,
    "lw_balance": true,
    "lightning_send": true
  }
}
```

| Field | Default | Meaning |
|---|---|---|
| `network` | `"mainnet"` | `"mainnet"` or `"testnet"` |
| `default_wallet` | `"default"` | Used when `--wallet-name` is omitted |
| `electrum_url` | `null` | Pins the Liquid chain backend. A value **replaces** the built-in list — single backend, no fallback, affects reads *and* broadcast. Liquid only. |
| `auto_sync` | `true` | Sync the wallet on every balance/address call |
| `lightning_provider` | `"indra"` | Lightning send backend: `"indra"` or `"boltz"` |
| `enabled_tools` | all on except `lightning_receive` | Per-tool on/off switches |

## Feature flags

`enabled_tools` gates **both** surfaces. A disabled tool is hidden from the MCP
`list_tools` response *and* its CLI command is not registered. If a documented
command is missing from `aqua --help`, that is the first thing to check.

`lightning_receive` ships **disabled**. To turn it on, set
`enabled_tools.lightning_receive` to `true` in `~/.aqua/config.json`. Tell the
user what they are enabling before editing their config, and never edit it
without being asked.

## Lightning provider

L-BTC → Lightning goes through a Boltz-v2 submarine swap. Two backends:

| Value | Service | Limits (sats) | Networks |
|---|---|---|---|
| `"indra"` (default) | `https://indra.aquabtc.com` (AQUA) | 1,000 – 100,000 | mainnet only |
| `"boltz"` | `https://api.boltz.exchange` | 100 – 25,000,000 | mainnet + testnet |

Those limits are client-side guards; the live pair is the authority and is
re-checked on every payment. **Testnet has no Indra endpoint** — `aqua` raises
instead of falling back, so testnet work needs `"boltz"`.

Override for a single run without touching the config:

```bash
AQUA_LIGHTNING_PROVIDER=boltz aqua lightning send --invoice lnbc...
```

A swap already on disk is always queried against the provider it was created
with. Switching providers does not orphan an in-flight swap.

## Environment variables

| Variable | Used by |
|---|---|
| `AQUA_PASSWORD` | Fallback for `--password-stdin` on every spending command |
| `AQUA_MNEMONIC` | Fallback for `--mnemonic-stdin` on `wallet import-mnemonic` |
| `AQUA_LIGHTNING_PROVIDER` | Overrides `lightning_provider` for one run |
| `INDRA_API_URL` | Points Indra at a different host (read at import time) |
| `SIDESHIFT_ALLOW_ALL_NETWORKS` | `=1` bypasses the curated SideShift pair allowlist |
| `WAPUPAY_API_KEY` | WapuPay auth, if not provisioned via JAN3 |

The root CLI group uses `auto_envvar_prefix="AQUA"`, so Click options also read
`AQUA_<COMMAND>_<OPTION>`-style variables.

**Never echo these.** See the Security section in [SKILL.md](../SKILL.md).

## Output format

```bash
aqua --format json balance
```

Pretty on a terminal, JSON when piped. Always pass `--format json` when you are
going to parse the result — the pretty renderer is for humans and its layout is
not a contract.
