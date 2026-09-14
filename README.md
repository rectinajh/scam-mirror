# Scam Mirror (骗局镜面)

A CALL-E powered **dual-fly verifier** for suspected callback scams. Drop in a
suspicious "call this number back" number and the organization it claims to be,
and Scam Mirror releases two short-lived "flies": one calls the published
official line, the other probes the suspect line. It compares what each one
actually says and returns a verdict plus an evidence hash. No identity data is
collected, and full recordings are not stored.

> 用官方模拟漏洞，照出骗局镜面。 — Use the official analog hole to hold up a
> mirror to the scam.

## One-liner

Cyber-fruit-fly truth swarm: a user hands over a suspicious callback number or a
"this is your bank/police/courier" call, and two disposable flies dial the
published official hotline and the suspect number, compare their stories, and
emit only a trust verdict and evidence hashes.

## Background (背景)

Callback scams — "this is your bank / police / courier, please call back this
number" — weaponize the one channel people still instinctively trust: a live
phone call. A fake webpage can be spotted, but a confident voice on a spoofed
number is hard for a non-specialist to challenge. The institutions being
impersonated publish real hotlines; the problem is that nobody calls both sides
and compares what they say.

CALL-E changes that equation: an agent can now place real outbound calls, so
verification stops being "search the number on a forum" and becomes a
reproducible, evidence-backed workflow.

## Core problem (核心问题)

When a person receives a suspicious callback or an "official" caller-ID claim,
there is no fast, trusted way to answer one question: is this number really the
organization it claims to be? The usual options are slow (hold queues), noisy
(forum threads), or inconclusive (a web search that cannot hear the caller's
script).

## What problem it solves (解决什么问题)

Scam Mirror turns that verification into a single agent run:

- It dials the published official line and the suspect line, and compares what
  each actually says.
- It returns `likely_legit | likely_scam | inconclusive` with confidence,
  signals, and evidence hashes instead of a gut feeling.
- It does this while collecting no identity and storing no full recording.

## How it solves (如何解决)

1. **Input** — suspect number, claimed organization, optional script details and
   region.
2. **Resolve the official contact** — look up the published hotline from
   `orgs.whitelist.json` (or a user-supplied official number).
3. **Release two flies**
   - **Fly-A (official)**: dials the official hotline and asks, in minimal
     words, whether a "callback / case / freeze" flow actually exists for that
     number.
   - **Fly-B (suspect)**: dials the suspect number and records red flags such as
     requests for codes, transfers, remote control, or threats.
   - Both flies carry a TTL, a max call length, and a kill switch (any request
     for a code or transfer ends the call immediately and marks it high risk).
4. **Compare and judge** — rule-based heuristics (+ optional LLM summary
   comparison) produce `likely_legit | likely_scam | inconclusive`.
5. **Emit a JSON attestation** — verdict, confidence, signals, per-fly summary,
   and content hashes. Optionally push to a webhook or write to `results/`.

## Innovations (创新点)

- **Cyber-fruit-fly**: verification flies are short-lived, replaceable, and
  disposable. They can rotate numbers and scripts, then burn out instead of
  becoming a spammer.
- **Cypherpunk minimal disclosure**: no real names, no IDs, no bank cards. Logs
  contain only the verdict, a summary, and a content hash — not full recordings.
- **The analog hole, used defensively**: instead of trusting one caller, the
  project forces the story to survive a live comparison against the official
  line's own spoken answer.
- **No single source of truth**: the official page, the official line, and the
  call summaries are cross-checked; a wrong whitelist entry degrades to
  `inconclusive`, not a false accusation.

## Real cyber-fruit-fly grounding (真实的赛博果蝇)

The flies are not just a metaphor. Each persona is anchored to a real neuron in
  the **male Drosophila CNS connectome** (FlyEM / HHMI Janelia, CC-BY), about
166,000 neurons and 125 million synapses:

- Fly-A (official verifier) → **Giant Fiber (DNp01)**, the escape-command neuron.
- Fly-B (suspect probe) → **DA1_lPN**, a pheromone-detecting olfactory
  projection neuron.
- Reserve persona → **MBON11**, a mushroom-body output neuron for learned
  avoidance.

The mapping, with real `body_id`, cell type, and Virtual Fly Brain `vfb_id`, is
in `skills/scam-mirror/flies.registry.json` and is embedded in every
attestation's `flies[].persona`.

