---
name: system-design-master
description: This skill should be used when the user asks to "design a system", "design <a product>" (e.g. "design WhatsApp", "design a URL shortener"), "write the design doc", "full system design in markdown", "HLD and LLD", "HLD/LLD with mermaid", or wants FR/NFR, estimates, HLD, LLD, trade-offs, and failure modes captured as one single MD file. Use it whenever a design problem statement must be solved as one complete markdown document with small mermaid diagrams, even if they don't say "system-design-master".
---

# System Design Master (single-MD output)

Solve a given design problem statement as **one single markdown file** — from vague prompt to justified, stress-tested HLD + LLD — **by reasoning outward from stated assumptions, never by recalling a finished diagram.**

Core stance:

> Do not memorize architectures; learn the forces that shape them. Every component must tie to a stated FR, NFR, or number — anything else is deleted. Every architecture is a hypothesis with a stated breaking point.

This skill owns the *method and the output contract*. The component recipes are **not** inlined here — they live in the building-block `SKILL.md` files under `skills/`, which are the **source of truth** for designing and taking decisions.

## Source of truth rule (blocking)

When a concern becomes active, **read the owning `skills/<block>/SKILL.md` and apply its recipe** — options, trade-offs, stress behavior, numbers. Do not paraphrase from memory, do not re-teach its internals.

Building blocks (all source of truth, read on demand):

| Concern | Read |
|---|---|
| Features / scope / clarifying questions | `skills/requirements-scoping/SKILL.md` |
| QPS / storage / bandwidth / servers | `skills/back-of-the-envelope/SKILL.md` (+ `scripts/botec.py` for exact check) |
| Endpoints, pagination, idempotency, errors | `skills/api-design/SKILL.md` |
| Monolith vs microservices, boundaries, gateway | `skills/service-decomposition/SKILL.md` |
| SQL vs NoSQL, keys, indexes, sharding, replication | `skills/data-storage/SKILL.md` |
| What to cache, TTL, eviction, herd / hot keys | `skills/caching/SKILL.md` |
| L4/L7, algorithms, health checks | `skills/load-balancing/SKILL.md` |
| Queue vs stream, guarantees, ordering, DLQ | `skills/messaging-streaming/SKILL.md` |
| CAP, models, quorum, saga / transactions | `skills/consistency-coordination/SKILL.md` |
| Breakers, retries, SPOF, degrade, recovery | `skills/resilience-failure/SKILL.md` |
| Edge, CDN, geo-routing | `skills/content-delivery/SKILL.md` |
| DNS, routing policies, TTL | `skills/dns/SKILL.md` |
| IDs at scale | `skills/sequencer/SKILL.md` |
| Blobs / media | `skills/blob-store/SKILL.md` |
| Search / typeahead | `skills/distributed-search/SKILL.md` |
| SLOs, metrics, alerting | `skills/observability/SKILL.md` |
| Log pipeline at volume | `skills/distributed-logging/SKILL.md` |
| Jobs / scheduling | `skills/task-scheduling/SKILL.md` |
| Hot counters | `skills/sharded-counters/SKILL.md` |
| 10x / 100x evolution, bottleneck | `skills/scaling-evolution/SKILL.md` |

Summarize each block in 1–3 sentences and name it in backticks — never duplicate its tables or numbers.

## Provider rule (blocking)

Default to the generic recipe in `skills/<block>/references/providers/generic.md`. If and only if the user names a cloud, also read `skills/<block>/references/providers/<provider>.md` (aws / azure / gcp, plus temporal where durable execution matters) for the managed-service mapping and limits, marked "verify against current docs". If no provider file exists, the generic recipe is the answer — never invent a service or quota.

## When NOT to

Do not use for a single-part question ("what cache eviction?", "SQL or NoSQL here?") — read that one building-block `SKILL.md` directly. Do not emit HTML/SVG — output is single MD only; diagrams are inline mermaid per below.

## Clarify first

Answer before writing a section. Unanswered load-bearing inputs become written, revisable `Assumption:` lines — never silent facts.

- **FR:** 2–4 verb-first core flows. What is the one journey the design lives or dies by?
- **NFR:** scale horizon (DAU, growth), read:write ratio, p99 per flow, nines per flow, consistency split (strong vs eventual per dataset), durability / RPO.
- **Slice:** `--full` (default, 12 sections, 5 diagrams) / `--hld-only` / `--lld-only`.
- **Out-of-scope:** 3–5 explicit exclusions.
- **Scale inputs for BOTEC:** DAU, actions/user/day, bytes, retention, peak factor. Computed QPS/storage are outputs with units.

