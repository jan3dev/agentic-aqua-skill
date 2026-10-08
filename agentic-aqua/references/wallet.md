# Wallet

One BIP39 mnemonic derives **both** wallets: Liquid (LWK) and Bitcoin (BDK,
BIP84). There is no separate "create Bitcoin wallet" step.

## Creating a wallet

Two commands, in this order. `generate-mnemonic` only generates — it does not
create anything on disk.

```bash
aqua wallet generate-mnemonic
```

Returns 12 words. **Hand them to the user and stop.** Do not repeat them in a
later message, do not write them to a file, do not include them in a summary.

Then create the wallet from that mnemonic:

```bash
aqua wallet import-mnemonic --mnemonic-stdin --password-stdin \
  --wallet-name default --network mainnet
```

| Flag | Default | Notes |
|---|---|---|
| `--mnemonic-stdin` | — | Reads from piped stdin, or prompts. Falls back to `AQUA_MNEMONIC`, then an interactive prompt. |
| `--wallet-name` | `default` | |
| `--network` | `mainnet` | `mainnet` or `testnet` |
| `--password-stdin` | — | Reads from piped stdin, or prompts. Falls back to `AQUA_PASSWORD`, then **no password**. |

**Never pass the mnemonic or password as a CLI argument.** There is no
`--mnemonic` flag precisely because command lines leak into shell history and
process listings.

### Password

Without a password the mnemonic is stored **in plaintext** under `~/.aqua/`.
Tell the user that explicitly before importing without one. With a password the
mnemonic is encrypted, and every spending command needs it again.

There is no recovery. Lose the password, lose access to that wallet's stored
seed — the 12 words are the only way back.

## Importing an existing wallet

Same command. The user supplies their own 12 words instead of generated ones:

```bash
aqua wallet import-mnemonic --mnemonic-stdin --password-stdin --wallet-name mywallet
```

Ask them to paste the words into the interactive prompt rather than into the
chat. If they already pasted them into the conversation, use them, but tell
them that transcript now contains their seed and they should treat the wallet
as compromised if the transcript is stored anywhere.

## Listing and deleting

```bash
aqua wallet list
```

```bash
aqua wallet delete --wallet-name mywallet
aqua wallet delete --wallet-name mywallet --yes   # skips confirmation
```

`delete` removes the wallet **and all its cached data**. If the mnemonic is not
backed up offline, the funds are gone. Always confirm with the user first, and
only pass `--yes` after they have said yes in conversation.

## Balances

Unified across both chains:

```bash
aqua --format json balance --wallet-name default
```

Per chain, see [liquid.md](./liquid.md) and [bitcoin.md](./bitcoin.md).

## Watch-only descriptors

Export to monitor a wallet elsewhere without the seed:

```bash
aqua liquid export-descriptor --wallet-name default   # CT descriptor
aqua btc export-descriptor --wallet-name default      # BIP84 descriptors + xpub
```

Import a watch-only wallet:

```bash
aqua liquid import-descriptor --descriptor "ct(...)" --wallet-name watch-liquid
aqua btc import-descriptor --descriptor "wpkh(...)" --wallet-name watch-btc
```

**The two chains do not derive from each other.** The Liquid CT descriptor
cannot produce the Bitcoin descriptor (different derivation path and xpub), and
the Bitcoin xpub cannot produce the Liquid one (different path plus a SLIP-77
blinding key). Export and import both separately if the user wants full
watch-only coverage.

`btc import-descriptor` auto-derives the change descriptor from the external one
(`/0/*` → `/1/*`) unless you pass `--change-descriptor`.

A watch-only wallet can read balances and history. It cannot sign — any `send`
or `sweep` against it will fail.

### Descriptors are privacy-sensitive

An exported descriptor reveals every past and future address of that wallet. It
cannot spend, but it does deanonymize the whole wallet. Hand it to the user,
never to a third-party service, and never paste it into a shared channel.

## Where state lives

`~/.aqua/` — config, encrypted seed material, JAN3 sessions, the WapuPay API
key, and swap records needed for Lightning refunds.

Do not read it looking for secrets, and do not delete it. Deleting swap records
can make a failed Lightning swap unrecoverable — see [lightning.md](./lightning.md).
