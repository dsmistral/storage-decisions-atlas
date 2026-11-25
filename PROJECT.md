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

**Chapter 1 AUTHORED + REVIEWED (sessions 4–5, 2026-11-25):** manuscript master
`manuscript/ch01-relational.md` (markdown canvas rendered for reading); scope per
BOOK-PLAN spec; inputs C01, C02, C03, C47, C37 + R6 counter-weights C56, C61,
whose SURVEY dossiers (22a/22b) were folded in with 7 new source-notes rows.
Review passed session 5 with 3 findings fixed (receipts in row 5) and a
planning gap logged (to-do 5).
**Chapter 2 AUTHORED + AUTHOR-CHECKED (session 6) and REVIEWED (session 7):**
manuscript master `manuscript/ch02-document.md` ("Document: three reasons, one
aggregate"; C06, C07, C08, C09 + T10/T19 preview; patterns P2/P15). Review
passed session 7 with 4 objections resolved (receipts in row 7).
**Chapter 3 AUTHORED + REVIEWED (sessions 8-9, 2026-11-25):** manuscript master
`manuscript/ch03-key-value.md` ("Key-value: one pattern, three scales"; C10
design origin, C11, C12, C13 at giant/product/small scale; patterns P1/P2/P16).
Review passed session 9 with 4 objections fixed (receipts in row 9).
**Chapter 4 AUTHORED + AUTHOR-CHECKED (session 10, 2026-11-25):** manuscript master
`manuscript/ch04-wide-column.md` ("Wide-column: three fleets and the JVM cure";
C14 Apple planetary fleet, C15 Netflix transitional stop, C16 Instagram
Rocksandra transplant, C38 Discord full-treatment exit, C36 Manhattan companion;
patterns P3/P9/P12; cure-ladder comparison; graded exercises + ◆ diagnostic).
Battery green (5/5 register IDs incl. C14/C15/C16/C36/C38, 10/10 exact URLs in
register, 6 `## N.` sections, markers each =1, zero em-dash, zero placeholders);
live probes verified Discord quotes verbatim, DevPro fleet credits, InfoWorld
Netflix quotes; New Stack fetch truncated at the 32,793-char cap, corroborated
via search snippets. Quote-hygiene fix at authoring: "proved unreliable at their
volumes" was a survey paraphrase, quote marks removed. Watch D1 (Apple
FoundationDB press-only). Phase 3 review passed session 11 with 4 objections
fixed (receipts in row 11). Next unit: Ch. 5 (Columnar: fan-in and fan-out;
C17 Cloudflare, C18 Bloomberg, C19 CERN LHCb on ClickHouse; T05, T06;
P2 + FC3 preview (F08)).
**Chapter 5 AUTHORED + AUTHOR-CHECKED (session 12, 2026-11-25):** manuscript master
`manuscript/ch05-columnar.md` ("Columnar: fan-in and fan-out"; C17 Cloudflare
fan-in as the T05/T06 pair (P2's disciplined face), C18 Bloomberg fan-out (VP,
ZooKeeper stress→Keeper, DR drills), C19 CERN LHCb science breadth-line (PR),
F08 opened as the FC3 teaser, full read Ch. 19; comparison table fan-in vs
fan-out vs science; graded exercises + ◆ diagnostic). Battery green (8/8
register/transition/failure IDs, 6/6 exact URLs register-copied, markers each
=1, zero em-dash, zero placeholders). Live probes: CF HTTP analytics and CF
logs posts verbatim (6M logs/s, "much smaller", "reduced by eight times"),
Wikipedia C19 verbatim, F08 "massive lock contention" verbatim; CF billing
post live-hit; ClickHouse-hosted pages (C17-VP, C18) fetch as ~471-char
shells, corroborated via search snippets (Bloomberg "hundred million securities
all at once" verbatim; Cloudflare quadrillion-event scale via news coverage).
Also this session: session-11's O2 was itself corrected, the Discord blog does
carry "we need to migrate trillions of messages with no downtime" verbatim
(earlier probe context truncation hid it); the Ch. 4 C38 bullet now quotes the
requirement as stated (ch04 commit f648106); register T07 "no downtime" stands
supported (no emendation needed). Next unit: Phase 3 review of Ch. 5, then
Ch. 6 (Graph: three jobs; C20, C21, C22).
**Chapter 5 REVIEWED (session 13, 2026-11-25):** review passed with 5
objections fixed plus a consistency patch (receipts in row 13). Next unit:
Ch. 6 (Graph: three jobs; C20 ICIJ Panama Papers, C21 Zurich Insurance,
C22 eBay ShopBot), then its review.
**Chapter 6 AUTHORED + AUTHOR-CHECKED (session 14, 2026-11-25):** manuscript
master `manuscript/ch06-graph.md` ("Graph: three jobs"; C20 ICIJ exploration,
C21 Zurich acceleration, C22 eBay ShopBot inference; maturity ladder,
family rule + P15 contrast, BOOK-PLAN P2 cell flagged; graded exercises +
◆ diagnostic). Battery green (5/5 register IDs, 4/4 exact URLs in register,
10/10 quote targets, markers each =1, zero em-dash, zero placeholders). Live
probes: Neo4j ICIJ, Zurich PDF, eBay interviews verbatim; Computer Weekly
page dead-fetch, corroborated via search snippets. Next unit: Phase 3 review
of Ch. 6, then Ch. 7 (Time-series: the family both giants built; C23, C24
(+ F09 counter-case, T17), P15, FC5).
**Chapter 6 REVIEWED (session 15, 2026-11-25):** review passed with 5
objections fixed (receipts in row 15). Next unit: Ch. 7 (Time-series: the
family both giants built; C23, C24, F09, T17; P15, FC5), then its review.
**Chapter 7 AUTHORED + AUTHOR-CHECKED (session 16, 2026-11-25):** manuscript
master `manuscript/ch07-time-series.md` ("Time-series: the family both giants
built"; C23 Uber M3, C24 Netflix Atlas, F09 NexusTrade counter-case, T17 two
mountains; patterns P15 + FC5; build-vs-buy two-directions comparison; graded
exercises + ◆ diagnostic). Battery green (6/6 IDs, 6/6 exact URLs
register-copied, 12/12 verbatim targets, markers each =1, zero em-dash, zero
placeholders). Live probes: Uber billion + M3 posts verbatim, Atlas docs
Epic passage verbatim, NexusTrade post verbatim; Netflix TechBlog URLs are
Medium shells, quotes corroborated via indexed excerpts noted in-chapter.
Next unit: Phase 3 review of Ch. 7, then Ch. 8 (NewSQL: three triggers, one
destination; C25, C26, C27, C28; T03, T04; P9, P11, P15), then its review.

**Pause status:** the 2026-10-21 pause was lifted at resume (2026-11-25), session
4 opened per the repo Cold resume protocol (mirror verified vs repo, HEAD 287f6b3).
This repo remains self-sufficient for a cold resume from GitHub alone.

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
- Ch. 2 Document: three reasons, one aggregate — C06, C07, C08, C09 (BOOK-PLAN spec inputs; incl. T10/T19 preview: P2, P15)
- Ch. 3 Key-value: one pattern, three scales — C10, C11, C12, C13 (BOOK-PLAN spec inputs; C10 is the design origin): P1, P2, P16
- Ch. 4 Wide-column: three fleets and the JVM cure — C14, C15, C16, C38 (+ C36 Twitter's Cassandra era; BOOK-PLAN spec inputs): P3, P9, P12
- Ch. 5 Columnar: fan-in and fan-out — C17, C18, C19 (T05, T06 Cloudflare; F08/FC3 preview; BOOK-PLAN spec inputs): P2
- Ch. 6 Graph: three jobs — C20, C21, C22 (BOOK-PLAN spec inputs; family rule + P15 contrast; BOOK-PLAN's P2 cell flagged in chapter §6)
- Ch. 7 Time-series: the family both giants built — C23, C24 (+ F09 counter-case, T17; BOOK-PLAN spec inputs): P15, FC5
- Ch. 8 NewSQL: three triggers, one destination — C25, C26, C27, C28
- Ch. 9 In-memory: memcached plus your ops wrapper — C29, C30, C31 (+ C57/C60 Aerospike at Indian scale, on-prem, R6)
- Ch. 10 Vector: where the index lives — C32, C33, C34, C35

**Part II — The transitions (Ch. 11–16)**
- Ch. 11 The transition matrix: how to read it — T01–T30 survey
- Ch. 12 Relational exits — T01, T02 (+ C15, C36, C46 history; BOOK-PLAN spec inputs)
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

Status: Ch. 7 AUTHORED session 16 (Chs. 1-7 done: 7 authored, 6 reviewed); Chs. 8–23 planned (specs in `manuscript/BOOK-PLAN.md`).

---

## Session log

| # | Date (IST) | Unit | Status | Notes |
|---|---|---|---|---|
| 1 | 2026-10-20/21 | Bootstrap + Phase 1 gate | DONE | Survey R1–R4 complete; register C/T/F built; P-index P1–P20; 23-chapter map; Knowledge topics created; root staging pruned once → masters kept in Knowledge topic, then pushed to repo; scope APPROVED (4 parts / 23 chapters), audience = working engineers/architects |
| 2 | 2026-10-21 | R5 freshness pass (user-directed) | DONE | 6 new cases (C48–C53), 2 new transitions (T21–T22); W2 refined, W3 substantially resolved, W4 added; honest negatives D6/D7; Ch. 1 inputs verified current; records pushed (ALWAYS-SYNC) |
| 3 | 2026-10-21 | R6 verification pass (user-directed: compare a third-party report's case list against the corpus; register the misses) | DONE | 13 new cases (C54–C66), 8 new transitions (T23–T30), T19 extended (Voucherify); D8 (Razorpay/Swiggy negatives), W5 (OpenAI write-scaling); Zerodha's three CH stories verified first-party via z-connect; records pushed (ALWAYS-SYNC) |
| 4 | 2026-11-25 | Ch. 1: Relational — three ways to stay relational | DONE | Cold resume from GitHub (mirror verified vs repo, HEAD 287f6b3); session-open row pushed (commit 4df2bb2, md5-verified); chapter master `manuscript/ch01-relational.md` (~23 KB): 3 topologies + C56/C61 counter-weights + C37 twin, P4/P6/P7/P8, graded exercises + ◆ diagnostic; SURVEY 22a/22b dossiers + 7 source-notes rows folded; all pushes byte-verified. Phase 3 review pending |
| 5 | 2026-11-25 | Phase 3 adversarial review of Ch. 1 | DONE | Two-expert review of `manuscript/ch01-relational.md`, fresh machine battery + live-source probes. Findings: O1 fleet-figures cite corrected to the MySQL 8.0 post (live-verified); O2 first-resharding quote misattributed to the Vitess history page: primary found (Percona interview, quote verified live), REGISTER C47 amended, chapter/recanvas fixed; O3 C53→"Chapter 18" cross-pin wrong: un-pinned, planning gap logged (to-do 5); clean: 17/17 exact URLs in register, 14/14 register IDs, 9/9 pattern IDs, 24/24 figure/quote targets in corpus, exercise/grading census ✓, live probes: Craver 2013, OpenAI, Zerodha/FOSS/DungBeetle, Uber MVCC, GitHub 8.0 all hit. Chapter status: REVIEWED. |
| 6 | 2026-11-25 | Ch. 2: Document — three reasons, one aggregate | DONE | Chapter master `manuscript/ch02-document.md` (~18.6 KB): C06/C07/C08 as three reasons + C09 Espresso build-dwell (P15), fit-test table, T10/T19 exits preview (P2), graded exercises + ◆; battery: 10/10 register IDs, 3/3 pattern IDs, 9/9 exact URLs in register, 11/11 figure/quote targets in corpus, markers ✓; live probes: eBay quote, Stripe 99.999, Espresso claims, Sailthru 40 TB/120 nodes verbatim ✓; Mixmax page fetch thin (register-corroborated); Voucherify ~30% figure is the author cross-post's claim: citation extended (URL from register C65), marked self-reported. Next: Phase 3 review of Ch. 2, then Ch. 3 |
| 7 | 2026-11-25 | Phase 3 adversarial review of Ch. 2 | DONE | Fresh reviewer battery re-run green (all register IDs valid, 3/3 pattern IDs, all URLs in register, no placeholders/em-dashes). Four objections resolved: O3 wording "Three companies" corrected to "Teams" (four organizations in three reasons); O7 absolute "never open-sourced or productized" softened to "stayed internal, no release or productization on record as of the 2012 post"; O11 aggregate-survival inference in T10 marked as reviewer assessment; O12 "Chapter 9 in reverse" pin replaced with the ops-burden driver (Ch. 21). Three edit-tool slips during fixing caught and repaired before push. Chapter status: REVIEWED |
| 8 | 2026-11-25 | Ch. 3: Key-value — one pattern, three scales | DONE | Chapter master `manuscript/ch03-key-value.md` (~15.5 KB): C10 Dynamo SOSP 2007 as design origin, C11 Amazon Financial Ledger (giant scale), C12 Duolingo (product scale), C13 Mixmax (small scale); P1/P2/P16 (+C36 uniqueness preview, C46 held for Ch. 12); graded exercises + ◆ diagnostic. Battery green: 6/6 register IDs, 4/4 pattern IDs defined, 5/5 exact URLs in register, markers each =1, zero em-dash, zero placeholders. Live probes: Dynamo PDF phrases ("inefficiencies and limit scale"), AWS Oracle-exit (120 TB, 40%, 70%, Financial Ledger), Duolingo TTL/60 lessons ✓. Quote fix: "too simplistic for any application" converted to sourced paraphrase; no fetch of the SOSP PDF survives past ~5 pages (extraction cap) and the phrase does not occur in the fetched portion, so no verbatim attribution is kept. Next: Phase 3 review of Ch. 3, then Ch. 4 |
| 9 | 2026-11-25 | Phase 3 adversarial review of Ch. 3 | DONE | Two-expert review + fresh machine battery green (6/6 register IDs, 4/4 pattern IDs defined, 5/5 exact URLs in register, 9/9 corpus quote/figure targets; live probe: the C36 quote "one email address may get attached to two accounts" verified verbatim on the Manhattan strong-consistency post; Duolingo "31 billion" = register's "31B", corroborated; markers each =1, zero em-dash/placeholders). Four objections fixed: O1 reflow artifact (stranded "the" before the cart sentence) reflowed; O2 "ancestor of Cassandra, DynamoDB, and most eventually consistent stores" trimmed to the register-supported "ancestor of Cassandra and DynamoDB"; O3 "P3-adjacent under the project's reading" misread P3 (Mixmax used the managed primitive as a destination at small scale, where P3's volume caution does not bind) reworded; O4 "the intake lives in the transition chapters" pinned to Chapter 12 (BOOK-PLAN C-home). Chapter map Ch. 12 line matched to BOOK-PLAN (C46 added). One edit-tool slip at session open (stray placeholder written into PROJECT.md) caught and reverted byte-identically before push. Chapter status: REVIEWED |
| 10 | 2026-11-25 | Ch. 4: Wide-column — three fleets and the JVM cure | DONE | Chapter master `manuscript/ch04-wide-column.md` (~19.9 KB, byte parity with canvas except 6-line front matter): C14 Apple planetary fleet (160,000+/100 PB/1,000+ clusters, PR; FoundationDB thread = D1 press-only), C15 Netflix Oracle→SimpleDB stop→Cassandra (P3 canonical face), C16 Instagram Rocksandra engine transplant (P12 purest face), C38 Discord Mongo→Cassandra→ScyllaDB full-treatment exit (FP; 177→72 nodes, 3 months→9 days migrator), C36 Manhattan companion (P10, full arc Ch. 12); patterns P3/P9/P12; cure-ladder comparison; graded exercises + ◆ diagnostic. Battery green: 5/5 register IDs, 10/10 exact URLs in register, 6 sections, markers each =1, zero em-dash, zero placeholders. Live probes: Discord blog quotes verbatim ("manually reboot and babysit", 177→72, 9 TB vs 4 TB, "improved drastically"); DevPro Journal carries Apache fleet credits verbatim; InfoWorld Netflix quotes verbatim; New Stack Rocksandra page truncated at 32,793-char fetch cap, corroborated via search snippets (planetcassandra P99 25-60ms GC detail; source NOT in register, noted for survey pass). Quote-hygiene fix at authoring: "proved unreliable at their volumes" was survey paraphrase, quote marks removed and reworded. Push verified in-program (fetch-back equal: true, 19,943 bytes; corrupted-transcription avoidance: content read in-sandbox, not transcribed). |
| 11 | 2026-11-25 | Phase 3 adversarial review of Ch. 4 | DONE | Two-expert review + fresh machine battery green (8/8 register IDs incl. D1/T07/W1, 10/10 exact URLs in register, 9/9 quote/figure targets in chapter, markers each =1, zero em-dash/placeholders; battery whitespace-normalized after line-wrap false negatives). Four objections fixed: O1 Manhattan motive overstated ("built response to eventual-consistency limits" vs register's "built for millions QPS low latency"; 2016 retrofit is where consistency limits surfaced) and the garbled "later built its own store first" sentence, rewritten with the survey dossier 12 ladder MySQL → Cassandra → Manhattan made explicit; O2 Discord "no downtime" claim removed (neither register source carries it; InfoQ and the blog verified live; reworded to "staged migration with messages migrated last", blog-supported); O3 "the register's verdict" on SimpleDB corrected to "the survey's verdict, dossier 3" (register summary carries no unreliability wording); O4 Instagram pain paren "(write amplification and GC on JVM heaps)" narrowed to "(read latency traced to GC issues)" (planetcassandra and the New Stack snippet corroborate the GC/read framing; write amplification is only class-level in the survey's engine-swap synthesis). Pattern census: P3/P9/P12 load-bearing (spec), P10/P19 load-bearing forward pins (Ch. 12/Ch. 15+22), P8 retained (load-bearing in the ●●○ worked answer: operator capacity). Live probes: InfoWorld carries the quotes but not the 2008 outage (chapter already marks it register-recorded); DevPro Netflix "10,000 instances" verbatim; InfoQ confirms nine days/three months. Chapter status: REVIEWED. Chapter commit e24874ad (equal: true). Next: Ch. 5 |
| 12 | 2026-11-25 | Ch. 5: Columnar — fan-in and fan-out | DONE | Chapter master `manuscript/ch05-columnar.md` (~17.4 KB, byte parity with canvas except 6-line front matter): per BOOK-PLAN ("why the same engine serves a CDN, a trading terminal, and a particle detector"); C17 Cloudflare fan-in (T05 CitusDB-Postgres→ClickHouse at 6M req/s; T06 Elasticsearch logs→"much smaller" cluster, inserter CPU/mem "reduced by eight times", FP), C18 Bloomberg fan-out (VP; "querying about a hundred million securities all at once"; on-prem immutable; ZooKeeper stress during MergeTree clones→DR drills + Keeper), C19 CERN LHCb (PR; metadata on 10B events, 1000+ attributes), F08 opened as the FC3 teaser ("massive lock contention" in the engine mutex; full read Ch. 19); patterns P2 + FC3 preview; one-engine-three-shapes comparison table; graded exercises + ◆ diagnostic. Battery green: 8/8 register/transition/failure IDs (C17, C18, C19, T05, T06, F08, FC3, P2), 6/6 exact URLs register-copied, markers each =1, zero em-dash, zero placeholders. Live probes: CF HTTP analytics + CF logs posts verbatim; Wikipedia C19 verbatim; F08 verbatim; ClickHouse-hosted pages fetch as ~471-char shells (C17-VP, C18), corroborated via search snippets (Bloomberg quote verbatim in snippet; quadrillion scale via news). Record correction: session-11 O2 was over-turned, the Discord blog carries "we need to migrate trillions of messages with no downtime" verbatim (probe context truncation hid it), Ch. 4 C38 bullet now quotes the requirement as stated (ch04 commit f648106), register T07 "no downtime" stands supported (no emendation needed). Grammar fix after first push (missing period, commit c7c98ecf). Chapter commits: 88486734 (first push), c7c98ecf (final). Next: Phase 3 review of Ch. 5, then Ch. 6 |
| 13 | 2026-11-25 | Phase 3 adversarial review of Ch. 5 | DONE | Two-expert review + fresh machine battery green (9/9 register IDs incl. C46, 8/8 corpus/page quote targets, 6/6 exact URLs register-copied, markers each =1, zero em-dash/placeholders; canvas-manuscript parity exact). Definitional checks: P2's registered rule ("split by access pattern, not one flagship product"), fails-when ("semantic translation assumed free") and recorded face (C46) read from the P-index; FC3 ("unprofiled layer: F08") read from ANALYSIS, chapter formulation matches both. Five objections fixed: O1 "MergeTree clone" was an unglossed elaboration not in the survey's wording ("clones"), dropped; O2 "3,000-person physics collaboration" headcount unsupported, removed; O3 P2 block rewritten: register face C46 named, Cloudflare T05/T06 pair reframed as an instance plus a proposed second face for the survey pass, fails-when aligned to the registered wording with C46's savepoint-timeout friction at 92% done as the cautionary template (the "big bang" fails-when was unregistered); O4 C17 lead corrected: the live post shows 6M logs/s belongs to the old pipeline's Kafka tier, "in the later configuration" dropped and the quote re-anchored; O5 strengthening: §3 C17 gained the one-time ETL of over 60 billion rows with consistency checks and the old-pipeline shutdown (verbatim on the live post). Consistency patch: ●○○ worked answer's "the face" aligned to the register (C46). Live probes: CF HTTP anatomy (old pipeline 2014, Kafka tier, RollupDB, Citus workers, 36 CH nodes, ~40 QPS vs 15 rate limits), verified verbatim. One edit-slip caught: none this session. Chapter status: REVIEWED. Next: Ch. 6 |
| 14 | 2026-11-25 | Ch. 6: Graph — three jobs | DONE | Chapter master `manuscript/ch06-graph.md` (~15.4 KB, byte parity with canvas except 6-line front matter): per BOOK-PLAN ("the common denominator is the multi-hop query; the three cases form a maturity ladder from exploration to production ML on graphs"); C20 ICIJ Panama Papers (FP/press; "a network of around 200 journalists in more than 65 countries", 11.5M files, Tika/Solr/Neo4j/Linkurious documented in the ICIJ's own interview; Cabra's "It's magic" quote via Computer Weekly, corroborated live in the CW search snippet), C21 Zurich Insurance (VP PDF fetched live: "500,000 claims", "a million calls" a year, "to make quick decisions"; rule-based stack context; unaudited figures flagged), C22 eBay ShopBot (VP, one interview; "all about the synergy between graphs and machine learning"; read as direction record); maturity-ladder comparison (exploration → acceleration → inference; blast radius and verifiability gradient); graded exercises + ◆ diagnostic. Pattern finding: ANALYSIS cites no C20-C22 pattern and no migration fires in the three cases, so the chapter names the family rule (multi-hop, buy-not-build) + P15 contrast; BOOK-PLAN's P2 cell is flagged in chapter §6 as a spec correction recorded here). Battery green: 5/5 IDs, 4/4 exact URLs register-copied, 10/10 quote targets, markers each =1, zero em-dash/placeholders, zero CJK artifacts (during-typing sed typos caught and fixed pre-push). Live probes: Neo4j ICIJ interview verbatim, Zurich PDF verbatim (6.4k chars), eBay interview verbatim; Computer Weekly page dead fetch (25 chars), Cabra quotes corroborated via search snippets (ZDNet, neo4j news, CW). Mar Cabra's title corrected in a follow-up commit. Chapter commits: 6bec9a83 (first push), f92ffc72 (title fix). Next: Phase 3 review of Ch. 6, then Ch. 7 (Time-series) |
| 15 | 2026-11-25 | Phase 3 adversarial review of Ch. 6 | DONE | Two-expert review + fresh machine battery green (4/4 register IDs, 11/11 verbatim targets, 4/4 exact URLs register-copied, markers each =1, zero em-dash/placeholders; canvas-manuscript parity exact). Five objections fixed: O1 C22's "as a virtual shopping assistant" was a stitched fragment: replaced by the interview's full verbatim framing ("...use Natural Language Understanding to bake machine learning into the graph database that drives eBay's virtual shopping assistant, eBay ShopBot", live-verified); O2 §5's "the vendor's claims about it ('magic')" mislabeled the ICIJ editor's own verdict, reworded; O3 "the consortium later opened parts of the graph data publicly" was not register-sourced, dropped; O4 evidence labels aligned to the register (C20 = FP/VP: speaker first-party, channel vendor's; chapter §cases intro and §3 corrected); O5 the multi-hop quote in the governing idea now carries its attribution ("the survey's characterization"; the sentence lives in the SURVEY comparison paragraph graded as assessment). Live probe: eBay interview full sentence verbatim with its completion recovered (earlier fetches had truncated it mid-word). Chapter status: REVIEWED. Chapter commit 31760765 (equal: true). Next: Ch. 7 |
| 16 | 2026-11-25 | Ch. 7: Time-series — the family both giants built | DONE | Chapter master `manuscript/ch07-time-series.md` (~15.4 KB, byte parity with canvas except 6-line front matter): per BOOK-PLAN ("at giant cardinality, cardinality+downsampling+retention economics aren't solved together by products; both built and open-sourced; below it, the generalist wins (F09)"); C23 Uber M3 (FP; "common solutions like Prometheus and Graphite become difficult to manage and sometimes cease to work" verbatim; ~2,500 queries/s, 8.5B data points/s, ~35 Gbps verbatim; downsampling at collection; ~6.6B series from the press carry, register-recorded), C24 Netflix Atlas (FP; Epic passage verbatim from the atlas-docs overview: "perl CGI scripts, RRDTool logging, and MySQL", ~2M series "regularly failing to keep up"; 1.2B series failover figure verbatim via indexed excerpt, live page now a stub, noted in-chapter; alerting rebuild: "45 minutes late", "by 5 times", streaming eval), F09 NexusTrade counter-case (SR; title-quoted receipt "I didn't last two weeks"; "continuous aggregates and compression" "worked... for a while"; "really *had* time-series collections"; rolled back to Mongo native, cheaper); T17 two-mountains framing (store, then query/alert layer); patterns P15 build-side faces + FC5; two-directions comparison table; graded exercises + ◆ diagnostic. Battery green: 6/6 IDs (C23, C24, F09, T17, P15, FC5), 6/6 exact URLs register-copied, 12/12 verbatim targets, markers each =1, zero em-dash/placeholders, zero CJK. Live probes: both Uber posts verbatim, atlas-docs verbatim, NexusTrade verbatim; Netflix TechBlog URLs dead (142-char Medium shells), quotes corroborated via indexed excerpts and noted in-chapter. Two self-caught fixes pre-push: a guessed quote completion ("over-shadowed... [issues]") replaced with paraphrase; a nonexistent "Chapter 25" cross-pin corrected to Part II. Chapter map Ch. 7 line matched to BOOK-PLAN. Chapter commits: abc43eb0 (first push), 735ff507 (cross-pin fix). Next: Phase 3 review of Ch. 7, then Ch. 8 (NewSQL) |
| 17 | 2026-11-25 | Phase 3 adversarial review of Ch. 7 | IN PROGRESS | Two-expert review of `manuscript/ch07-time-series.md`, fresh machine battery + live-source probes. Open questions: the ~6.6B-series press-carry attribution; bracketed paraphrases of verbatim quotes; the M3 DAG-engine claim's sourcing; the T17 cross-pin (migration playbook = Ch. 22); FC5 fails-when wording vs ANALYSIS. |

## Standing to-do

1. ~~Push bootstrap to GitHub repo~~ (done, session 1).
2. ~~Phase 1 scope gate~~ (PASSED, session 1).
3. ~~Author Ch. 1~~ (DONE, session 4; Phase 3 adversarial review is the next unit).
4. Fold R6 dossiers into `corpus/SURVEY.md` at authoring time (Ch. 1's C56/C61 done as dossiers 22a/22b; the rest pending as their chapters are authored).
5. Planning gaps found in review (session 5): C43 (Durable Objects) and C53 (Fizzy SQLite tenancy) have no chapter home in the chapter map (P18's fails-when is scheduled nowhere); SURVEY dossier 4 still cites the Vitess history page for the first-resharding quote (Percona is the primary; register already amended): fix at the next survey-revision pass or when the owning chapter is assigned.

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
