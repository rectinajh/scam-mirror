# Examples

## Dry-run demo (no real call)

```bash
python3 scripts/run_dual_fly.py \
  --number "+1 555 010 0100" \
  --org "Example Bank"
```

The fixture makes the suspect fly detect `asks_for_otp` and `demands_secrecy`,
so the verdict is `likely_scam`. Output is printed to stdout and written to
`results/`.

## Override the official number

```bash
python3 scripts/run_dual_fly.py \
  --number "+1 555 010 0100" \
  --org "Example Bank" \
  --official "+1 800 555 0100"
```

## Live run

```bash
cp .env.example .env
# set CALLE_API_KEY in .env
set -a; . ./.env; set +a

python3 scripts/run_dual_fly.py \
  --number "<real E.164 suspect number>" \
  --org "Example Bank" \
  --region "US" \
  --live
```

`--live` places two real outbound calls. Replace the whitelist placeholder with
a real published official line first.

## Sample output (abbreviated)

```json
{
  "verdict": "likely_scam",
  "confidence": 0.83,
  "signals": [
    "asked for a one-time passcode or verification code",
    "demanded secrecy from family or authorities",
    "detected 2 high-risk request(s)"
  ],
  "official_fly": {
    "number_called": "+1 800 555 0100",
    "reached": true,
    "summary": "The official line confirmed Example Bank does not operate an 'account frozen' callback flow...",
    "content_hash": "sha256..."
  },
  "suspect_fly": {
    "number_called": "+1 555 010 0100",
    "reached": true,
    "red_flags": ["asks_for_otp", "demands_secrecy"],
    "content_hash": "sha256..."
  }
}
```

