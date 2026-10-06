# Agentic AQUA Skill

An agent skill that teaches AI coding agents how to use
[Agentic AQUA](https://github.com/jan3dev/agentic-aqua) — a self-custodial
Bitcoin, Liquid and Lightning wallet driven by the `aqua` CLI or its MCP server.

One BIP39 seed backs both Bitcoin and Liquid. No node, no custodian for keys.

## Installation

### Install the skill

```bash
npx skills add jan3dev/agentic-aqua-skill
```

Works on Claude Code, Cursor, Codex, OpenCode and any other host the open
`skills` registry supports. Idempotent — safe to re-run.

### Or let an agent bootstrap everything

Paste this into any agent that can run shell commands:

```
Install Agentic AQUA by following https://raw.githubusercontent.com/jan3dev/agentic-aqua-skill/main/setup.md
```

The agent reads [`setup.md`](./setup.md), installs the `aqua` CLI, optionally
registers the MCP server with its host, and then installs this skill locally so
the URL is never needed again.

## Example agent prompts

- "Set up an AQUA wallet for me."
- "What's my total balance across Bitcoin and Liquid?"
- "Send 50,000 sats of L-BTC to this address."
- "Pay this Lightning invoice."
- "Convert 0.01 BTC to L-BTC — peg or swap, whichever is cheaper."
- "Swap 100 USDt on Liquid for USDt on Tron."
- "Buy me the Lightning username `andy` on my JAN3 account."
- "Pay 50,000 ARS to this bank alias with USDT."

## Reference

| Topic | File |
|---|---|
| Install the CLI / MCP server | [install.md](./agentic-aqua/references/install.md) |
| CLI vs MCP, config and env vars | [surfaces.md](./agentic-aqua/references/surfaces.md) |
| Wallets, seeds, passwords, balances | [wallet.md](./agentic-aqua/references/wallet.md) |
| Liquid — L-BTC, USDt, assets | [liquid.md](./agentic-aqua/references/liquid.md) |
| Bitcoin on-chain | [bitcoin.md](./agentic-aqua/references/bitcoin.md) |
| Lightning send, status, refunds | [lightning.md](./agentic-aqua/references/lightning.md) |
| SideSwap — pegs and atomic swaps | [sideswap.md](./agentic-aqua/references/sideswap.md) |
| SideShift — custodial cross-chain | [sideshift.md](./agentic-aqua/references/sideshift.md) |
| Changelly — USDt cross-chain | [changelly.md](./agentic-aqua/references/changelly.md) |
| JAN3 account and Lightning Address | [jan3-account.md](./agentic-aqua/references/jan3-account.md) |
| WapuPay — ARS bank payouts | [wapupay.md](./agentic-aqua/references/wapupay.md) |
| QR codes | [qr.md](./agentic-aqua/references/qr.md) |
| Diagnostics and common errors | [troubleshooting.md](./agentic-aqua/references/troubleshooting.md) |

## License

MIT
