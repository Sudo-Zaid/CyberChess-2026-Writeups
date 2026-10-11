# 10 — GHZ Collapse

**Category:** Quantum (signed-GHZ stabilizer readout)  |  **Difficulty:** Hard

**Flag:** `HTB{p4u1i_st4b1l1z3rs_unl0ck_th3_ghz_5ecr3t_291e599502e752180980a5584dae6fa8}`

## How I found the flag

**Scenario:** Halcyon Dynamics' **STELLAR-4** relay shares a **5-qubit GHZ state** across five satellite nodes; the authentication descriptor is baked into *how* the qubits are jointly entangled. The tamper cage never exposes the secret as a classical bitstring — the only way in is to measure the qubits in the right Pauli bases and rebuild the descriptor from the joint correlations. Recover the per-session secret across **all 8 GHZ preparations**, then `/submit`.

**Technique:** Each preparation is a **signed GHZ stabilizer state** `|ψ⟩ = (|v⟩ + (-1)ᵖ|v̄⟩)/√2`. Secret = **5 bits/state** = 4 (`s_vec`) + 1 (`p`) → 8×5 = **40 bits = 10 hex**.

- **MEASURE ZZZZZ** → outcome is `v` or `v̄` (random global flip). The **pairwise-XOR correlations are flip-invariant** → recover `v` (qubit 0 is the GHZ control, `v₀=0`) ⇒ `s_vec = v[1:5]`.
- **MEASURE XXXXX** → `X^⊗5` eigenvalue = `(-1)ᵖ` ⇒ **parity of the 5 bits = p**.
- `YYYYY` (odd # of Y) is a **random decoy**; only even-Y Pauli strings are stabilizers (`YYXXX` etc. are deterministic) — the "which bases give deterministic outcomes" puzzle.

**Server traps (reversed from `server.py`):**

- Per-shot **measurement noise** (that's why each state gets 6 rounds × up to 5 shots) → majority-vote; `v` via **ML fit over all 10 pairwise correlations** (error-correcting), `p` via **20-shot parity majority**.
- **One `/submit` per session** (first is free, then `RATE_LIMITED`) ⇒ every one of the 40 bits must be perfect in a single shot.
- Convention gotcha: `s_vec = v[1:5]` (GHZ control + X-flips on qubits 1–4), **not** the textbook adjacent-generator XOR `vⱼ⊕vⱼ₊₁` — the adjacent reading was the first guess and failed.

**Winning line:** per state spend **2 rounds `ZZZZZ` + 4 rounds `XXXXX`** (≤6 budget), ML-reconstruct `v`, majority-vote `p`, pack with the server's own `_pack_secrets`, single `SUBMIT`. Re-verified on a fresh session.
