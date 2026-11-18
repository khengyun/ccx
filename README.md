# ccx — for when you're too rich and have too many Claude Code accounts

> One Claude Max account isn't enough? Sick of `/logout` → `/login` every time you hit
> "5-hour limit reached"? Congratulations, you are the *target audience*.

`ccx` is **a single bash file**. It does one thing: run several Claude Code accounts
on one machine without them stepping on each other.

## How it works

Claude Code reads all of its config from `CLAUDE_CONFIG_DIR`. `ccx` gives every account
its own directory under `~/.claude-accounts/<name>/`, so credentials, history and
`settings.json` are fully isolated. Log in once per account, run them side by side,
and when one runs out of breath, `ccx next` hops to the one that's rested the longest.

When Claude Code exits, `ccx` prints the session id you just ran and how to resume it:

```
session: 1b4eac0b-48c2-4351-9953-b089498958f2
resume:  ccx use max1 --resume 1b4eac0b-48c2-4351-9953-b089498958f2
```

(Plain `claude --resume` won't find it — it doesn't know which account to look in.)

## Install

```bash
install -Dm755 ccx ~/.local/bin/ccx
```

Optional aliases for `~/.bashrc`:

```bash
alias cc='ccx next'
alias ccl='ccx ls'
ccu() { ccx use "$@"; }
ccenv() { eval "$(ccx env "$1")"; }
```

## Commands

| Command | What it does |
| --- | --- |
| `ccx add <name>` | create the account and open Claude Code so you can `/login` |
| `ccx ls` | NAME / EMAIL / LAST USED / STATUS (default when you run bare `ccx`) |
| `ccx use <name> [args…]` | run Claude Code as that account; extra args go straight to `claude` |
| `ccx next [args…]` | pick the free account that has rested the longest |
| `ccx limit <name> [hours]` | mark "hit the limit", benched for 5 hours by default |
| `ccx clear <name>` | clear the cooldown |
| `ccx env <name>` | print `export` lines for `eval` |
| `ccx rm <name>` | delete the account; retype the name to confirm (no undo) |

Need `/usage` or `/login` again? `ccx use <name>` and type it inside Claude Code.

## A typical day

```bash
ccx add max1                 # log in with /login
ccx add max2
ccx ls
# TÊN   EMAIL               DÙNG LẦN CUỐI  TRẠNG THÁI
# max1  kang+1@gmail.com    3h trước       sẵn sàng
# max2  kang+2@gmail.com    30p trước      limit, còn 4h12p

ccx next                     # picks max1
ccx use max2 --model opus    # explicit account
ccx limit max1               # just saw "5-hour limit reached" → bench it
```

## Environment variables

| Variable | Default | Meaning |
| --- | --- | --- |
| `CCX_HOME` | `~/.claude-accounts` | where accounts and `ledger.tsv` live |
| `CCX_CLAUDE_BIN` | `claude` | the Claude Code command |
| `CCX_DANGER` | `1` | add `--dangerously-skip-permissions`; `0` to have Claude ask first |

## Notes

- For **your own** accounts only. Sharing an account between people violates
  Anthropic's terms — be rich, don't be sneaky.
- `--dangerously-skip-permissions` means Claude runs commands and edits files without
  asking. Scared? `CCX_DANGER=0`.
- `ccx limit` is manual: there's no API to detect the limit, so type it when you see it.
- `ledger.tsv` is one line per account: `<name>	<last used epoch>	<cooldown end epoch>`.
- Needs only `bash`, `awk`, `grep`, `column`. Passes `shellcheck` clean.
