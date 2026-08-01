<div align="center">

# 🏭 Manufacturing Spec Lookup

**An AI agent that opens an authoritative materials database, pulls the properties an engineer needs for a spec sheet, and reports back — then films itself doing it.**

[![CI](https://github.com/coasty-ai/coasty-manufacturing-spec-lookup/actions/workflows/ci.yml/badge.svg)](https://github.com/coasty-ai/coasty-manufacturing-spec-lookup/actions/workflows/ci.yml)
[![Node](https://img.shields.io/badge/node-%E2%89%A520.11-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Dependencies](https://img.shields.io/badge/dependencies-0-brightgreen)](package.json)
[![Runs offline](https://img.shields.io/badge/runs%20offline-%240.00-blue)](#try-it-in-30-seconds)
[![License](https://img.shields.io/badge/license-MIT-black)](LICENSE)

<img src="media/demo.gif" alt="The agent searching the NIST Chemistry WebBook and reading a substance record" width="820">

<sub>Every frame above is a **real screenshot the model actually saw** — pulled from the run's own model-input frames, not a reconstruction.</sub>

</div>

---

## What this is

A complete, production-grade [Coasty](https://coasty.ai) computer-use automation for **material and chemical property lookup**. It gives an AI agent one goal in plain English, and the agent drives a real browser on a real cloud desktop to accomplish it — no selectors, no scraping rules, no DOM parsing to maintain.

Every material spec, safety data sheet, bill of materials line and supplier qualification packet needs the same handful of facts about a substance — its formal name, its formula, its molecular weight, its CAS registry number — and they have to be right, because a transposed CAS number on a shipping document is a compliance problem, not a typo. So an engineer opens a reference database, types a name, reads four values off a record, and pastes them into a form. That loop is worth automating and it is exactly the loop that defeats a scraper: reference sites are human-facing interfaces built for reading, not feeds built for machines, and a scraper against one is a pile of selectors that breaks the first time the page is restyled. An agent is given the *goal* instead, so it reads the page the way the engineer does and keeps working when the layout moves.

**Zero dependencies. Runs offline for $0 on a fresh clone. ~$0.75 to run for real.**

```
"Go to https://webbook.nist.gov/chemistry/ and look up the substance ethanol by
 its CAS registry number, 64-17-5. Open the matching substance record and read
 the general information NIST lists for it. Then report exactly four values: the
 IUPAC name, the molecular formula, the molecular weight in g/mol, and the CAS
 registry number printed on the record. Also state whether that CAS registry
 number matches the 64-17-5 you searched for. Stop once you have reported those
 four values and the match verdict."
```

That prompt *is* the automation. When the site redesigns, the prompt still works. Point it at a different substance — or a different CAS number — by editing one string.

## Try it in 30 seconds

No API key. No account. No install. No spend.

```bash
git clone https://github.com/coasty-ai/coasty-manufacturing-spec-lookup
cd coasty-manufacturing-spec-lookup
npm start
```

That boots a bundled offline mock in-process and runs the whole agent loop against it. Then render the demo video from the run's own frames:

```bash
npm run demo     # needs ffmpeg; writes media/demo.mp4 + demo.gif + poster.jpg
```

Check your setup any time with `npm run doctor`.

## Run it for real

```bash
export COASTY_API_KEY=sk-coasty-test-...      # sandbox keys never bill
export COASTY_BASE_URL=https://coasty.ai/v1
export COASTY_ALLOW_LIVE=1                     # destination consent
npm start -- --live --confirm-cost-cents 120   # cost consent
```

Both consents are required and they are deliberately separate. A live key alone will not spend; a base URL alone will not spend. See [Safety](#safety).

| | |
|---|---|
| Expected cost | **75¢** (15 steps × 5 credits) |
| Worst case | **120¢** (24-step cap) |
| Model-input frames | **free** |
| Machine runtime | Coasty provisions and destroys its own VM |

`npm run estimate` prints this before anything runs.

The target — the NIST Chemistry WebBook — is a public reference database. This automation reads it with no account and no credentials of any kind.

## How it works

```
POST /v1/tasks                          Coasty provisions its own ephemeral VM,
                                        drives the agent, and destroys the VM
GET  /v1/runs/{id}                      poll to a terminal state
GET  /v1/runs/{id}/screenshots          the exact frames the model saw — free
GET  /v1/runs/{id}/events               per-step narration (SSE)
ffmpeg                                  frames → demo.mp4 + demo.gif + poster
```

The demo video is a **byproduct of running the automation**, not a separate artifact to author and keep in sync. There is no storyboard, no HTML mock, and nothing that can drift from reality — if the agent did something different, the video shows something different.

Verification is intrinsic and runs without a human watching:

```
✓ frames captured              15 frames
✓ frame count matches steps    15 frames vs 15 steps
✓ not all frames degraded      0 degraded
✓ frames are distinct          15/15 unique
✓ duration matches pacing      10.20s vs 10.20s expected
✓ stream width correct         1280x720
✓ video is non-trivial         306 packets
```

## Safety

This repo is built so that **accidental spend is structurally impossible**, not merely discouraged:

- **Fail-closed destination.** An unset `COASTY_BASE_URL` resolves to the bundled offline mock. Production is never a default.
- **Two independent consents.** `COASTY_ALLOW_LIVE=1` authorises the *destination*; `--confirm-cost-cents N` authorises the *cost*, and N must equal the server-computed worst case exactly.
- **Idempotency by default.** The submit key is derived from the prompt, so a retried submit returns the original run instead of provisioning a second machine.
- **A hard cap per unit.** A worst case above `capCents` in [`automation.json`](automation.json) is refused before any request is made.
- **No credentials, ever.** This automation targets a public site. Nothing here reads a password, a token, or a cookie.

## Project layout

```
automation.json      the entire unit definition — prompt, target, budget, caps
src/client.mjs       Coasty client: fail-closed target, retry, idempotency
src/capture.mjs      model-input frames → mp4/gif/poster, with sanity checks
src/cli.mjs          run · demo · estimate
tools/mock.mjs       the bundled offline Coasty (real 1280×720 PNG frames)
tools/doctor.mjs     preflight
test/                25 tests, zero dependencies, fully offline
```

Adding a new automation is one `automation.json` and one prompt — `src/` never forks. See [AGENTS.md](AGENTS.md) for the authoring contract used by Claude Code and Codex.

## Tests

```bash
npm test     # node --test, no install, no network, no key
```

## Related

Part of the **Coasty automation catalog** — production-grade computer-use automations across 12 industries. See [the index](https://github.com/coasty-ai) for finance, healthcare, legal, logistics, energy, public sector, HR, education, retail, nonprofit and e-commerce.

- [Coasty docs](https://coasty.ai/docs) · [API reference](https://coasty.ai/docs/llms.txt)
- [computer-use-cookbook](https://github.com/coasty-ai/computer-use-cookbook) — the API, by endpoint, in 4 languages
- [open-cowork](https://github.com/coasty-ai/open-cowork) — the open-source AI coworker

## License

MIT © Coasty
