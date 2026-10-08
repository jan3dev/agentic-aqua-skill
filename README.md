# Agentic AQUA Skill

An agent skill that teaches AI coding agents how to use
[Agentic AQUA](https://github.com/jan3dev/agentic-aqua) — a self-custodial
Bitcoin, Liquid and Lightning wallet driven by the `aqua` CLI or its MCP server.

One BIP39 seed backs both Bitcoin and Liquid. No node, no custodian for keys.

## Installation

Two independent pieces. You need both.

| | What it is | Without it |
|---|---|---|
| **1. The skill** | This repo — markdown that teaches the agent how to drive AQUA | The agent has the wallet but no idea how to use it safely |
| **2. The `aqua` CLI** | The actual wallet ([jan3dev/agentic-aqua](https://github.com/jan3dev/agentic-aqua)) | The agent knows the commands but they do not exist |

### Quickest path — let an agent do both

Paste this into any agent that can run shell commands:

```
Install Agentic AQUA by following https://raw.githubusercontent.com/jan3dev/agentic-aqua-skill/main/setup.md
```

The agent reads [`setup.md`](./setup.md), installs the CLI, optionally registers
the MCP server with its host, then installs this skill locally so the URL is
never needed again.

### 1. Install the skill

```bash
npx skills add jan3dev/agentic-aqua-skill
```

Idempotent — safe to re-run.

**Scope.** Project by default, so the skill only loads inside that repo. Add
`-g` to install for your user and have it available everywhere:

```bash
npx skills add jan3dev/agentic-aqua-skill -g
```

| Scope | Lands in |
|---|---|
| project (default) | `./.claude/skills/` (per-agent equivalent) |
| global (`-g`) | `~/.claude/skills/` (per-agent equivalent) |

**Agent.** The CLI detects what you have installed. Target specific agents with
`-a`, repeatable:

```bash
npx skills add jan3dev/agentic-aqua-skill -a claude-code -a cursor
```

Supported: `claude-code`, `cursor`, `codex`, `opencode`, `cline`, `openhands`,
`roo`.

**Manual install**, if you would rather not run `npx`. Copy the skill directory
into the agent's skills folder:

```bash
git clone https://github.com/jan3dev/agentic-aqua-skill.git
cp -r agentic-aqua-skill/agentic-aqua ~/.claude/skills/
```

Adjust the destination for your agent — Cursor, Codex and OpenCode each use
their own path.

**Verify:**

```bash
npx skills list
```

The skill loads on the agent's next session. Only its `description` stays in
context; the agent opens `SKILL.md` and a single reference file when a request
actually matches.

### 2. Install the `aqua` CLI

Requires **Python 3.13**. `uv` handles that for you:

```bash
# uv, if missing
curl -LsSf https://astral.sh/uv/install.sh | sh          # macOS / Linux
powershell -c "irm https://astral.sh/uv/install.ps1 | iex" # Windows

uv tool install --python 3.13 agentic-aqua
aqua doctor
```

That is enough — the skill is written entirely in CLI terms.

**Optionally** register the MCP server. Resolve the absolute path first, because
MCP hosts do not read your shell PATH:

```bash
which agentic-aqua
claude mcp add agentic-aqua --scope user -- /full/path/to/agentic-aqua
```

Other hosts use their own config with the same `command` + `args` shape. See
[install.md](./agentic-aqua/references/install.md).

### Updating and removing

```bash
npx skills update agentic-aqua      # the skill
npx skills remove agentic-aqua

uv tool upgrade agentic-aqua        # the CLI
uv tool uninstall agentic-aqua
```

Uninstalling the CLI leaves `~/.aqua/` in place — wallets, config and swap
records survive.

### Try it

Ask the agent:

> What's my AQUA wallet balance?

If it reaches for `aqua balance`, both pieces are wired up.

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
