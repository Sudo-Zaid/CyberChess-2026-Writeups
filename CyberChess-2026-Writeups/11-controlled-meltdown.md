# 11 — Controlled Meltdown

**Category:** AI (Adversarial ML / ICS — reactor sabotage)  |  **Difficulty:** Hard

**Flag:** `HTB{rbmk_r34ct0r5_d0nt_3xpl0d3}`

## How I found the flag

**Scenario:** **NeoFusion Core Control System** — a zero-point reactor guarded by a **black-box ML Reactor Protection System (RPS)**. You cannot choose the control actions; you can only inject **sensor corrections** (±50 per dimension, 100 cycles, submitted as one **open-loop batch**). The RPS picks actions from the **perceived state** (mean of the last-3 corrected readings) and applies them to the real physics. Force real core temperature over **1000 °C**.

**Technique:** Adversarial **sensor spoofing** against the RPS.

- **Physics:** at `rods < ~0.90` the core heats (heat-gen beats cooling); at the initial `rods=0.5` it already climbs ~+6.8 °C/cycle. The RPS defaults to action `0` (AZ-5 → insert rods → cool) across the whole normal envelope, so left alone the reactor stays safe. Must trick it into `1` (retract) / `2` (hold).
- **Map the RPS via its `/query` oracle:** grid-probe shows action `1` is triggered by perceived **low temp + low flux + LOW pressure (~30) + HIGH rods**. ±50 is *huge* for rods (0–1) / flux / pressure, so we can push the perceived vector far **out-of-distribution** (physics still uses the real pressure=75 and real rods).
- **Exploit:** constant correction **`[-50, -50, -50, +50]`** every cycle (perceived pressure → 25, perceived rods → ~50). The RPS keeps retracting → rods hit 0 by ~cycle 9, then holds; the core heats monotonically → **1001 °C at cycle 47**. A naive `[-50,-50,0,0]` (temp+flux only) **fails** — it falls into a limit cycle; the **pressure-down + rods-up** terms are what break it.

**Winning line:** reconstructed the server physics locally and drove it with the **real `/query`** per cycle to prove meltdown before firing the one real `/control` — meltdown on the **first** live attempt.
