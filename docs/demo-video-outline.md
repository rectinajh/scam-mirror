# Scam Mirror — Demo Video Outline (~3 min)

## Goal

Show a real, reproducible anti-fraud verification: input a suspect callback
number, watch two CALL-E "flies" compare the official line against the suspect
line, and get a verdict plus an evidence hash. Ground the metaphor in real
Drosophila connectome neurons.

## Timeline

### 0:00-0:20 — Hook

- Title card: **Scam Mirror (骗局镜面)**.
- One line: "Use the official analog hole to hold up a mirror to the scam."
- Show [assets/cyber-flies.png](../assets/cyber-flies.png) (real neuron render).

### 0:20-0:50 — Problem

- Voice: "This is your bank. Your account was compromised. Please call back
  this number."
- The gap: a normal person has no fast, trusted way to check that callback
  number. Search results cannot hear what the caller actually says.

### 0:50-1:30 — The dual-fly mechanism

- Diagram: Fly-A dials the published official hotline; Fly-B dials the suspect
  number; the two summaries are compared.
- Real personas:
  - Fly-A = Giant Fiber (DNp01), the escape-command neuron.
  - Fly-B = DA1_lPN, a pheromone-detecting olfactory projection neuron.
- Emphasize: short-lived, disposable flies; kill switch on secret requests.

### 1:30-2:20 — Live run

- First, dry-run (no call), then `--plan` against the real CALL-E service.
- If a supported-region number is available, run `--live`.
- Screen shows the verdict: `likely_scam`, the `red_flags`
  (`asks_for_otp`, `demands_secrecy`), and the `verdict_hash`.

### 2:20-2:45 — Safety & ethics

- Short calls; hang up the moment the callee asks for a code or transfer.
- No real personal information is shared; no full recordings are stored.
- Output is advisory, not a legal conclusion.

### 2:45-3:00 — Close

- Closing line: "用官方模拟漏洞，照出骗局镜面。"
- Call to action: GitHub repo, PR, and the attestation schema.

## Capture list

- `python3 skills/scam-mirror/scripts/run_dual_fly.py --number "+1 555 010 0100" --org "Example Bank"` (dry-run)
- `... --plan` (real service, no dial)
- `... --live` (real call, when a supported-region number is available)
- `skills/scam-mirror/results/example-result.json` (attestation)
- `assets/cyber-flies.png` (neuron grounding)

## Narration beats (draft)

1. "Scammers can fake a webpage, but they have to survive a real phone call."
2. "Scam Mirror calls both sides of the story and compares what they actually say."
3. "The flies are named after real neurons in the male fruit fly connectome."
4. "It stores the verdict and a hash, not your identity or the recording."

