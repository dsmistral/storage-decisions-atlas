# corpus/ANALYSIS.md — Per-pattern firing evidence

Which corpus items instantiate each P-index pattern (see `PROJECT.md`), with the
evidence that makes each firing real. Use when designing chapters: a pattern is
taught from its faces, and its fails-when condition is taught from the failures.

| Pattern | Faces (firing evidence) | Fails-when evidence |
|---|---|---|
| P1 availability-first | C10: cart "must accept writes during a partition"; conflict policy pushed to app (SOSP FP) | Twitter's retrofit (C36): uniqueness semantics the app couldn't own → P10 |
| P2 workload-match | C46: ledger→DynamoDB, transactional→Aurora, analytics→Redshift; Buyer Fraud 40 TB/1h | C46 itself: savepoint-mismatch timeouts at 92% done — semantic translation is the residual cost |
| P3 managed-primitive | C15: SimpleDB "proved unreliable at their volumes," itself replaced | — |
| P4 natural tenant key | C01 (domain), C04 (shop_id), C05 (user id in ID), C26 (app-managed ID→shard) | F10/P5: the tenant exists but chosen coarse |
| P5 seductive wrong key | F10: "original sin of early Slack infrastructure" | F10 + Slack's own 3-year correction (P19 discipline applied) |
| P6 vertical-first | C01: domain table moves before row splitting; linters prevent cross-domain joins | — |
| P7 movable tenants | C04: "entirely online, virtually zero downtime" whole-shop moves | — |
| P8 team-size input | C05 (3 engineers), C02, C03 (one DB, small team) | F11: skill justified the move; blog blamed the engine — the conflation is the failure |
| P9 learning loop | C26: NoSQL "broke catastrophically" → ShardDB → 2022 evaluation of 10+ → TiDB | C09/C36: internal-only builds that never re-ran the evaluation |
| P10 consistency bought back | C36: 2016 strong consistency retrofit — "one email address may get attached to two accounts" | — |
| P11 latency-for-correctness | C27: F1 accepted higher per-op latency, hidden by async app patterns; C28: ~10% per-query gap accepted | — |
| P12 engine swap | C41 (−62.3% instance size, rollback replicas), T07 (177→72 nodes), C16 (Rocksandra), C42 (HBase→MyRocks) | — the matrix's lowest-blast-radius move; repeated independent invention is the signal |
| P13 fixed-shard debt | C40: "only 2 more splits … no more splits available" | — |
| P14 abstraction escape hatch | C40: Panda "independent of the underlying storage engine"; C39: bidirectional S3↔Pocket | — |
| P15 build-what-converges | Vitess (C47, CNCF), MyRocks (C41, Percona), M3/Atlas (C23/C24, open-sourced) vs. Espresso (C09), Manhattan (C36: "may even replace our internal offerings" — Twitter's own words) | — |
| P16 cost-at-scale | C39 (S3 margins), C41 (halve servers), C46/C28 (Oracle exits), C11 (−70% cost) | All figures vendor/self-reported — unaudited (open question) |
| P17 bold-system-boring-parts | C39: Block Index "a giant sharded MySQL … a really great decision in terms of simplifying development" | — |
| P18 distribute-the-app | C43: per-object SQLite, single writer ⇒ strong consistency without consensus | C43's own ceiling: per-object storage limits — per-tenant store, not global analytics |
| P19 migration playbook | C01/Slack (double-writes + read-diffing, 3 years), C41 (staged + consistency checks), T07 (checkpointed Rust migrator, 9 days), C04 (per-shop), C39 (bidirectional) | The playbook steps skipped = the failure classes below |
| P20 reversibility | C39, C41, T13/F09 (NexusTrade survived because old path intact) | F06: no out-of-domain backup = nothing to return to |

Failure classes (Part III spine):
- FC1 backup-failure: F02 (silently failing pg_dump, version mismatch), F04 (no verified backup), F05 (backups on same broken stack), F06 (backups inside compromise domain)
- FC2 distribution's failure modes: F01 (unwatched shard growth; page-size/doc-size ratio), F03 (split-brain during 43s partition)
- FC3 unprofiled layer: F08 (lock contention invisible in hardware metrics)
- FC4 runbooks-as-code: F07 (playbook command over-removed servers; status page on S3)
- FC5 purpose-built marketing: F09 (continuous aggregates fought the workload)

**Cross-case driver synthesis** (feeds Ch. 21): the five real drivers —
availability/blast radius (C10, C15, C04, C01), ops burden (C36, C38, C26, C40),
cost at scale (C41, C39, C46, C28), correctness requirements (C27, C28, C36, F10-adjacent),
team skill (C05, C37, C02, C03).

## R5 additions (session 2)

| Pattern | New firing evidence (R5) |
|---|---|
| P10 (consistency bought back) | C48: Uber found Cassandra's eventual consistency "impeding developer productivity" → Docstore with strict serializability at partition level — the same consistency-buyback as Twitter's Manhattan retrofit, at Uber |
| P14 (abstraction escape hatch) | C51: Netflix builds KV, Time-Series and Graph data abstractions over Cassandra + a Key-Value Data Abstraction Layer across multiple online DBs; C53: 37signals' Active Record Tenanted gem abstracts tenant routing |
| P15 (build what converges) | C48: Docstore is Uber re-running the internal build (Schemaless → general-purpose transactional DB) — watch W4 for whether it converges; C49: Netflix *buys* (operates CRDB as a service) rather than builds — the 2023 answer to the same requirement Manhattan was built for in 2012 |
| P16 + P19 + P20 | C50: 37signals moved 5 PB / ~5B objects off S3 with zero downtime — custom copy-and-reconciliation tooling (budget for verification, not just the move), intentional move order, and the verification anxiety of the irreversible delete |
| P18 (distribute-the-app) fails-when | C53: FIRST-PARTY fails-when evidence — SQLite-per-customer attempted for Fizzy, "globally replicated SQLite" edge cases, switched to MySQL before launch. P18's per-object ceiling, now documented from a team that tried it |
| FC3 (unprofiled layer) ops cousin | C52: Tinybird's 5-year CH ops lessons — memory budgets and external sorts are the settings that bite at petabyte scale (Ch. 5 operations sidebar) |
| Ch. 1 caveat | C02: Nick Craver has moved to Microsoft Azure; his 2013/2016 accounts remain the canonical first-party record (cite with dates, as ever) |

## R6 additions (session 3, user-directed verification pass)

| Pattern | New firing evidence (R6) |
|---|---|
| P4 (natural tenant key) | C55 Notion: workspace ID as shard key — all of a workspace's blocks co-located; the P4 face list gains its Postgres-at-product-scale member |
| P6 (vertical before horizontal) | C54 Figma: vertical partitioning into a dozen DBs was the explicit 2020–22 stepping stone before 9-month horizontal sharding — second first-party face after C01 |
| P13/P7 (fixed-shard debt, planned away) | C55 Notion: 480 logical shards chosen for divisibility (32→40→48 host jumps possible), which made the later re-shard 32→96 a PgBouncer redirect with slow verified cutover instead of a second migration. The rare case of P13 debt designed out in advance |
| P19 (migration playbook) | C54 Figma: shadow-planning / "shadow application readiness" on live traffic = the playbook's diff/predict step, tool-automated. C55 Notion: dark reads with diffing, AND migration vs. verification logic implemented by different people — first-party evidence that verification independence is a real step, not paranoia |
| P8 (team size is an input) | C62 Yelp: the honest mirror of F11 — Yelp moved a product DB to org-standard MySQL because the team lacked deep Postgres expertise and said so plainly; Uber's same-direction move aged badly because it conflated engine mechanics with ops inexperience. F11's control case |
| P2 (workload-match) | C58 Zomato: ledger/payout workload with bursty peaks → pay-per-use KV; TiDB's over-provisioning was the cost of the wrong shape. C63 Deliveroo: after-commit Kafka semantics rebuilt on DynamoDB Streams — semantic translation is again the residual cost (cf. C46 savepoints) |
| P2/P16 (columnar on immutable/log data) | Three independent first-party confirmations of the T05/T06 family: T28 Uber (one CH node ingests ~300K logs/s, ~10× an ES node) + Zerodha (28B lines/13 TB ELK → 30B lines/2.5 TB CH); T29 Zerodha Timescale→CH (~1 TB → ~120 GB); T30 Zerodha back-office PG→CH (20 TB → 4 TB, "1000×" in many scenarios) |
| P3-adjacent (single-node mastery) | C56 Zerodha: Stack Overflow's kin at Indian-fintech scale — ~trillion-row DB served from single-node self-hosted Postgres plus an ephemeral-table result cache (7M tables/day, DungBeetle). Ch. 1 counter-weight alongside C02 |
| Ch. 1 counter-example | C61 OpenAI: 800M users, one primary + ~50 read replicas; reads scale by replication, not sharding; the write bottleneck is the honest limit (W5). The strongest modern "you may not need to shard yet" datapoint; teach alongside C02 and C56 in Ch. 1 |
| P16 caution + Ch. 9 breadth | C57 PhonePe + C60 Flipkart: the in-memory family gains two hyperscale Indian cases, both self-run on-prem (Aerospike) — rare non-cloud counterpoints to the managed-everything narrative. Both sit on Aerospike properties (VP); PhonePe's fraud-model post is the first-party anchor |
| Direction findings (Ch. 11 matrix) | T25 (Zomato TiDB→DynamoDB) and T27 (Bhoos DynamoDB→PG) show neither distributed SQL nor managed KV is a terminal state — both reverse flows now first-party. T26 vs C57: Freecharge chose managed while PhonePe chose self-run in the same workload class — one decision, two answers (Ch. 21 ops-burden driver). T19's "quiet return" gains Voucherify (mid-size, zero downtime, ~30% cheaper) |

