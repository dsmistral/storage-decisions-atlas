# PROJECT.md — Storage Decisions Atlas project record

## Current state (updated each session)

**Phase: 1 → 2 (Phase 1 scope gate PASSED 2026-10-20/21; R5 freshness pass DONE; R6
verification pass DONE; chapter authoring may begin).** The survey corpus is complete
through six research rounds (R1–R6): 66 cases (C01–C66), 30 documented transitions
(T01–T30), and 12 failure post-mortems (F01–F12), all register-first with exact source
URLs. The readable master survey lives at `corpus/SURVEY.md`; R5/R6 findings live in the
register and analysis (fold into the survey at authoring time if needed).

**Scope approval (user gate, session 1):** the 4-part / 23-chapter structure
(10 family chapters with 2–4 compared cases each, 6 transition chapters, 4
failure chapters, 3 synthesis chapters, plus appendices A/B/C) is APPROVED as
planned, with **primary audience: working engineers / architects** — favor
technical depth (access patterns, operational detail, failure mechanics) over
executive framing; decision guidance addresses the engineer who runs the system.

**Next unit:** author Chapter 1 (Relational: three ways to stay relational —
C01, C02, C03, +C47, C37), one working session, ALWAYS-SYNC. R6 enriched the
Ch. 1 bench: C61 OpenAI and C56 Zerodha are first-party counter-weights to
"you must shard" (reads scale by replicas and mastery, writes are the honest
limit — W5); C55 Notion and C54 Figma are the modern sharding-in-place faces
for Ch. 13.

---

## Master Pattern Index (P-index)

Each pattern has: the decision rule, the faces (documented cases), and the
fails-when condition. Case/transition/failure IDs refer to `corpus/REGISTER.md`.

