# ccx — for when you're too rich and have too many Claude Code accounts

> One Claude Max account isn't enough? Two still doesn't cut it? You've got three,
> four, five of them and you're sick of `/logout` → `/login` every time you hit
> "5-hour limit reached"? Congratulations, you are the *target audience* of this tool.
> Our condolences to your wallet.

`ccx` is **a single bash file**, lighter than the guilt of paying for a third account.
Nothing to install beyond `bash`, `awk` and `python3` (only for reading JSON, promise).
Good enough, no frills, no framework, no "AI-powered account manager".

## What it does

Claude Code reads all of its config from the directory pointed to by `CLAUDE_CONFIG_DIR`.
`ccx` gives every account its own "apartment" under `~/.claude-accounts/<name>/`, so
credentials, conversation history, `settings.json` and MCP servers are **fully isolated**
per account — nobody steps on anybody's toes:

- log into the whole herd once, never `/logout` → `/login` again;
- run several terminals side by side, one account per window, farm away;
- when one account runs out of breath, `ccx next` hops to the one that's rested the
  longest — fair rotation, like a shift roster;
- always runs `claude --dangerously-skip-permissions` by default, because when you're
  this rich you shouldn't have to click "Yes" every time (can be turned off, see below).

It also keeps a little "timesheet" at `~/.claude-accounts/ledger.tsv`, one line per
account, 3 TAB-separated columns:

```
<account name>	<epoch of last use>	<epoch when cooldown ends>
```

## Install (30 seconds, faster than topping up your card)

```bash
install -Dm755 ccx ~/.local/bin/ccx
```

Make sure `~/.local/bin` is in your `PATH`, then add the aliases to save your fingers:

```bash
ccx shellinit >> ~/.bashrc && source ~/.bashrc
```

## Commands

| Command | What it does |
| --- | --- |
| `ccx add <name>` | create the account's "apartment" (chmod 700) and open Claude Code so you can `/login` |
| `ccx ls` | roll call: NAME / EMAIL / PLAN / LAST USED / STATUS (default when you run bare `ccx`) |
| `ccx use <name> [args…]` | run Claude Code as that account, extra args passed straight to `claude` |
| `ccx next [args…]` | pick the account that is **not** cooling down and has rested the longest — "whoever's free, go" |
| `ccx sh <name>` | open a subshell with `CLAUDE_CONFIG_DIR` and `CCX_ACCOUNT` already exported |
| `ccx env <name>` | print `export` lines for `eval` |
| `ccx limit <name> [hours]` | mark "this one's out of breath", rests 5 hours by default |
| `ccx clear <name>` | clear the cooldown (when you know it's recovered) |
| `ccx usage <name>` | open Claude Code with a reminder to type `/usage` and check how much HP is left |
| `ccx login <name>` | log in again |
| `ccx token <name>` | paste a long-lived OAuth token (`claude setup-token`), stored chmod 600 |
| `ccx rm <name>` | evict the account; you must retype the exact name to confirm (no undo, seriously) |
| `ccx shellinit` | print the alias snippet for `~/.bashrc` |
| `ccx --help` | print the header comment of the script |

## A typical day in the life of a rich person

```bash
ccx add max1        # first account, log in with /login
ccx add max2        # second account, "just as a backup"
ccx add max3        # third account, no explanation needed

ccx ls
# NAME  EMAIL                 PLAN LAST USED   STATUS
# max1  kang+max1@gmail.com   max  3h ago      ready
# max2  kang+max2@gmail.com   max  30m ago     limit hit, 4h12m left
# max3  kang+max3@gmail.com   max  never       ready

ccx next                       # picks max3 (rested the longest)
ccx use max1 --model opus      # explicit account, args go straight to claude
CCX_DANGER=0 ccx use max1      # feeling cautious today: WITHOUT --dangerously-skip-permissions
ccx limit max1                 # just got "5-hour limit reached" → bench it for 5h, move on to max2/max3
```

`ccx sh <name>` opens a subshell with the environment already set — handy when you
need to run several `claude` commands in a row as the same account. (Inside that shell,
add the flag yourself when you type `claude`; `ccx` can't read your mind.)

## Long-lived token (optional, for people too lazy to open a browser)

```bash
CLAUDE_CONFIG_DIR=~/.claude-accounts/max1 claude setup-token
ccx token max1      # paste the token, stored at ~/.claude-accounts/max1/oauth_token (chmod 600)
```

When `oauth_token` exists, `use` / `next` / `sh` automatically export `CLAUDE_CODE_OAUTH_TOKEN`.

## Environment variables

| Variable | Default | Meaning |
| --- | --- | --- |
| `CCX_HOME` | `~/.claude-accounts` | root directory holding the accounts and `ledger.tsv` |
| `CCX_CLAUDE_BIN` | `claude` | the Claude Code command to run |
| `CCX_DANGER` | `1` | always run `claude` with `--dangerously-skip-permissions`; set to `0` if you'd like Claude to ask before doing things |

## Notes (the only serious part)

- This script is for managing **your own** accounts. Sharing one account between
  multiple people violates Anthropic's terms of service — be rich, don't be sneaky.
- `--dangerously-skip-permissions` is *dangerously* in the literal sense: Claude will run
  commands, edit files and delete files without asking. Whatever directory you run it in,
  you own the consequences. Scared? `CCX_DANGER=0`.
- `ccx limit` is a **manual** mark — Claude Code has no API for a script to detect that
  you've hit the limit; when you see "5-hour limit reached", type `ccx limit <name>`,
  think of it as punching the clock.
- Passes `bash -n` and `shellcheck -S warning` clean. Light and good enough, that's it.
