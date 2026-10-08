# JAN3 account

A JAN3 account adds a **Lightning Address** to a self-custodial AQUA wallet.
Inbound Lightning payments are delivered to Liquid addresses from that wallet's
own address pool — the keys never leave the device.

Sessions are per-email and persisted locally. Multiple accounts can coexist.

## Login

Two flows. **Use the default one.**

### Default — free email OTP

```bash
aqua jan3 login --email user@example.com --language en
aqua jan3 verify --email user@example.com --otp-stdin
```

`--language` is `en`, `es` or `pt` and only affects the OTP email.

`verify` accepts `--otp 123456`, but prefer `--otp-stdin` or the interactive
prompt. An OTP in a command line lands in shell history.

### Fallback — paid captchaless

Only when the default flow is blocked. **This one costs money** — it pays a fee
from the wallet to trigger the OTP.

```bash
aqua jan3 login-start --email user@example.com --wallet-name default --password-stdin
aqua jan3 login-complete --email user@example.com --otp 123456
```

Tell the user it is a paid flow and get an explicit yes before running
`login-start`.

### OTP handling

- **Never guess or hardcode the user's email.** Ask for it.
- **Never store or display an OTP beyond its immediate use.** Do not repeat it
  in a summary.
- An OTP is single-use and short-lived. If it expires, start over with `login`.

## Sessions

```bash
aqua jan3 list-sessions                      # no tokens shown
aqua jan3 session-info --email user@example.com
aqua jan3 logout --email user@example.com    # deletes the persisted session
```

Session tokens grant account access. They live under `~/.aqua/` — do not read
those files and never echo a token.

## Account info

```bash
aqua jan3 user-info --email user@example.com --wallet-name default
```

Profile plus Lightning Address status. When the LN address is active this also
tops up the wallet's Liquid address pool (best-effort, reported under
`ln_address_pool`). If inbound Lightning stops arriving, run this — an exhausted
address pool is a plausible cause.

## Lightning username

### Check availability first

```bash
aqua jan3 ln-check-username --email user@example.com --ln-username andy
```

`--ln-username` is the local part only — `andy`, not `andy@domain`.

### Purchase

```bash
aqua jan3 purchase-ln-username --email user@example.com --ln-username andy \
  --asset l-btc --wallet-name default --password-stdin
```

**This spends money on-chain.** Funded with L-BTC (default) or USDt via
`--asset usdt`.

Without `--yes` the command quotes the price and asks for confirmation, then
funds that exact quoted order — aborting if the quote expired. `--yes` skips the
quote and accepts the current price sight unseen.

**Do not pass `--yes` unless the user has already seen a price and agreed to
it.** Let the prompt show the quote, relay it, and get a yes.

## Lightning Address

```bash
aqua jan3 enable-lightning-address --email user@example.com --enable --wallet-name default
aqua jan3 enable-lightning-address --email user@example.com --disable --wallet-name default
```

Enabling populates a batch of Liquid receive addresses so AQUA can deliver
inbound Lightning payments into them. That is the mechanism: Lightning in,
L-BTC out, on addresses the user controls.

### Rebinding to a different wallet

```bash
aqua jan3 rebind-wallet --email user@example.com --wallet-name otherwallet
```

**Destructive.** It moves Lightning Address delivery to a different local
wallet. Payments after the rebind land in the new wallet; the old binding is
overwritten.

The command previews the change and asks for confirmation unless `--yes` is
given. Rebinding to the already-bound wallet is a no-op.

Only pass `--yes` after the user has explicitly confirmed the new wallet by
name.

## Order of operations

For a user starting from nothing:

1. `aqua jan3 login` → `aqua jan3 verify`
2. `aqua jan3 ln-check-username` to find a free name
3. `aqua jan3 purchase-ln-username` (quote → user confirms → buy)
4. `aqua jan3 enable-lightning-address --enable`
5. `aqua jan3 user-info` to confirm it is live

Step 3 costs money. Steps 1, 2, 4 and 5 do not.
