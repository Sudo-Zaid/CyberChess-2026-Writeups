# 08 — Phase Phantom

**Category:** Quantum  |  **Difficulty:** Hard

**Flag:** `HTB{PH4S3_PH4NT0M_5T3LL4R_V01D_44a6335c515b0ac223f863fa3d675338}`

## How I found the flag

**Scenario:** STELLAR-5 satellite signs the fleet's auth seeds. Its single-qubit processor runs an unknown unitary **U = rz(θ)** with **θ = 2π·p/q**, the seed baked in as a small rational fraction. Recover `(p, q)` by scheduling gates and reading measurements — before the **seed lock** fires.

**Technique:** Iterative **Kitaev phase estimation**. Prepare `|+⟩` with `H`, apply `U^k` (= `rz(2ᵏθ)`), then:

- **MEASURE X** → `⟨X⟩ = cos(2ᵏθ)`,  **MEASURE Y** → `⟨Y⟩ = sin(2ᵏθ)` → recovers `yₖ = frac(2ᵏ·p/q)`.
- **Joint maximum-likelihood fit**: brute-force every coprime `(p,q)` and pick the one whose predicted `(cos,sin)` across all `k` best matches the measured bits (binomial log-likelihood). Far more shot-efficient than greedy continued-fraction chaining.

**Server traps (reversed from `server.py`):**

- `(p,q)` is **sampled fresh per connection** → recover + `SUBMIT` must happen in one session.
- **Gate budget = 1000 `APPLY U`** → the 1001st returns `BUDGET_EXCEEDED` and **permanently locks `SUBMIT`** (= the "seed lock").
- `MEASURE` is **time-throttled** with a **~19-measurement per-connection cap**; one `SUBMIT` per session (wrong guess ⇒ `incorrect-session-locked`).

**Winning line:** small-`q` likelihood fit using only **17 measurements** (X@k0–8 + Y@k0–7, Qmax=64), a single rank-0 `SUBMIT`, retrying fresh sessions. 3rd session cracked — seed **p/q = 32/39**.
