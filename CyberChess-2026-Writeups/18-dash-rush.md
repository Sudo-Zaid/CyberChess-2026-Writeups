# 18 — Dash Rush

**Category:** Web (pg-promise `--` double-dash type confusion)  |  **Difficulty:** Easy

**Flag:** `HTB{d0ubl3_d4sh_d0ubl3_th3_pr0f1t_5342ed5f4cdac47c245280e438a69365}`

## How I found the flag

**Target:** "Maelstrom's Mug" pirate-tavern betting app — **Fastify + PostgreSQL via pg-promise 11.5.4**; `package.json` brands it a **"CVE-2025-29744 SQL Injection Demo"**. The flag is served by `POST /api/account` **only when `account.coins >= 1337`** (you start at 100). DB user is superuser `postgres`; flag also at `/flag.txt`.

**Decoys:** every server query uses `$1` parameters (safe); the client `query-builder.js` is a purely cosmetic "live preview" illustrating the CVE. The real, reachable bug is in `placeStake`: `UPDATE accounts SET coins = coins-$1` with **no positivity check**, and pg-promise formats values **inline** (client-side), not as a server prepared statement.

**Bug (numeric vs string type confusion):**

- `amount = -5000` (JSON **number**) → `coins--5000` — the `--` is a **SQL line comment** → the rest of the statement (incl. `WHERE`) is commented out, coins **unchanged**. (This is the "double dash" trap — exactly the CVE behavior.)
- `amount = "-5000"` (JSON **string**) → pg-promise quotes/escapes it → `coins-'-5000'` → PostgreSQL coerces the literal and computes `coins - (-5000)` = **coins + 5000**. (Arbitrary SQLi is *not* possible — strings are safely cast, and non-numeric payloads die with `invalid input syntax for integer` — only this arithmetic-coercion path works.)
- The guard `if (account.coins < amount)` is bypassed too: JS `100 < "-5000"` coerces → `false`.

**Winning line:** register → login → `POST /api/stake {contestantId:"c6", amount:"-5000"}` → coins **100 → 5100** → `POST /api/account {username}` → flag (gated on `coins ≥ 1337`).
