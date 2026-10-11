# 17 — WeakLink-2

**Category:** ICS / OPC UA (self-signed cert auto-trust → active-recipe write)  |  **Difficulty:** Medium

**Flag:** `HTB{@ut0_tru5t_15_n0_tru5t}`

## How I found the flag

**Scenario:** MeridiPharma **Line 3 fill/finish**. WeakLink-1's patch hardened the OPC UA endpoint to **Basic256Sha256 + SignAndEncrypt + client certificate required** — *but left the trust store on auto-trust*. Two services: the **OPC UA server** (`opc.tcp://…:31334/meridipharma/line3/`, FreeOpcUa Python) and a **Flask HMI** (read-only monitor).

**Recon (the win condition is in the HMI):** `GET /api/state` → `commissioning_log: "(sealed — token released after active-recipe change)"`, and the page JS gates the seal on `released = /HTB\{[^}]+\}/.test(commLog)`. So: **change the active recipe over OPC UA → server injects the flag into the commissioning log.**

**Chain (all "hardening" defeated):**

1. **Mint & walk in** — endpoint advertises `SignAndEncrypt`/`Basic256Sha256` but accepts user token **`anonymous`**. Generated a self-signed RSA-2048 client cert (keyUsage digital-signature/key-encipherment, EKU client/serverAuth, **SAN URI = the client application URI**). The server **auto-trusted** it → secure channel established (`asyncua`).
2. **Find the bug node** — browsed ns=2 (`http://meridipharma.local/line3`): active recipe `ns=2;i=5 Recipe` has **AccessLevel=3 (read+write)** — the WeakLink-1 recipe-write bug; the sealed `CommissioningLog` sits at `ns=2;i=32` under `Recipes/VAX-2026`.
3. **Release the seal** — wrote `ns=2;i=5 = "VAX-2026"` (switch the live line to the staged pandemic-vaccine recipe) → backend released the commissioning token → `/api/state` `commissioning_log` became the flag.
