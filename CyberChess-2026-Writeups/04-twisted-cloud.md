# 04 — twisted cloud

**Category:** Crypto  |  **Difficulty:** Hard

**Flag:** `HTB{always_read_the_documentation_before_using_built_in_crypto_functions_313f4ee26751b5ca5f12accd092dbe17}`

## How I found the flag

**Target:** PHP "Twisted Cloud" file app. On first session it does `mt_srand(mt_rand(0, 0x13333337))`, then builds the **HTBAdministrator** password from 10× `mt_rand()%62`, then creates an avatar `avatar_<mt_rand()>.jpg` (the **11th** output of the same reseeded stream). The flag is shown in `/show` to `HTBAdministrator`.

**Bug:** Predictable Mersenne Twister. The avatar filename **leaks output #11** → brute-force the small reseed space (0..0x13333337 ≈ 322M) to find the seed whose 11th MT output equals the leaked avatar number, then regenerate outputs #1–10 → the admin password.

- Registered a user → `/show` leaked `avatar_<N>.jpg`.
- Implemented PHP-MT19937 and brute-forced 322M seeds with **numba (8 cores, ~88 s)** matching the 11th tempered output.
- Recovered password (e.g. `N2kFbcDw61`) → login as HTBAdministrator → flag.
