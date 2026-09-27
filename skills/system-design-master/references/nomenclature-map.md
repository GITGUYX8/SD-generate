# Nomenclature map (use these terms exactly)

## Core terms

- **FR (functional requirements):** verb-first user-visible capabilities ("place order", "track courier"). Each FR maps to ≥1 endpoint. Home: `requirements-scoping`.
- **NFR (non-functional requirements):** quantified qualities only — scale (DAU, QPS in/out), p99 latency per flow, availability nines per flow, consistency model per dataset, durability (RPO), retention, peak factor. A numberless NFR is rewritten or deleted. Home: `requirements-scoping` (capture) + `back-of-the-envelope` (compute).
- **HLD (high-level design):** service topology — clients, edge, services, async, state — plus which NFR/number each box satisfies. One concern per diagram. Home: `system-design` loop steps 3–4 + `service-decomposition`.
- **LLD (low-level design):** concrete contracts — endpoints, schemas, primary/shard/sort keys, indexes per hot query, TTLs/retention, partition keys, saga steps + compensations, matcher/scheduling logic. Home: `api-design` + `data-storage`.
- **BOTEC:** the visible multiply/divide chain with units (inputs → math → verdict). Home: `back-of-the-envelope`.
- **SPOF:** any box whose loss halts a core flow; each gets a degrade + recovery line. Home: `resilience-failure`.
- **SLO/SLI:** objectives (e.g. "99.95% browse serves <300ms") and indicators measuring them; each SLO names its signal for the evolution section. Home: `observability`.
- **Out-of-scope:** explicit non-goals (3–5). Deferral is a decision with a forcing condition ("no ES until search exceeds trivial filters").

## Ownership map (link, don't re-teach)

| Term/concept | Owner skill | This skill may state |
|---|---|---|
| FR/NFR capture, clarifying questions | `requirements-scoping` | 3-list output + assumptions |
| QPS/storage/bandwidth/servers, latency table, nines | `back-of-the-envelope` | Step-2 chains + verdicts |
| Endpoints, pagination, idempotency, versioning, errors | `api-design` | contract list |
| Sharding, replication, indexes, SQL/NoSQL | `data-storage` | keys + rationale |
| TTL, eviction, herd/hot-key | `caching` | strategy + numbers |
| Queue vs stream, guarantees, ordering, DLQ | `messaging-streaming` | topology + guarantees |
| CAP, models, quorum, saga | `consistency-coordination` | position + mechanism name |
| Breakers, backoff, bulkheads, degrade, rate limits | `resilience-failure` | per-failure rows |
| Bottlenecks, 10×/100× | `scaling-evolution` | bottleneck + signals + next step |
| Gateway, boundaries, discovery | `service-decomposition` | granularity + drivers |
| LB, DNS, CDN behavior | `load-balancing`, `dns`, `content-delivery` | one-line mechanism |
| IDs, blobs, search, logs, jobs, counters, SLOs | `sequencer`, `blob-store`, `distributed-search`, `distributed-logging`, `task-scheduling`, `sharded-counters`, `observability` | choice + key knob or defer-line |

When a concept is load-bearing but owned elsewhere, write a 1–3 sentence working summary, name the owner skill in backticks, and move on.
