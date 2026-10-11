# 13 — The Debug Door

**Category:** Forensics (Cloud DFIR — AWS: API Gateway → Lambda → Secrets → IoT Core)  |  **Difficulty:** Medium

**Flag:** 21/21 questions answered (see table — the final answer is the full kill-chain reconstruction)

## How I found the flag

**Scenario:** The National Energy Operations Authority's grid-monitoring API (API Gateway + Python Lambda). IR exported CloudTrail, API Gateway + Lambda configs, API access logs, Lambda execution logs, and IoT Core MQTT logs. Reconstruct exactly how **STORM-87** found the unprotected endpoint, pulled IoT creds through Lambda, enumerated devices over HTTP, then pivoted onto the MQTT bus.

**Technique:** Pure log correlation (no exploitation). Parsed the config JSONs + 4 CSV log exports; isolated the attacker from internal `10.0.x.x` / `172.18.x.x` noise by the external **Tor exit `185.220.101.47`** (5 clustered requests), then followed that identity through every log tier.

**Kill chain:** `debug=true` leaked the config → `iot_bootstrap` made the Lambda pull the bootstrap secret from Secrets Manager → `devices`/`telemetry`/`shadow` enumerated grid devices over HTTP → attacker connected to IoT Core with the stolen creds, subscribed to all telemetry, then pushed a device-shadow update to alter transformer state.

**Answers (21/21):**

| # | Question | Answer |
|---|----------|--------|
| 1 | Route key | `GET /api` |
| 2 | Authorization type | `NONE` |
| 3 | Access-log group | `/aws/apigateway/neo-grid-api-access` |
| 4 | Lambda handler | `lambda_function.lambda_handler` |
| 5 | Execution role ARN | `arn:aws:iam::427831506294:role/service-role/neo-grid-api-role-p8n2xw4q` |
| 6 | Code last updated (UTC) | `2026-03-21T06:51:32Z` |
| 7 | Attacker source IP | `185.220.101.47` |
| 8 | Requests : response code | `5:200` |
| 9 | Debug-leak query param | `debug=true` |
| 10 | Internal action (debug) | `debug_config` |
| 11 | HTTP client library | `Python-urllib/3.12` |
| 12 | Lambda req-id (iot_bootstrap) | `83fde603-910c-4bb3-b7d4-998cb6e35b8e` |
| 13 | Billed duration (iot_bootstrap) | `286` ms |
| 14 | Internal action (enum) | `devices` |
| 15 | IoT thing (shadow over HTTP) | `substation-12-transformer-3` |
| 16 | Secret retrieved | `neo/iot-device-bootstrap` |
| 17 | Source IP of GetSecretValue | `13.215.142.77` (Lambda, boto3) |
| 18 | MQTT client ID | `storm87-pivot-01` |
| 19 | First subscribe topic | `neo/telemetry/+` |
| 20 | Publish before shadow | `neo/control/storm87` |
| 21 | Shadow-update topic | `$aws/things/substation-12-transformer-3/shadow/update` |
