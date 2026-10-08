# Install

Agentic AQUA ships one Python package that provides both a CLI and an MCP
server. Installing it once gives you both.

**Python 3.13 is required** (`>=3.13,<3.14`). Use `uv` so you do not have to
manage that yourself.

## Install

```bash
# uv, if missing
curl -LsSf https://astral.sh/uv/install.sh | sh          # macOS / Linux
powershell -c "irm https://astral.sh/uv/install.ps1 | iex" # Windows

uv tool install --python 3.13 agentic-aqua
```

Three executables land on PATH:

| Executable | Entry point | What it is |
|---|---|---|
| `aqua` | `aqua.cli.main:cli` | The CLI |
| `aqua-mcp` | `aqua.server:main` | MCP server over stdio |
| `agentic-aqua` | `aqua.server:main` | Alias of `aqua-mcp` |

`aqua serve` starts the same MCP server from the CLI.

## Verify

```bash
aqua --version
aqua doctor
```

`aqua doctor` creates `~/.aqua/config.json` with shipped defaults on first run
and exits 0 when healthy. `aqua doctor --fix` repairs a corrupted config. See
[troubleshooting.md](./troubleshooting.md).

## Register the MCP server

Optional. The CLI covers every operation on its own — see
[surfaces.md](./surfaces.md) before deciding.

MCP hosts do not read your shell PATH, so resolve the absolute path first:

```bash
which agentic-aqua
# e.g. /home/you/.local/bin/agentic-aqua
```

**Claude Code:**

```bash
claude mcp add agentic-aqua --scope user -- /full/path/to/agentic-aqua
```

**Claude Desktop** — `~/.claude/claude_desktop_config.json`:

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

Cursor, Codex and OpenCode use their own config file with the same
`command` + `args` shape. Restart the host after editing.

## Update and remove

```bash
uv tool upgrade agentic-aqua
uv tool uninstall agentic-aqua
```

Uninstalling leaves `~/.aqua/` in place — wallets and config survive. To wipe
state the user must delete that directory themselves. **Never delete it for
them**: it holds encrypted seed material and swap refund records.

## From source

```bash
git clone https://github.com/jan3dev/agentic-aqua.git
cd agentic-aqua
uv python install 3.13
uv sync --python 3.13
uv run aqua --help
```

## Troubleshooting install

| Symptom | Cause | Fix |
|---|---|---|
| `No solution found … requires-python` | Wrong Python | `uv tool install --python 3.13 agentic-aqua` |
| `aqua: command not found` after install | `~/.local/bin` not on PATH | `uv tool update-shell`, then restart the shell |
| MCP host shows no AQUA tools | Relative path in config | Use the absolute path from `which agentic-aqua` and restart the host |
| MCP host shows only some tools | Feature flags | Check `enabled_tools` in `~/.aqua/config.json` — see [surfaces.md](./surfaces.md) |
