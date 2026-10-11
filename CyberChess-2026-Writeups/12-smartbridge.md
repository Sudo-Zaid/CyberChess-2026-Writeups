# 12 — SmartBridge

**Category:** ICS / Web (KNX/IP cloud bridge: SSRF → stolen mTLS cert → cEMI passthrough)  |  **Difficulty:** Insane

**Flag:** `HTB{cl0ud_tru5t_15_bu5_tru5t_l3g4cy_p455thr0ugh_w1ns}`

## How I found the flag

**Scenario:** Helia's cloud bridge fronts the Central Government District's **KNX/IP** building-automation bus — everything from lobby lighting to secure-wing access control. The bus's **audit canary** holds a ticket nobody outside the integrator should be able to claim. Three services: **BMS HMI** (read-only dashboard, polls `/api/state`), **Helia bridge portal** (docs + `/tools/preview`), and the **MQTT broker** (mqtts, mutual-TLS).

**Architecture (reversed from the portal docs):** cloud → edge → bus. Actuation flows as **AWS-IoT-shadow** `state.desired` documents published over **mqtts** to `$aws/things/cgd-knx-bridge-001/shadow/update`; the bridge validates them against a GA allowlist and forwards **cEMI `L_Data.req`** frames to **KNXnet/IP on `127.0.0.1:3671/udp` (loopback only)**. Three trust layers: mTLS client cert (CN → per-thing topic ACL), GA allowlist (**mains 4 life-safety / 5 access-control BLOCKED**), and the secure-wing interlock.

**Chain (all four layers broken):**

1. **SSRF filter bypass** — `/tools/preview` fetches a URL *from the bridge box*, but a resolve-then-check filter blocks `localhost` / `127.0.0.1` / decimal `2130706433` / `0.0.0.0` / `[::1]` / IPv4-mapped `[::ffff:127.0.0.1]`. The box has **no outbound DNS**, so redirect/DNS-rebind are dead. Bypass = **alternative IPv4 encodings** that `ipaddress` rejects as literals (so the loopback check is skipped) but `requests`/`getaddrinfo` still resolves to 127.0.0.1: **`http://127.000.000.001:8081/...`** (also octal `0177.0.0.1`, hex `0x7f.0.0.1`).
2. **Steal the mTLS identity** — SSRF to the internal cert minter `http://127.000.000.001:8081/internal/cert/issue?thing=cgd-knx-bridge-001`; the preview echoes the JSON response = **client cert + private key + CA** for the thing ("treat the thing certificate as the master key to the building").
3. **mTLS to the broker** — the broker enforces client certs (plain connect → `TLSV13_ALERT_CERTIFICATE_REQUIRED`). Raw MQTT CONNECT over TLS presenting the stolen cert → `CONNACK rc=0` (client-id = `cgd-knx-bridge-001` to match the ACL).
4. **`legacy_payload` raw-cEMI passthrough** — commissioning_mode left `true` (go-live default). The shadow's `desired.legacy_payload` carries a hex cEMI frame that the bridge forwards **verbatim, without the GA allowlist** → drives the blocked main-5 access-control GAs.

**Interlock (HEL-CGD-503) — three stages, not a bare write:** state at `5/0/98` (0 idle / 1 armed / 2 tripped) → **arm** = *long-frame* cEMI write of the 14-byte ASCII magic **`AUDIT-CANARY-1`** to `5/0/99` (DPT 16.001, NPDU = payload_len+1, APCI byte stands alone) → **release** = *short-frame* GroupValueWrite `1` to `5/0/100` while armed (release-while-not-armed is silently ignored). GA 3-level → 16-bit: `(main<<11)|(mid<<8)|sub`, so `5/0/99 = 0x2863`, `5/0/100 = 0x2864`.

**Winning line:** publish two shadow updates over the one mTLS MQTT session —

- arm: `legacy_payload = 1100bce0000028630f008041554449542d43414e4152592d31`
- release: `legacy_payload = 1100bce000002864010081`

→ mantrap `armed → tripped`, door `LOCKED → UNLOCKED`, `audit_ticket_released: true`; the ticket surfaced in the HMI `/api/state`.
