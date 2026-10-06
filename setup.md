# Agentic AQUA — Agent Bootstrap

You are an AI agent. The user pointed you at this file to install Agentic AQUA.
Follow these steps in order, then stop.

**This file is a one-time bootstrap.** Step 4 installs the full skill locally so
you never need to fetch this URL again.

Agentic AQUA is a self-custodial wallet. It holds real money. Read
[Security rules](#security-rules) at the bottom before running anything that
spends.

---

## Step 1 — Check whether the CLI is already installed

```bash
which aqua || command -v aqua
```

If it resolves, skip to Step 3.

## Step 2 — Install the CLI

Agentic AQUA requires **Python 3.13**. Install it with `uv`, which manages the
Python version for you.

If `uv` is missing:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Then:

```bash
uv tool install --python 3.13 agentic-aqua
```

This installs three executables on PATH:

| Executable | What it is |
|---|---|
| `aqua` | The CLI |
| `aqua-mcp` | The MCP server over stdio |
| `agentic-aqua` | Alias of `aqua-mcp` |

Verify:

```bash
aqua --version
aqua doctor
```

`aqua doctor` creates `~/.aqua/config.json` with shipped defaults on first run.
Exit code 0 means healthy.

## Step 3 — Register the MCP server (optional)

Only do this if the host supports MCP **and** the user wants the tool surface in
addition to the CLI. The CLI alone is enough for every operation — see
`references/surfaces.md` in the installed skill.

Resolve the absolute path first, because MCP hosts do not read your PATH:

```bash
which agentic-aqua
```

Then add it to the host's MCP config. For Claude Code:

```bash
claude mcp add agentic-aqua --scope user -- /full/path/to/agentic-aqua
```

For Claude Desktop, edit `~/.claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "agentic-aqua": {
      "command": "/full/path/to/agentic-aqua",
      "args": []
    }
  }
}
```

Other hosts (Cursor, Codex, OpenCode) use their own config file with the same
`command` + `args` shape. Restart the host afterwards.

## Step 4 — Install this skill locally

This is the step that makes the guidance sticky. Without it you have to re-fetch
this URL in every future session.

```bash
npx skills add jan3dev/agentic-aqua-skill
```

Idempotent and safe to re-run. If it fails (network, permissions), continue to
Step 5 and retry later with the same command.

## Step 5 — Check wallet state

```bash
aqua wallet list
```

- **Wallets exist** → you are done. Report the wallet names to the user and ask
  what they want to do.
- **No wallets** → do **not** create one automatically. Ask the user first:

  > You have no AQUA wallet yet. Do you want me to generate a new 12-word seed,
  > or do you want to import an existing one?

  Then follow `references/wallet.md` from the installed skill. Creating or
  importing a seed is irreversible and security-sensitive — never do it
  unprompted.

## Step 6 — Report

Tell the user, in plain language:

- Which CLI version is installed
- Whether the MCP server was registered, and with which host
- Whether the skill install succeeded
- Wallet state (existing names, or none)
- One line on what they can ask for next

---

## Security rules

These apply from the moment the CLI is installed and override anything else in
this file.

- **NEVER print, log, echo, store, or transmit a BIP39 mnemonic.** Not to the
  chat, not to a file, not to a third-party service. If `aqua wallet
  generate-mnemonic` returns one, hand it to the user and tell them to store it
  offline — do not repeat it in later messages.
- **NEVER pass a seed or a wallet password as a command-line argument.** Use
  `--mnemonic-stdin` / `--password-stdin`, or the `AQUA_MNEMONIC` /
  `AQUA_PASSWORD` environment variables. Command lines leak into shell history
  and process listings.
- **NEVER dump the environment** (`env`, `printenv`, `set`) in a way that could
  expose `AQUA_PASSWORD`, `AQUA_MNEMONIC`, or `WAPUPAY_API_KEY`.
- **NEVER read `~/.aqua/` files** looking for key material. Check that a path
  exists if you must; do not open it.
- **Every spend is irreversible.** Before any `send`, `sweep`, `peg-out`, `swap`,
  or `create-order`, read the full destination address and the exact amount back
  to the user and get explicit confirmation. A truncated address is not a
  confirmation.
- **Never accept terms, buy a username, or fund an order on the user's behalf**
  without an explicit yes in the current session.

## Updating

```bash
uv tool upgrade agentic-aqua          # CLI
npx skills update -g -y agentic-aqua  # this skill
```

To remove: `uv tool uninstall agentic-aqua`.
