# 14 — Unfinished Business

**Category:** Forensics (Memory Forensics / .NET ransomware RE — Back to the Future)  |  **Difficulty:** Hard

**Flag:** `HTB{4n0th3r_0n3_T4nn3n_d3f34t3d}`

## How I found the flag

**Scenario:** "Mad Dog" Tannen's ransomware re-encrypts Marty's files on every reboot. Given a 1 GB memory dump (`memory.raw`) + the encrypted `big_plan.pdf.enc`; recover the key and read Mr. Brown's escape plan.

**Technique:** **Volatility 3** + raw-memory carving + crypto reconstruction.

- `windows.info` → **Win7 SP1 x86**; `windows.registry.printkey` on `HKCU\...\CurrentVersion\Run` → persistence **`Rans = C:\Users\Marty\Downloads\Rans.exe`** (runs each boot, already exited).
- `Rans.exe` is a **.NET Core 3.1 apphost**; the real logic is in **`Rans.dll`**. Carved the managed metadata straight out of the dump (`#Strings`/`#US` heaps): refs `Aes`, `set_Key`, `set_IV`, `CreateEncryptor`, `MD5`, `ComputeHash`, `Encoding.get_Unicode`, targets Documents, Office extensions, appends `.enc`.
- **Crypto:** AES-128-CBC, **key = `MD5(Encoding.Unicode.GetBytes("32xDrd6kBp4gJjOw"))`** (the 16-char literal was a UTF-16 user-string in the dump). Decrypting with a zero IV gave a clean PDF from block 2 on; XOR-recovering block 0 against the known `%PDF-1.` header spelled **`chicken…`**, and the full 16-byte IV **`chicken Marty!!!`** ("Nobody calls me chicken!") was sitting in memory as ASCII.
- Decrypted `big_plan.pdf.enc` → valid 5-page PDF ("Go Back To The Future" plan: Delorean → steam locomotive → 88 mph → flag).
