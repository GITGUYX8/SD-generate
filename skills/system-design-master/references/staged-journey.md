# Staged journey (the 7-stage method in depth)

Work stages in order. Each stage has entry inputs, actions, and a document output. Skipping a stage is how hallucinated designs happen.

## Stage 0 — Assumptions table

- Inputs: vague prompt + clarifying answers (or labeled assumptions).
- Actions: restate the prompt in one sentence. Fill Question | Assumption | Why-it-drives-design for: core flows (FR), scale horizon, consistency split, latency/SLOs, peak factor, out-of-scope.
- Output: markdown table + one-line scope ("design X for Y at Z scale, strong A, eventual B").
- Anti-hallucination: any cell without a user answer gets `Assumption:` prefix. No silent facts.

## Stage 1 — Naive HLD v1

- Actions: draw ≤3 boxes (Clients → API → single store). List what is deliberately deferred with the forcing condition ("no cache until reads exceed ~10k QPS").
- Output: mermaid #1 + 3-bullet "why starting simple is correct".
- Worked mini-pass (Uber Eats): monolith + single Postgres is correct on day one — order+payment commit atomically, `UNIQUE(idempotency_key)` is free. Deferred: cache, queue, search.

## Stage 2 — BOTEC break-table

- Mandatory chains (show inputs → math → result with units):
  - `QPS_avg = DAU × actions_per_user_per_day ÷ 86,400` (use 1e5 for mental math, then state exact).
  - `QPS_peak = QPS_avg × peak_factor` (state factor; 2–3× default, dinner/spike higher).
  - `storage/day = writes_per_day × avg_bytes`; `total = storage/day × retention_days`.
  - `bandwidth = QPS × payload_bytes` (reads and writes separately; egress dominates cost).
  - `servers ≈ peak_QPS ÷ per-server_QPS` (RDBMS ~1k, KV ~10k, cache ~100k–1M per `back-of-the-envelope`).
- Output: per-load table Load | Math (with units) | Verdict (breaks what / forces what).
- Worked mini-pass: order writes 12M/day → 140 avg / 1.5k peak (fine); tracking 100k QPS (never DB); locations 250k/s (never RDBMS); browse 18k (cache); photos 12.5TB (leave app tier).
- Anti-hallucination: every QPS/storage/bandwidth row shows inputs. A number without inputs + units is deleted.

## Stage 3 — API contracts

- Actions: per core flow write method + path + key request/response fields; cursor pagination; idempotency keys on non-idempotent writes; error contract; versioning. Tie each endpoint to the FR it serves.
- Output: endpoint list + one-sentence per-endpoint rationale (retry safety, cacheability).
- Route to `api-design` for shape details; do not invent new contract jargon.

## Stage 4 — Evolved reads

- Trigger: Step-2 verdict names a read load exceeding its tier.
- Actions: add exactly the components the verdict forces (CDN, cache, search index, blob). Per component write Why / Without-it / Change-when (one line each, tied to the verdict number).
- Output: mermaid #2 (≤10 nodes) + rationale cards.
- Route to `caching` (strategy/TTL/eviction/herd), `content-delivery` (push vs pull, cache-control), `distributed-search` (index/rank/typeahead), `blob-store` (durability/chunking/signed URLs).

## Stage 5 — Evolved writes

- Trigger: write spike, durability, or cross-service coupling verdict.
- Actions: split sync money path (TX + idempotency + saga happy/compensation) from async movement path (buffered stream, partition key, retention, per-key ordering scope, at-least-once + idempotent consumers). State partition keys and retention with reasons (locality, replay window).
- Output: mermaid #3 (≤10 nodes) + rationale cards.
- Route to `data-storage` (shard key, indexes, replication), `messaging-streaming` (queue vs stream, guarantees, DLQ, backpressure), `consistency-coordination` (CAP position, saga vs 2PC), `sequencer` (ID choice).

## Stage 6 — LLD deep-dive + final topology

