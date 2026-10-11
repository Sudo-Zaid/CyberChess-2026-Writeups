# 15 — Gradient Heist

**Category:** AI / ML (gradient leakage from a zeroed parameter)  |  **Difficulty:** Medium

**Flag:** `HTB{gr4d13nt5_sp1ll_53cr37s}`

## How I found the flag

**Scenario:** Neo-Yokado's engineers swear their Oracle "leaks nothing" — *"read its tells and prove them wrong."* Given a trained classifier `challenge_model.pt` (TinyNet, 1×32×32 input, 10 classes) and a `distilled_batch.pt` (4 synthetic images + labels). Recover the secret the model spills.

**Technique:** The net adds a hidden per-class logit offset `(offsets_weight @ context_vector) · temperature`, and ships with **`context_vector = 0`** — so the offsets are inactive and the model "leaks nothing". But the `offsets_weight` buffer (10×64) is **rank-1**: every row is one shared 64-vector `base` times a per-class scalar, and only its **first 28 columns** are nonzero (28 = flag length).

- Backprop cross-entropy on the distilled batch: `∂L/∂context_vector = temperature · offsets_weightᵀ · r = base · scalar` — the gradient w.r.t. the *zeroed, "leaks-nothing"* parameter is **proportional to `base`** (the "tell").
- The distilled batch is crafted so that scalar = 1 exactly → **`grad = −ASCII(flag)`**. Take `abs()` → read bytes → flag.
- Two independent confirmations: (a) one `loss.backward()` gave `context_vector.grad = [-72,-84,-66,-123,-103,-114,-52,-100,...]` → `H T B { g r 4 d ...`; (b) direct decode `offsets_weight[0] · 11/32` (since `base_j = ASCII_j · 32/11`, temperature = 1/32) yields the identical string.