Source: <https://male-cns.janelia.org/download/>
Citation: "Sexual dimorphism in the complete Drosophila male central nervous
system connectome" (FlyEM, 2026).

## Repo layout

```text
.
├── README.md              # this file
├── PRD.md                 # product requirements
├── docs/TECHNICAL.md      # architecture + CALL-E integration
└── skills/scam-mirror/    # the reusable Agent Skill (PR target)
    ├── SKILL.md
    ├── references/safety.md
    ├── references/examples.md
    ├── scripts/run_dual_fly.py   # runnable CLI
    ├── flies.registry.json       # real Male CNS connectome neuron personas
    ├── .env.example
    ├── orgs.whitelist.json
    └── results/example-result.json
```

## Quickstart (dry-run, no real call)

The CLI defaults to dry-run: it uses bundled fixtures and never dials a real
number. It needs no dependencies (Python 3.10+ standard library only).

```bash
python3 skills/scam-mirror/scripts/run_dual_fly.py \
  --number "+1 555 010 0100" \
  --org "Example Bank"
```

This prints a full attestation and writes it to
`skills/scam-mirror/results/`. The demo fixture produces
`verdict: likely_scam` because the suspect fly detects
`asks_for_otp` and `demands_secrecy`.

## Live CALL-E setup (CLI)

1. Install the pinned CALL-E CLI and authorize once with your browser:

   ```bash
   cd skills/scam-mirror/scripts
   npm install
   ./node_modules/.bin/calle auth login
   ```

   `auth login` prints a one-time authorization URL. Open it, approve, and the
   CLI caches your token locally.

2. Verify the integration:

   ```bash
   ./node_modules/.bin/calle auth status
   ./node_modules/.bin/calle mcp tools
   ```

3. Replace the fictional official numbers in `orgs.whitelist.json` with real,
   published hotlines, then run the dual-fly flow:

   ```bash
   cd ../..
   python3 skills/scam-mirror/scripts/run_dual_fly.py \
     --number "<E.164 suspect number>" \
     --org "Example Bank" \
     --live
   ```

`--live` uses the CALL-E CLI (`--backend cli`) by default. A REST Developer API
path is also available with `--backend rest` plus `CALLE_API_KEY` (see
`.env.example`). Real calls are side effects; only run them for a verification
you personally initiated.

## Official number configuration

`orgs.whitelist.json` maps organizations to published hotlines by region:

```json
{
  "version": 1,
  "orgs": [
    {
      "name": "Example Bank",
      "aliases": ["example bank", "examplebank"],
      "regions": { "US": "+1 800 555 0100" }
    }
  ]
}
```

The shipped numbers are fictional (North America reserved `555-01xx` range).
Replace them with verified published lines before any live run. You can also
override on the command line with `--official`.

## Verdict heuristics

High risk (leans `likely_scam`):

- asks for a verification code, password, PIN, immediate transfer,
  cryptocurrency, remote control, or secrecy from family
- claims to be police/prosecutor but asks you to call a non-official number
- threatens "arrest/freeze now" and pushes a same-second transfer

Leans `likely_legit`:

- the suspect number matches the published official number after normalization
- the official hotline confirms the flow exists and the accounts match

`inconclusive`:

- call did not connect, IVR was too deep, the official line did not confirm, or
  there was not enough information

## Example result

See [skills/scam-mirror/results/example-result.json](skills/scam-mirror/results/example-result.json)
for a complete attestation in the required schema.

## Safety & ethics

- Only for user-initiated anti-fraud verification. No mass dialing, no untargeted
  number scanning.
- Suspect calls are kept short; any request for secrets ends the call
  immediately. The flies never provide real personal information.
- Official calls are polite, short, and non-harassing; the same official line
  should be rate-limited.
- Full recordings are not stored by default; only summaries and hashes are kept.
- The output is an aid to judgment, not a legal conclusion. UI and copy must say
  so.

See `skills/scam-mirror/references/safety.md` for the full safety contract.

## Docs

- [PRD.md](PRD.md) — product requirements and demo storyboard
- [docs/TECHNICAL.md](docs/TECHNICAL.md) — architecture, CALL-E integration,
  schema, and extension points

## License

MIT. See [LICENSE](LICENSE).