| ID | Pattern | Faces | Fails when |
|---|---|---|---|
| P1 | **Availability-first, app-side reconciliation**: for partition-tolerant writes, push conflict resolution to the application, which knows the semantics | C10 (Dynamo) | Copied by teams without the app-side discipline — the trade needs semantics the app must own |
| P2 | **Workload-match the migration**: split by access pattern, not one flagship product | C46 (Amazon Oracle exit: ledger→DynamoDB, transactional→Aurora, analytics→Redshift) | Semantic translation (savepoints, PL/SQL) is assumed free — the savepoint-timeout friction |
| P3 | **Managed primitive ≠ production database**: transitional stops can be unreliable at volume | C15 (Netflix's SimpleDB stop) | Used as a destination rather than a bridge |
| P4 | **Natural tenant key sharding**: keep SQL, split along a real-world boundary (user, channel, shop, domain) | C01, C04, C05, plus C26 | No natural tenant exists, or the key is chosen late |
| P5 | **The seductive wrong key**: "the tenant is the shard" is the most seductive wrong choice in SaaS | F10 (Slack workspace-per-shard, its "original sin") | Tenant granularity is coarser than the hot unit — big tenants become immovable hotspots |
| P6 | **Vertical before horizontal**: split table groups (domains) before splitting rows | C01 (GitHub domain splits) | Joins across domains are still needed by the app |
| P7 | **Movable tenants beat rehashing**: make the tenant relocatable instead of statically hashed | C04 (Shopify shop moves) | The runtime isn't podded — per-tenant state is scattered |
| P8 | **Team size is an architectural input**: the engine your small team can operate wins | C05 (Instagram, 3 engineers), C02, C03 | Used to excuse avoidable learning: skill justified a move whose published mechanics were overstated (F11, Uber) |
| P9 | **The adoption learning loop**: adopt → break → build simple → evaluate honestly → buy the converged engine | C26 (Pinterest: NoSQL broke → ShardDB → TiDB) | The honest re-evaluation step is skipped — one bad era becomes permanent infrastructure |
| P10 | **Consistency bought back expensively**: every team that gave up strong consistency for availability bought it back later | C36 (Twitter Manhattan's 2016 strong-consistency retrofit) | Uniqueness/billing semantics are quietly load-bearing from day one |
| P11 | **Latency as the price of correctness**: accept higher per-op latency rather than give up transactions | C27 (F1/Spanner), C28 (OceanBase) | The workload can't hide the latency (no async app patterns) |
| P12 | **Engine swap, same protocol**: replace storage internals, keep schema/protocol/ops — the lowest-blast-radius move | C41 (InnoDB→MyRocks), T07 (Discord→ScyllaDB), C16 (Rocksandra) | The pain is in the data model or query layer, not the storage engine |
| P13 | **The fixed-shard-count debt**: a fixed split budget is deferred architecture with a known due date | C40 (Dropbox Edgestore: "only 2 more splits") | Splits are spent without planning the escape |
| P14 | **Engine-agnostic abstraction as escape hatch**: decouple the API from the engine so the engine becomes replaceable | C40 (Dropbox Panda), C39 (Magic Pocket bidirectional S3↔Pocket) | The abstraction leaks the engine's semantics into the app |
| P15 | **Build only what converges**: the builds that paid off became platforms or open source; the internal-only builds delivered less than the 2026 product | C41/C42 (MyRocks), Vitess (C01-adjacent), C23/C24 (M3, Atlas open-sourced) vs. C09 (Espresso), C36 (Manhattan), C37 (Schemaless) | No open-source or platform plan — the team bears full maintenance forever |
| P16 | **Cost at scale drives the largest migrations**: licenses, servers, and storage margins, not fashion | C39 (Magic Pocket), C41 (MyRocks halving), C46/C28 (Oracle exits) | Cost is vendor-reported and unaudited — verify before generalizing |
| P17 | **Bold system, boring components**: inside a bold architecture, choose the boring engine | C39 (Magic Pocket's sharded-MySQL Block Index) | The boring component becomes the bottleneck — then a targeted swap (P12), not a rebuild |
| P18 | **Distribute the app, keep the DB single-writer**: shard the application and give each shard an embedded, serialized database | C43 (Cloudflare Durable Objects) | Cross-shard analytics or global queries are needed — per-object ceilings |
| P19 | **The migration playbook**: dual writes → backfill → shadow reads with diffing → incremental cutover per tenant/shard → rollback kept warm | C01 (Slack read-diffing), C41 (MyRocks staged), T07 (Discord checkpointed migrator), C04 (per-shop moves) | Any step skipped — especially the diff/verification tooling, "where migrations die" |
| P20 | **Reversibility is the survival requirement**: if your plan cannot return to the old system on day 60, it is not a plan | C39 (bidirectional replication), T13/F09 (NexusTrade's two-week round trip survived only because the old path was intact), C41 (InnoDB rollback replicas) | The old path is dismantled at cutover — the first post-migration regression becomes unrecoverable |

**Failure-mode classes (cross-cutting, used in Part III):**

| ID | Class | Faces |
|---|---|---|
| FC1 | Backups are the failure class: unverified, co-located, or in-blast-radius copies are not backups | F02, F04, F05, F06 |
| FC2 | Distribution creates new failure modes (unmonitored shard growth, split-brain failover) | F01, F03 |
| FC3 | Migrations regress in the unprofiled layer | F08 |
| FC4 | Runbooks and automation are code and need guardrails | F07 |
| FC5 | "Purpose-built" is marketing until measured on your queries | F09 |

---

## Chapter map

**Part I — The families: one engine each, several cases each (Ch. 1–10)**
- Ch. 1 Relational: three ways to stay relational — C01, C02, C03 (+ C61 OpenAI, C56 Zerodha counter-weights, R6)
- Ch. 2 Document: three reasons, one aggregate — C06, C07, C08 (+ C44/C45 returns preview)
- Ch. 3 Key-value: one pattern, three scales — C11, C12, C13
- Ch. 4 Wide-column: three fleets and the JVM cure — C14, C15, C16
- Ch. 5 Columnar: fan-in and fan-out — C17, C18, C19
- Ch. 6 Graph: three jobs — C20, C21, C22
- Ch. 7 Time-series: the family both giants built — C23, C24 (+ F09 counter-case)
- Ch. 8 NewSQL: three triggers, one destination — C25, C26, C27, C28
- Ch. 9 In-memory: memcached plus your ops wrapper — C29, C30, C31 (+ C57/C60 Aerospike at Indian scale, on-prem, R6)
- Ch. 10 Vector: where the index lives — C32, C33, C34, C35

**Part II — The transitions (Ch. 11–16)**
- Ch. 11 The transition matrix: how to read it — T01–T30 survey
- Ch. 12 Relational exits — T01, T02 (+ C15, C36 history)
- Ch. 13 Scale-out relational done right — C01, C04, C05, C26-precursor (P4–P7) (+ C54 Figma, C55 Notion, R6)
- Ch. 14 The NewSQL absorptions — T03, T04, C27, C28 (+ T25 Zomato reverse flow, R6)
- Ch. 15 The engine-swap twins — T07, T08, T16 (P12)
- Ch. 16 Returns and round trips — T10, T13, T19, T20 (+ T11 Uber; + T23–T27 R6 flows: Yelp standardization, Deliveroo/Zomato/Freecharge outflows, Bhoos return)

**Part III — The failures (Ch. 17–20)**
- Ch. 17 Backup failures: the class that kills — F02, F04, F05, F06 (FC1)
- Ch. 18 Distribution's own failure modes — F01, F03 (FC2)
- Ch. 19 Process and verification failures — F07, F08 (FC3, FC4)
- Ch. 20 Wrong choices, wrong transitions — F09, F10, F11, F12

**Part IV — Synthesis (Ch. 21–23)**
- Ch. 21 The five real drivers (cross-case analysis)
- Ch. 22 The migration playbook (P19 + P20)
- Ch. 23 The decision framework (seven rules)

**Appendices**
- A. The case register (from `corpus/REGISTER.md`)
- B. Methodology and the evidence ladder
- C. Open questions, deferrals, and watch-lists

Status: no chapter authored yet. All 23 chapters planned (specs in `manuscript/BOOK-PLAN.md`).

---

## Session log

| # | Date (IST) | Unit | Status | Notes |
|---|---|---|---|---|
| 1 | 2026-10-20/21 | Bootstrap + Phase 1 gate | DONE | Survey R1–R4 complete; register C/T/F built; P-index P1–P20; 23-chapter map; Knowledge topics created; root staging pruned once → masters kept in Knowledge topic, then pushed to repo; scope APPROVED (4 parts / 23 chapters), audience = working engineers/architects |
| 2 | 2026-10-21 | R5 freshness pass (user-directed) | DONE | 6 new cases (C48–C53), 2 new transitions (T21–T22); W2 refined, W3 substantially resolved, W4 added; honest negatives D6/D7; Ch. 1 inputs verified current; records pushed (ALWAYS-SYNC) |
| 3 | 2026-10-21 | R6 verification pass (user-directed: compare a third-party report's case list against the corpus; register the misses) | DONE | 13 new cases (C54–C66), 8 new transitions (T23–T30), T19 extended (Voucherify); D8 (Razorpay/Swiggy negatives), W5 (OpenAI write-scaling); Zerodha's three CH stories verified first-party via z-connect; records pushed (ALWAYS-SYNC) |

## Standing to-do

1. ~~Push bootstrap to GitHub repo~~ (done, session 1).
2. ~~Phase 1 scope gate~~ (PASSED, session 1).
3. Author Ch. 1 (first unit session).

## Quality checklist (per chapter)

- [ ] Multiple cases compared, not one flagship
- [ ] Every factual claim cites a register URL
- [ ] Hindsight judgements labelled "Verdict (assessment)"
- [ ] Evidence levels stated (FP/VP/PR/SR)
- [ ] The pattern IDs named for each decision discussed
- [ ] Cross-case comparison paragraph per family/move
- [ ] Session log + chapter map updated and pushed (ALWAYS-SYNC)

---

## Working protocol

- One unit (chapter or split part) per working session; split-on-trouble when a
  chapter exceeds system limits — each part closes its own records and pushes.
- **ALWAYS-SYNC**: records and pushes are one unit; the repo is never stale.
- Read `PROJECT.md` and `corpus/STATE.md` first on every resume.
- Conflicts resolve in favor of the repo.
- The survey track feeds the book and never edits it (see `CONVENTIONS.md`).
