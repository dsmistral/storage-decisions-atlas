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
