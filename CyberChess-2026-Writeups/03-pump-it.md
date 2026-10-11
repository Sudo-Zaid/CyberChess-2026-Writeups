# 03 — pump it

**Category:** Crypto  |  **Difficulty:** Hard

**Flag:** `HTB{merkle_damguard_based_hash_functions_are_vulnerable_to_hash_length_extension_attacks!_45b226d534761f7a9d5845c1424531af}`

## How I found the flag

**Target:** Flask app. Custom JWT where signature = `base64(SHA-512(secret ‖ "header.payload"))`, but `check_admin` verifies the sig over **just** `base64decode(payload)` and requires the last `&`-field to be `is_admin=True`.

**Bug:** Merkle–Damgård **hash length extension**. The `secret` length = **23** (visible in source `os.urandom(23)`).

- `/register` → get a legit token → recover `H(secret ‖ M)` from its sig (base64-decoded raw digest).
- Use `hlextend` (sha512) to compute `H(secret ‖ M ‖ glue ‖ "&is_admin=True")` with `keylen=23`.
- Craft token = `b64(hdr).b64(M‖glue‖"&is_admin=True").b64(new_digest)` → hit `/dashboard` as admin.
