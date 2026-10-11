# 01 — Operation Phantom Runner

**Category:** OSINT  |  **Difficulty:** Easy

**Flag:** `mountain_phoenix_runner` (final answer — the criminal's fitness alias)

## How I found the flag

**Scenario:** Map a criminal network ("Alejandro Martinez") hidden inside four fitness-tracking apps — **FitTracker Pro**, **FitSocial Network**, **Phoenix Fitness** (corporate wellness) and **RouteMapper Analytics** (all Vite/React SPAs).

**Technique:** All data was **hard-coded in the client JS bundles** (`/assets/index-*.js`). Downloaded all 4 bundles and grepped the embedded JSON for the entities — no backend/API needed.

**Answers (6/6):**

| # | Question | Answer | Source |
|---|----------|--------|--------|
| 1 | Logistics manager | `David_Torres` | FitSocial `occupation:"Logistics Coordinator"` (`backup_runner_23`); Phoenix role "Maritime logistics" |
| 2 | Phase 2 start date | `07/07` | FitSocial event "Night Training Session — Jul 7" tagged group-chat `"Phase 2 logistics ready"` |
| 3 | Corporate front org | `Phoenix Fitness` | wellness platform brand (`"Phoenix Holdings International Ltd."` = registered parent, was REJECTED) |
| 4 | Secure comms manager | `Marcus_Chen` | `occupation:"Communications Specialist"` (`secure_comm_rider`) |
| 5 | Surveillance coords | `49.2827N_123.1207W` | Burnaby Industrial Complex (RouteMapper `coordinates:{lat:49.2827,lng:-123.1207}`, owner = Phoenix Holdings) |
| 6 | Criminal's fitness alias | `mountain_phoenix_runner` | Alejandro "A. Martinez", employeeId PH-001, "Network coordinator" |