- Deep-dive: pick the single most fragile subsystem (highest write fan-in, tightest SLO, or money path). Output mermaid #4 (≤10 nodes) plus: hot-path walk with per-hop protocol, key/index/TTL table, failure injection (kill each box, state user-visible degrade), and the metric that signals the next bottleneck.
- Final: assemble mermaid #5 (≤14 nodes, topology only — no rationale on arrows). Follow with data model, trade-offs table (Solves / Worsens / Change-it-when; Worsens non-empty on every row), SPOF + degradation + recovery table, 10×/100× evolution with signals, open questions.
- Coverage sweep before emit: media/IDs/search/logs/SLOs/jobs/counters each addressed or explicitly deferred with one line. Route to `resilience-failure`, `scaling-evolution`, `observability`, `distributed-logging`, `task-scheduling`, `sharded-counters`, `service-decomposition`, `dns`, `load-balancing` as hit.

## Adapting to any problem archetype (agnostic rule)

Same 7 stages for every problem. What changes per archetype is the v1 shape, the BOTEC chains that matter, and the deep-dive target — never the method. Classify first, then adapt:

| Archetype | Examples | v1 shape | BOTEC chains that decide | Deep-dive target | Blocks likely forced |
|---|---|---|---|---|---|
| Single-node stateful (data structure / LLD) | LRU cache, rate limiter (single-node), ID generator | Client → struct → memory (no network) | ops/sec, entry bytes × capacity, eviction cost, concurrency (locks vs sharding) | Core struct + concurrency: hashmap + doubly-linked list, lock granularity, expiry wheel, persistence choice | `caching` (eviction), `sequencer` (IDs), `sharded-counters` (counts) |
| Read-heavy serve | News feed, browse/search, video catalog | Clients → API → single DB | read:write ratio, hot working-set bytes vs RAM, CDN offload % | Read path: cache strategy, denormalized read model, fan-out mode, herd guards | `caching`, `content-delivery`, `data-storage`, `distributed-search` |
| Write-heavy ingest / streaming | Metrics/logs pipeline, live streaming platform, chat ingest | Producer → API → single store/queue | peak write QPS, payload bytes, retention, consumer lag budget | Ingest path: buffer choice, partitioning, backpressure, DLQ, replay | `messaging-streaming`, `data-storage`, `distributed-logging`, `blob-store` |
| Live fan-out (low-latency push) | Stocks ticker, sports scores, presence | Publisher → API → polling clients | concurrent subscribers, update rate × fan-out factor, freshness SLO | Fan-out: push vs pull tier (poll → SSE → WS → MQTT/edge), delta encoding, sticky/regional fan-out | `messaging-streaming`, `caching`, `load-balancing`, `observability` |
| Transactional core | Orders/payments, reservations, booking | Client → API → single RDBMS | write QPS vs 1k/node ceiling, contention on hot rows, idempotency coverage | TX boundary: shard key, saga happy + compensation, idempotency, isolation level | `consistency-coordination`, `data-storage`, `api-design`, `resilience-failure` |

Adaptation rules (blocking):

- **v1 always matches the archetype's cheapest correct system.** LRU v1 is an in-process struct, not Clients → API → Postgres. Streaming v1 is one origin + one store, not a microservice mesh. State the archetype in Step 1 ("Archetype: write-heavy ingest → v1 = …").
- **Only run BOTEC chains that can change a decision.** LRU: capacity bytes + eviction rate (skip bandwidth/regions). Stocks: subscribers × ticks/sec × bytes (skip long retention). Streaming: ingest QPS × segment bytes × retention + egress (skip per-row TX analysis). Say which chains are skipped and why (YAGNI).
- **Deep-dive where the archetype hurts.** LRU → eviction + concurrency + TTL correctness. Streaming → chunking/transcode, CDN edge, DVR window. Stocks → fan-out fan-in, ordering per symbol, stale-vs-down degrade. Transactional → shard key + saga. One subsystem only.
- **Coverage sweep stays universal** but defers loudly per archetype: LRU defers CDN/ES/Kafka with one line each; streaming defers strong TX with one line; stocks defers blob search with one line. A silent skip fails validation.
- Worked one-liners (same method, different numbers): LRU 100k keys × 1KB = 100MB fits RAM → no shard; streaming 5k ingest × 2MB segments × 7d retention → object store + CDN, never RDBMS on media path; stocks 1M subs × 10 ticks/s × 200B = 2GB/s fan-out → push tier + delta cache, never per-tick DB reads.
