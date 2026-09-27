# system-design-skills (slim: master + building blocks)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Skills](https://img.shields.io/badge/Building_Blocks-20_+_master-brightgreen)](#building-blocks)
[![Providers](https://img.shields.io/badge/Providers-AWS%20%C2%B7%20Azure%20%C2%B7%20GCP%20%C2%B7%20Temporal-blue)](#provider-modularity)

**Solve a design problem statement as one single markdown file.** One master skill (`skills/system-design-master/`) reasons outward from your FR/NFR/numbers and reads the 20 building-block `SKILL.md` files as the source of truth for every decision. Output is a single MD doc: 12-section HLD + LLD with 5 small mermaid diagrams. Nothing else.

## How to use

Trigger the master skill conversationally — no slash command, no agent, no config:

> *"Design a URL shortener for 100M DAU, full HLD+LLD in a single md"*
> *"Write the design doc for WhatsApp-scale messaging — single md with mermaid"*

The master skill will:

1. Lock FR / NFR / out-of-scope (or label `Assumption:` and move on).
2. Run the BOTEC break-table (QPS, storage, bandwidth, servers — with units).
3. Read the owning `skills/<block>/SKILL.md` per concern and apply its options / trade-offs / stress behavior (1–3 sentence summary, skill named — never duplicated).
4. Emit the single MD from `skills/system-design-master/assets/single-design-template.md` — Steps 0–7, exactly 5 `mermaid flowchart` blocks (caps 10/10/10/10/14).
5. Run the blocking validation gate, then close with weakest dimension + bottleneck + next-magnitude change.

For a single-part question, skip the master and read that one block directly (e.g. "what caching strategy here?" → `skills/caching/SKILL.md`).

## Building blocks

20 source-of-truth blocks + 1 master:

| Layer | Blocks |
|:------|:-------|
| **L0 Frame** | `requirements-scoping` · `back-of-the-envelope` |
| **L1 Edge** | `dns` · `load-balancing` · `content-delivery` |
| **L2 Services** | `api-design` · `service-decomposition` |
| **L3 State** | `data-storage` · `caching` · `blob-store` · `sequencer` · `sharded-counters` · `distributed-search` |
| **L4 Async** | `messaging-streaming` · `task-scheduling` |
| **L5 Correctness** | `consistency-coordination` |
| **L6 Ops** | `resilience-failure` · `observability` · `distributed-logging` |
| **L7 Growth** | `scaling-evolution` |
| **Master** | `system-design-master` (method + single-MD output contract, mermaid-only) |

Each block: `SKILL.md` (options/method, trade-offs, stress, numbers) + `references/deep-dive.md` + `references/providers/` (generic default; aws/azure/gcp/temporal on demand). `back-of-the-envelope` additionally ships `scripts/botec.py` — verify with `python3 skills/back-of-the-envelope/scripts/test_botec.py`.

## Provider modularity

Generic by default. Name a cloud and the master reads `skills/<block>/references/providers/<provider>.md` for that block only ("verify against current docs"). No provider file → generic is the answer, never an invented service.

## Project structure

```
system-design-skills/
├── skills/
│   ├── system-design-master/   # MASTER — SKILL.md + assets/single-design-template.md + references/{staged-journey,mermaid-conventions,nomenclature-map}.md
│   ├── <20 building blocks>/   # SOURCE OF TRUTH — SKILL.md + references/deep-dive.md + references/providers/
│   └── back-of-the-envelope/   # + scripts/botec.py + expected_outputs/
├── docs/GUIDE.md               # the ten failure modes (method source)
├── meta/SKILL-CONTRACT.md      # authoring contract for blocks
└── README.md
```

## Design principle

> Do not memorize architectures; learn the forces that shape them. Every component ties to an FR, NFR, or number — anything else is deleted.

## License

[MIT](./LICENSE)