Terms are enforced per `references/nomenclature-map.md` (FR / NFR / HLD / LLD / BOTEC / SPOF / SLO/SLI).

## Method (staged journey — never skip, never merge diagrams)

Follow `references/staged-journey.md`. Seven stages, each producing document content:

1. **Step 0 — Assumptions table.** Question | Assumption | Why-it-drives-design. Every later section traces here.
2. **Step 1 — Naive HLD v1 + mermaid #1 (≤3 nodes).** Cheapest working system. State archetype, why simple is correct, what is deferred (YAGNI) + forcing condition.
3. **Step 2 — BOTEC break-table.** Mandatory chains with units: `QPS_avg = DAU × actions/day ÷ 86,400`, `Peak = avg × factor` (state it), `storage/day = writes/day × bytes`, `total = storage/day × retention`, `bandwidth = QPS × payload`, `servers ≈ peak ÷ per-server_QPS` (RDBMS ~1k, KV ~10k, cache ~100k–1M). Table: Load | Math | Verdict (breaks what / forces what). Only chains that can change a decision; justify skips.
4. **Step 3 — API contracts (LLD surface).** Concrete endpoints + shapes, cursor pagination, idempotency keys on non-idempotent writes, `429 + Retry-After` / `409`, `/v1`. No request/response/key = guesswork.
5. **Step 4 — Evolved reads + mermaid #2 (≤10 nodes).** Add CDN/cache/search/blob only where Step 2 forces them. Per component: Why / Without-it / Change-when.
6. **Step 5 — Evolved writes + mermaid #3 (≤10 nodes).** Split sync money path (TX, idempotency, saga + compensation) from async path (buffered stream, at-least-once + idempotent consumers, per-key ordering, partition key, retention, DLQ).
7. **Step 6 — LLD deep-dive + mermaid #4 (≤10 nodes, one subsystem only), then final topology + mermaid #5 (≤14 nodes).** Hot-path walk, key/index/TTL table, failure injection per box, then data model, trade-offs, failures, 10×/100×, open questions, validation gate.

Full template: `assets/single-design-template.md`. Copy it per design; keep all 12 sections.

## Mermaid rules (mermaid-only — no HTML/SVG)

Per `references/mermaid-conventions.md`. Non-negotiable subset:

- Exactly 5 ` ```mermaid flowchart ` blocks in `--full` (v1, reads, writes, deep-dive, final). HLD-only: 4. LLD-only: 2.
- `flowchart TB` or `LR` only. Solid ` -->` = sync, numbered `1 VERB path`. Dashed `-.->` = async/cache/replay only.
- ≤10 nodes staged (final ≤14). Short labels (`Order Svc`, `PG city-shard`). One subgraph per layer max (`Edge`, `Services`, `Async`, `State`); no nesting, no click, no HTML labels, ≤5 `style` lines on final only.
- Over cap = split the stage, don't ship it.

## Anti-hallucination rules (blocking)

- No invented requirements — every FR/NFR/number traces to user input or `Assumption:`.
- No invented services/quotas — generic names by default (Postgres, Redis, Kafka, CDN, blob store).
- No fake guarantees — at-least-once + idempotent consumers; ordering scoped to a partition key, never global.
- No decoupled numbers — every figure shows inputs → math → units.
- No rationale-free boxes — any component without Why/Without/Change is deleted.

## Coverage sweep (before emit)

Media/IDs/search/logs/SLOs/jobs/counters each addressed or explicitly deferred with one line — else the doc fails validation.

## Dos and don'ts

Do: expand assumption → number → box, in that order; tie every box to FR/NFR/verdict; state breaking points; end with weakest dimension + current bottleneck + next-magnitude change.
Don't: open with final architecture; name-drop without Solves/Worsens/Change; put rationale on arrows; emit HTML; compute numbers that change no decision.

## References

- **`assets/single-design-template.md`** — the 12-section template with 5 mermaid slots + blocking validation gate. Copy per design.
- **`references/staged-journey.md`** — 7-stage method + archetype adaptation. Read before writing.
- **`references/mermaid-conventions.md`** — diagram syntax, caps, arrow semantics. Read before drawing.
- **`references/nomenclature-map.md`** — term definitions + skill ownership map. Read when tempted to redefine a term.
