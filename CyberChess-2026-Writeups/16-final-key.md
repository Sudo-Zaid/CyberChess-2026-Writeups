# 16 — Final Key

**Category:** Pwn (CyberChess mini-game → stack overflow → ret2libc)  |  **Difficulty:** Medium

**Flag:** `HTB{3v3ry0n3_l0v35_345t3r_4gg5}`

## How I found the flag

**Scenario:** "CyberChess" tavern mini-game. *"Finish this game. Win? Find the egg? What is the way to earn the key?"* Binary `last_key` (not stripped) ships with its own **glibc 2.35** (`./glibc/ld-...`, `libc.so.6`). Non-PIE, NX, **no canary**.

**The game (`alter_map`):** a 33×16 grid — player `@` at row 8, column `pos = rand()%5+1` (1–5); goal `*` at row 8, column `goal = rand()%10+10` (10–19). Each `read(0,buf,1)` move: `R`→`pos++`; win when `pos==goal` → the function **returns**. The banner even admits the game is a decoy: *"Congratulations on winning the game but it was not enough to get the prize.."*

**Bug (`set_score`, runs unconditionally after the game):** `fgets(buf, 0x80, stdin)` into a buffer only `0x10` below `rbp` → **stack overflow**, return address at offset **24**.

**Chain (ret2libc, deterministic — non-PIE + exact libc):**

1. **Win cleanly** — parse the first rendered map (`@`/`*` columns in the row-8 line) → send exactly `goal−pos` `R` bytes so no leftover corrupts the next read.
2. **Leak libc** — stage-1 ROP: `pop rdi;ret`(`0x40178d`) → `puts@got`(`0x403f88`) → `puts@plt`(`0x401100`) → return to `set_score`(`0x401687`) to loop. `libc_base = leak − 0x80ed0`.
3. **Shell** — stage-2: `pop rdi` → `/bin/sh`(`+0x1d8698`) → `ret`(`0x40101a`, stack align) → `system`(`+0x50d60`).

`getchar()` eats one byte before each `fgets`, so each stage is prefixed with a dummy byte. Shell as `uid=100(ctf)` → `cat /home/ctf/flag.txt`.
