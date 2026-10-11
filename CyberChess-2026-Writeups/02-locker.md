# 02 — Locker

**Category:** Reversing (Android / Mobile RE)  |  **Difficulty:** Medium

**Flag:** `actually_the_content_is_not_that_important_but_just_submit_this_on_the_question_:D` (decrypted plaintext — the final answer)

## How I found the flag

**Scenario:** Reverse an Android ransomware APK (`com.ctf.locker`), understand the crypto, decrypt the locked file and find the unlock protocol.

**Technique:** Extracted `base.apk`, decompiled with **androguard** (`AnalyzeAPK`). Classes: `CrypterManager`, `PaymentService`, `FileEncryptionService`, `LockScreenActivity`.

- `CrypterManager`: **AES/CBC/PKCS5Padding**, key = `SHA-256("L0ck3rK3y$@lt#2024")` (AES-256, device-independent!), `.lbyte` layout = `IV(16) ‖ ciphertext`.
- `PaymentService.generateUnlockPayload()` = `System.currentTimeMillis() + "|" + deviceId + "|" + nonce`.
- C2 endpoint `http://locker.htb/api/v1/verify`.
- Decrypted the vault file locally with pycryptodome (key is a fixed plant constant).

**Answers (5/5):**

| # | Question | Answer |
|---|----------|--------|
| 1 | Fields in unlock payload | `timestamp+deviceId+nonce` |
| 2 | Encryption mode + algo | `AES/CBC/PKCS5Padding` (AES-256-CBC) |
| 3 | Device ID (device_info.txt) | `604dc9b92f2685b6` |
| 4 | Unlock endpoint path | `/api/v1/verify` |
| 5 | Plaintext of encrypted file | `actually_the_content_is_not_that_important_but_just_submit_this_on_the_question_:D` |
