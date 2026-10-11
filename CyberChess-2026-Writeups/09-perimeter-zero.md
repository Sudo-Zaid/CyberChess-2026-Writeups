# 09 — Perimeter Zero

**Category:** Pwn (ZTAP / glibc 2.31 heap → ROP, seccomp ORW)  |  **Difficulty:** Insane

**Flag:** `HTB{tunnel_v1s10n_gr0up_p0l1cy_h34p_0verwr1te_2_r0p_0rw_817d852f34acc19bc31134bc8f5c0a39}`

## How I found the flag

**Scenario:** "Zero-Trust Access Gateway" speaking a custom binary protocol **ZTAP/1**. Forks a fresh worker per connection, each locked in a **seccomp sandbox** (no shell to pop). Reverse the protocol, find the bug in the policy compiler, read the vault off disk.

**Protocol:** 12-byte frame `5A 01 <cmd> 00 <len:u32> <4b>` + payload. Commands: `01 HANDSHAKE`, `02 TUNNEL_OPEN`, `03 TUNNEL_CLOSE`, `04 POLICY_PUSH`, `05 POLICY_DUMP`.

**Bug (policy compiler):** `POLICY_PUSH` does `malloc(0x40)` but reads **up to 4096 bytes** into it → **heap overflow**. `POLICY_DUMP` sends back an attacker-controlled length from that 64-byte slot → **OOB heap read**. `TUNNEL_OPEN` = fully controlled `malloc(size)` + content; `TUNNEL_CLOSE` = free primitive.

**Seccomp:** classic **ORW sandbox** — `open/openat/read/write/sendto/lseek/fstat` allowed, **no execve/clone**, `rt_sigreturn` allowed.

**Chain (glibc 2.31, no safe-linking, fork-stable ASLR):**

1. **Libc leak** — OOB `DUMP` reads an unsorted-bin chunk's fd (`main_arena+0x60`) ⇒ `libc_base = leak − 0x1ecbe0`.
2. **tcache poisoning** — overflow a staged `0x50` slot to point `tcache[0x50]`'s fd at an arbitrary address ⇒ **arbitrary read + write** (poisoned slot becomes both the `DUMP` source and the `PUSH` destination).
3. **Leak the rest** — arb-read `environ` → stack; scan the stack for the dispatch return `PIE+0x1810` ⇒ PIE base + the address `S` holding a handler's **saved return address**.
4. **RIP control** — arb-write the ORW **ROP chain over `S`**; the *writing* `POLICY_PUSH` overwrites its own saved return and `ret`s straight into the chain (canary/saved-regs sit below `S`, untouched).
5. **ORW** — `open("flag.txt",0)` → `read(3,buf,0x200)` → `write(4,buf,0x200)` (fd 4 = client socket) → **blocking `read(4,…)`** so the process doesn't `exit` and RST-discard the sent bytes.
