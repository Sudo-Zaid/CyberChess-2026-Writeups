# 06 — Siren

**Category:** Web  |  **Difficulty:** Medium

**Flag:** `HTB{c4p_1d3nt1f13r5_n33d_m0r3_th4n_p4rt14l_bl0ckl15t5_d0959a43702e7950ca44c9b7a208c5c8}`

## How I found the flag

**Target:** Flask "CEMA Alert" console. `/preview` takes an **Audio file identifier** used to build `/tmp/<identifier>.wav`, which is passed **unquoted into a shell command** (TTS render).

**Bug:** Partial blocklist only — blocked `space`, `&`, `<`, `,`; `; | ( ) " '` just broke the command. But **`$(...)` command substitution was allowed**; used **`${IFS}`** for spaces → blind RCE.

- No egress (curl/wget absent). Built a **time-based oracle**: `$(sleep$(grep -c PATTERN FILE))` → delay iff match.
- Located flag at **`/flag.txt`**, cwd = app dir (`index.js`), static dir = **`static/`** (served at `/`).
- Exfil: `$(cat /flag.txt > static/leak.txt)` then `GET /leak.txt`.
