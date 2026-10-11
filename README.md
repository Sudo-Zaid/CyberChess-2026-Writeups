# CyberChess 2026 — Writeups

Solved challenges from the **CyberChess** CTF (flag format `HTB{...}`).
Each file is a short writeup: challenge info, the flag, and how I found it.

## 🏆 Result

> ### 🐐 Team Wargaye 🇵🇰 — **Full clear 49/49 flags** — **16,750 points** — **3 first bloods**
> Every challenge on the board cracked and captured.

| Metric | Value |
|--------|------:|
| **Flags** | **49 / 49** |
| **Points** | **16,750** |
| **First bloods** | **3** |

**Writeups documented: 18** across these categories — Pwn, Reversing, Crypto, Web, Forensics, OSINT, ICS, AI and Quantum.

| # | Challenge | Category | Difficulty | Key technique |
|---|-----------|----------|------------|---------------|
| 01 | [Operation Phantom Runner](CyberChess-2026-Writeups/01-operation-phantom-runner.md) | OSINT | Easy | Entities hard-coded in client JS bundles |
| 02 | [Locker](CyberChess-2026-Writeups/02-locker.md) | Reversing | Medium | Androguard decompile, fixed AES-256 key |
| 03 | [pump it](CyberChess-2026-Writeups/03-pump-it.md) | Crypto | Hard | SHA-512 hash length extension |
| 04 | [twisted cloud](CyberChess-2026-Writeups/04-twisted-cloud.md) | Crypto | Hard | PHP `mt_rand` seed brute-force (numba) |
| 05 | [MetroControl](CyberChess-2026-Writeups/05-metrocontrol.md) | ICS | Medium | S7Comm DB write, XOR auth key leak |
| 06 | [Siren](CyberChess-2026-Writeups/06-siren.md) | Web | Medium | Blind RCE via `$(...)` + `${IFS}` |
| 07 | [Shard Council](CyberChess-2026-Writeups/07-shard-council.md) | AI | Medium | LLM PIN leak + prompt-injection SSRF |
| 08 | [Phase Phantom](CyberChess-2026-Writeups/08-phase-phantom.md) | Quantum | Hard | Kitaev phase estimation + ML fit |
| 09 | [Perimeter Zero](CyberChess-2026-Writeups/09-perimeter-zero.md) | Pwn | Insane | Heap overflow → tcache → ROP ORW |
| 10 | [GHZ Collapse](CyberChess-2026-Writeups/10-ghz-collapse.md) | Quantum | Hard | Signed-GHZ stabilizer readout |
| 11 | [Controlled Meltdown](CyberChess-2026-Writeups/11-controlled-meltdown.md) | AI | Hard | Adversarial sensor spoofing vs RPS |
| 12 | [SmartBridge](CyberChess-2026-Writeups/12-smartbridge.md) | ICS | Insane | SSRF → stolen mTLS → cEMI passthrough |
| 13 | [The Debug Door](CyberChess-2026-Writeups/13-the-debug-door.md) | Forensics | Medium | AWS CloudTrail log correlation (21/21) |
| 14 | [Unfinished Business](CyberChess-2026-Writeups/14-unfinished-business.md) | Forensics | Hard | Volatility 3 + AES key carve |
| 15 | [Gradient Heist](CyberChess-2026-Writeups/15-gradient-heist.md) | AI | Medium | Gradient leakage from a zeroed param |
| 16 | [Final Key](CyberChess-2026-Writeups/16-final-key.md) | Pwn | Medium | Stack overflow → ret2libc |
| 17 | [WeakLink-2](CyberChess-2026-Writeups/17-weaklink-2.md) | ICS | Medium | OPC UA auto-trust → recipe write |
| 18 | [Dash Rush](CyberChess-2026-Writeups/18-dash-rush.md) | Web | Easy | pg-promise `--` type confusion |

## Categories

- **Pwn** — 2
- **Reversing** — 1
- **Crypto** — 2
- **Web** — 2
- **Forensics** — 2
- **OSINT** — 1
- **ICS** — 3
- **AI** — 3
- **Quantum** — 2

## Notes

- All flags use the `HTB{...}` format.
- Entries 1–7 are from the main event; 8–18 were cracked in follow-up sessions (quantum, adversarial-ML, ICS, cloud/memory forensics and pwn), all flags captured.

---

See [COPYRIGHT.md](CyberChess-2026-Writeups/COPYRIGHT.md).

> These writeups were written by **Claude Code**, with the help of screenshots and content provided by **MUHAMMAD ZAID**.
