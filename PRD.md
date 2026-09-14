# Scam Mirror — Product Requirements Document

- Status: DRAFT (CALL-E hackathon)
- Mode: Builder
- Repo: scam-mirror

## 1. Summary

Scam Mirror is a reusable phone-call Agent Skill that verifies a suspected
callback scam by dialing two numbers: the published official hotline of the
claimed organization and the suspect number. It compares what each line says and
returns a trust verdict plus evidence hashes, without collecting the user's
identity or storing full recordings.

## 2. Problem

"This is the police / bank / courier / platform, please call back this number."
An ordinary person has no fast way to check that claim. Existing responses are
manual (search for the number, read forum threads) or slow (waiting on hold with
the real institution). Scam Mirror makes verification reproducible and
evidence-backed.

## 3. Users

- **Primary**: an individual who received a suspicious callback or caller-ID
  claim and wants to know whether to comply.
- **Secondary**: a family member or support worker helping a less-technical
  person verify a number.

## 4. Goals and non-goals

Goals:

- Return `likely_legit | likely_scam | inconclusive` with confidence and evidence
  hashes.
- Use CALL-E to place real outbound calls at runtime.
- Ship as a portable Agent Skill that can be PR'd into
  `CALLE-AI/awesome-phone-call-agents`.
- Minimize stored data to verdict + summary + hash.

Non-goals (explicitly out of scope):

- Ghost-desk inbound front desk (this project is outbound verification only).
- Bulk dark-pattern dialing or pressure-style mass calls against institutions.
- Collecting ID numbers, bank cards, or SMS verification codes.
- A legal or authoritative conclusion; output is advisory.

## 5. User flow

1. User provides: suspect number, claimed organization, optional script details,
   optional region.
2. System resolves the official contact from the whitelist or a user-supplied
   number.
3. Fly-A calls the official line; Fly-B calls the suspect line. Both have a TTL,
   max duration, and sensitive-action kill switch.
4. Rule (+ optional LLM) comparison yields a verdict.
5. A JSON attestation is emitted (stdout, `results/`, or webhook).

## 6. Functional requirements

- **FR-1 Input**: accept suspect number (E.164), claimed organization, optional
  script notes and region.
- **FR-2 Official resolution**: resolve the published hotline from
  `orgs.whitelist.json`, or accept an explicit `--official` override.
- **FR-3 Fly-A (official)**: confirm whether a "callback / case / freeze" flow
  exists for the suspect number; return `reached`, `summary`,
  `confirmed_business`.
- **FR-4 Fly-B (suspect)**: probe the suspect number; return `reached`,
  `summary`, and `red_flags`.
- **FR-5 Verdict**: emit `likely_legit | likely_scam | inconclusive` with
  confidence in `[0,1]`.
- **FR-6 Attestation**: emit the required JSON schema (see section 9).
- **FR-7 TTL / kill switch**: both flies carry `ttl_hours`; any request for
  codes, passwords, transfers, remote control, crypto, or secrecy ends the call
  immediately.
- **FR-8 Dry-run**: a no-call fixture path that is on by default.

## 7. Non-functional requirements

- **Privacy**: no real names, IDs, or full recordings by default.
- **Safety**: no mass dialing; rate-limit the same official line.
- **Reproducibility**: the attestation is deterministic from its inputs and
  carries hashes.
- **Portability**: standard-library-only CLI; the skill folder is self-contained.

## 8. Verdict heuristics

See the "Verdict heuristics" section of `README.md` and the rule table in
`docs/TECHNICAL.md`. Rules are the minimal slice; an LLM summary comparison is an
optional extension.

## 9. Output schema

```json
{
  "case_id": "string",
  "suspect_number": "string",
  "claimed_org": "string",
  "verdict": "likely_legit | likely_scam | inconclusive",
  "confidence": 0.0,
  "signals": ["string"],
  "official_fly": {
    "number_called": "string",
    "reached": true,
    "summary": "string",
    "content_hash": "sha256..."
  },
  "suspect_fly": {
    "number_called": "string",
    "reached": true,
    "summary": "string",
    "red_flags": ["string"],
    "content_hash": "sha256..."
  },
  "attestation": {
    "timestamp": "ISO-8601",
    "verdict_hash": "sha256...",
    "note": "one-line conclusion"
  },
  "flies": [
    { "id": "A|B", "generation": 1, "ttl_hours": 1, "persona_id": "string" }
  ]
}
```

## 10. Safety & ethics

- User-initiated anti-fraud verification only.
- Suspect calls: shortest possible; hang up on secret requests; never supply
  real personal information.
- Official calls: polite, brief, non-harassing; rate-limited.
- Default: no full recordings, only summary + hash.
- Output is advisory, not a legal conclusion.

## 11. Demo storyboard (~3 min)

1. Show input: a suspect number claiming to be "bank fraud control" with the
   script "your account was compromised, call back".
2. Fly-A dials the official hotline (or the demo fixture); the official summary
   appears.
3. Fly-B dials the suspect number; `red_flags` appear (asks for a code, etc.).
4. The screen prints `verdict: likely_scam` plus the attestation hash.
5. Closing line: "Use the official analog hole to hold up a mirror to the scam."

## 12. Success metrics (hackathon)

- The dual-fly flow runs end-to-end in dry-run without a real call.
- A live run can place two real CALL-E calls and emit a schema-valid attestation.
- The skill folder passes the `awesome-phone-call-agents` repository validation.
- A PR is opened into `skills/scam-mirror/` in the correct contribution area.

## 13. Roadmap

- **v0 (minimal slice, now)**: one hardcoded official number + one suspect call
  + verdict JSON.
- **v1**: whitelist of 3-5 organizations and the sensitive-action kill switch.
- **v2**: optional LLM summary comparison, webhook output, and rate limiting.
- **v3**: minimal web UI that always renders the advisory disclaimer.

## 14. Premises

1. Voice is the right analog hole: a scam's story must survive a real spoken
   comparison against the official line.
2. Minimal disclosure is a product feature, not a limitation: hashes + summaries
   are enough evidence for a reproducible verdict.
3. A disposable, short-lived fly is safer than a persistent dialer and avoids
   becoming a harassment tool.
4. The official number must be trustworthy; a wrong whitelist entry degrades to
   `inconclusive` rather than a false accusation.

## 15. Approaches considered

- **A — Self-contained Agent Skill (chosen)**: one `skills/scam-mirror/` folder
  with SKILL.md, references, and a stdlib CLI. Best fit for the PR target and
  the "reusable skill" requirement.
- **B — Bare CLI**: fewer files but not an installable skill, so it fails the
  hackathon's "reusable Agent Skill" bar.
- **C — Demo web app**: nice for judges but larger surface and not what the
  "minimal slice first" instruction asks for.

