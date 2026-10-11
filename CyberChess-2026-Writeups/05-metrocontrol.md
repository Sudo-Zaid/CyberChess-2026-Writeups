# 05 — MetroControl

**Category:** ICS  |  **Difficulty:** Medium

**Flag:** `HTB{r3v3nu3_br04dc45t5_l34k_th3_pl4nt_k3y}`

## How I found the flag

**Target:** Siemens **S7-1200** signalling PLC (Aurora Junction). The flag ("audit ticket") shows on the HMI when the auditor consist **T-AUDIT** is dispatched onto a service pattern via **DB200** (`magic, route_id, auth, source`).

**Technique:** Spoke S7Comm with **python-snap7** (`client.connect(ip,0,1,port)`).

- Read `DB99` (engineering ref) + `DB200` (dispatch reg). `magic = deadbeef`, `source = 0x01 runtime / 0x00 auditor`.
- Polled DB200 during revenue broadcasts → `auth` varies per **destination station**. Using the train's out-and-back motion (middle station repeats → equal auth) recovered the per-(route,station) auth table.
- **Key scheme:** `auth = 0xC9 ⊕ route_id ⊕ XOR(station_name_ASCII)` — fits all 9 observations (K=0xC9, the "plant key leaked by revenue broadcasts").
- Route 4 (yard) is the only **free** track (occupancy=0); dest = YARDSPUR → `auth = 0xC9 ⊕ 4 ⊕ XOR("YARDSPUR") = 0xC7`.
- Wrote `DB200 = deadbeef 0004 c7 00` → `AUDITOR DISPATCH ACCEPTED` → ticket released.
