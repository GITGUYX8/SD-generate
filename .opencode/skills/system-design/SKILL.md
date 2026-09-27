---
name: system-design
description: This skill should be used when the user asks to "design a system", "design <a product>" (e.g. "design WhatsApp", "design a URL shortener"), "write the design doc", "full system design in markdown", "HLD and LLD", or "HLD/LLD with mermaid", or wants FR/NFR, BOTEC estimates, trade-offs, and failure modes as one single MD file. Use it whenever a design problem statement must be solved as one complete markdown document, even if they don't say "system-design".
license: MIT
---

# System Design (local router)

Solve a design problem as **one single markdown file** by chaining the repo's guides into the master method. This skill holds no method itself — it triggers the read chain below and then follows it.

## Mandatory read chain (in order, blocking)

Read these before writing any design content. Do not paraphrase from memory.

1. `docs/GUIDE.md` — the ten failure modes + success loop (clarify → estimate → HLD → trade-offs → failure → iterate). Every section you emit defends against one failure mode.
2. `meta/SKILL-CONTRACT.md` — component vs method archetypes, DRY/ownership map (link concepts, never re-teach), frontmatter and cross-link rules.
3. `skills/system-design-master/SKILL.md` — the method and output contract. Obey its source-of-truth rule, provider rule, mermaid caps, and anti-hallucination rules.
4. `skills/system-design-master/references/staged-journey.md` — the 7-stage journey (Steps 0–6). Work stages in order; never skip or merge diagrams.
5. `skills/system-design-master/references/nomenclature-map.md` — term definitions (FR / NFR / HLD / LLD / BOTEC / SPOF / SLO/SLI) + skill ownership map. Use terms exactly.
6. `skills/system-design-master/references/mermaid-conventions.md` — diagram syntax and caps (staged ≤10 nodes, final ≤14; `flowchart TB/LR` only; solid numbered sync, dashed async).
7. `skills/system-design-master/assets/single-design-template.md` — copy per design; keep all 12 sections and the blocking validation gate.

## Building blocks (source of truth, read on demand)

When a concern becomes active, read the owning `skills/<block>/SKILL.md` and apply its options / trade-offs / stress behavior in 1–3 sentences, naming it in backticks. Never duplicate its tables or numbers. Owner map: `requirements-scoping` (scope), `back-of-the-envelope` (QPS/storage/bandwidth/servers + `scripts/botec.py`), `api-design` (contracts), `service-decomposition` (boundaries/gateway), `data-storage` (keys/sharding/replication), `caching` (TTL/eviction/herd), `load-balancing` (L4/L7/health), `messaging-streaming` (queue vs stream/DLQ), `consistency-coordination` (CAP/saga), `resilience-failure` (breakers/degrade), `content-delivery` (edge/CDN), `dns` (routing/TTL), `sequencer` (IDs), `blob-store` (media), `distributed-search` (index/rank), `observability` (SLOs/alerts), `distributed-logging` (log pipeline), `task-scheduling` (jobs), `sharded-counters` (hot counters), `scaling-evolution` (10x/100x).

## Provider rule (blocking)

Default to `skills/<block>/references/providers/generic.md`. If and only if the user names a cloud, also read `skills/<block>/references/providers/<provider>.md` (aws / azure / gcp, plus temporal where durable execution matters), marked "verify against current docs". If no provider file exists, the generic recipe is the answer — never invent a service or quota.

## When NOT to

Do not run the full chain for a single-part question ("what cache eviction?", "SQL or NoSQL here?") — read that one building-block `SKILL.md` directly. Do not emit HTML/SVG — output is single MD with inline mermaid only.

## Validation gate

Close via the template §12 checklist: every trade-off row has non-empty Worsens + breaking point, every number shows inputs → math → units, every component traces to an FR/NFR/verdict, every critical dependency has a degrade path, mermaid caps hold, and the coverage sweep (media / IDs / search / logs / SLOs / jobs / counters) is addressed or explicitly deferred with one line.

## Tone and glossary rule (applies at emit)

Carried from user prompts; obey whenever emitting design content.

- On first use of any shortform, keyword, or buzzword, add a plain bracket explanation beside it: `QPS (queries per second: requests arriving each second)`. Keep skill names as bare backticked links per contract; the gloss follows the link.
- Explain from first principles in a friendly voice — an expert friend over coffee: direct address, short sentences, why-before-what, all keywords kept, no unnecessary analogies.
- Mathematical details stay untouched: every number, formula, and table value stays byte-identical to the computed design. Tone changes prose only — never diagrams, numbers, table values, or checklist wording.
- Mermaid safety still wins: never put parentheses inside `|edge labels|` (it breaks parsing); glosses live in prose, never in edge labels.
