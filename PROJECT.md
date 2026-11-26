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
**Chapter 7 REVIEWED (session 17, 2026-11-25):** review passed with 4 fixes
(the 6.6B figure quoted verbatim from the press carry; the alerting quotes
restored to verbatim segments; the T17 DAG detail re-attributed to the
survey's characterization; the Ch. 22 pin corrected). Receipts in row 17.
Next unit: Ch. 8 (NewSQL: three triggers, one destination; C25, C26, C27,
C28; T03, T04; P9, P11, P15), then its review.
**Chapter 8 AUTHORED + AUTHOR-CHECKED (session 18, 2026-11-25):** manuscript
master `manuscript/ch08-newsql.md` ("NewSQL: three triggers, one
destination"; era-shaped: C27 Google 2012 build, C28 Alibaba 2013-19 build,
C25 DoorDash + C26 Pinterest 2022-23 buys; T03/T04; patterns P9, P11, P15;
three-triggers comparison; graded exercises + ◆ diagnostic). Battery green
(12/12 exact URLs register-copied, 28/28 quote targets
whitespace-normalized-verified, markers each =1, zero em-dash/placeholders).
Live probes: Google Cloud blog, dbdb.io, Alibaba Cloud case study, DoorDash
post, InfoQ 2026 verbatim live; The New Stack = cookie boilerplate before
text under the fetch cap, quotes via indexed excerpts noted in-chapter;
Pinterest Medium pages block fetches, quotes via indexed excerpts. Next
unit: Phase 3 review of Ch. 8, then Ch. 9 (In-memory: C29, C30, C31 + C57/C60
Aerospike, R6).
**Chapter 8 REVIEWED (session 19, 2026-11-25):** review passed with 3 fixes
(the Demirbas channel label; the Spanner-product convergence claim
downgraded to assessment with OceanBase as the register-sourced face; the
~10% per-query assessment pinned to SURVEY dossier 11). Receipts in row 19.
Next unit: Ch. 9 (In-memory: memcached plus your ops wrapper; C29, C30, C31
+ C57/C60 Aerospike at Indian scale, on-prem, R6), then its review.
**Chapter 9 AUTHORED + AUTHOR-CHECKED (session 20, 2026-11-25):** manuscript
master `manuscript/ch09-in-memory.md` ("In-memory: memcached plus your ops
wrapper"; C29 EVCache, C30 Facebook Memcache, C31 Twemcache + the R6
extension C57 PhonePe and C60 Flipkart on Aerospike, on-prem; the
correctness-owner split as the chapter's spine; patterns P1, P8 + P16
caution per ANALYSIS; T26 preview; graded exercises + ◆ diagnostic).
Battery green (8/8 exact URLs register-copied, 32/32 quote targets
whitespace-normalized-verified, markers each =1, zero em-dash/placeholders).
Live probes: EVCache wiki, NSDI'13 abstract, Twemcache post, PhonePe
customer story + fraud-model post, Flipkart blog + press release all
verbatim; BigDATAwire dead-fetch noted in-chapter. No SURVEY fold needed
for the R6 cases (register rows + ANALYSIS line 67 carried them; quotes
verified at the sources). Next unit: Phase 3 review of Ch. 9, then Ch. 10
(Vector: where the index lives; C32, C33, C34, C35; T14, T18, T20; P16, W1,
per BOOK-PLAN).

**Chapter 9 REVIEWED (session 21, 2026-11-25/26; close redone after the
session's first close pushes did not land):** review passed with 7 findings
fixed (receipts in row 21), headlined by the C30 contract-quote
re-attribution (the conference page's Public Summary by Michael Piatek, not
the authors' abstract) and the EVCache zone-claim rewrite; the
fraud-complaints quote restored to full verbatim. Chapter status: REVIEWED.
Next unit: Ch. 10 (Vector: where the index lives; C32, C33, C34, C35; T14,
T18, T20; P16, W1, per BOOK-PLAN), opened in row 22; then its review.

**Chapter 10 AUTHORED + AUTHOR-CHECKED (session 22, 2026-11-26):** manuscript
master `manuscript/ch10-vector.md` ("Vector: where the index lives"; the
placement axis — dedicated engine (C32 turbopuffer), library (C33 Spotify
Annoy→Voyager), embedded index (C34 Confident AI Pinecone→pgvector) — with
the incumbent's counter-flow (C35 Pinecone, users arriving after pgvector)
documenting the unwind in both directions per T14/T18/T20; ANALYSIS
records no pattern rows for C32-C35, so §6 uses the Ch. 6 convention
(placement-axis family rule, P16's caution side, W1 watch held open);
graded exercises + ◆ diagnostic). Battery green (9/9 exact URLs
register-copied, 30/30 quote targets — 25 verbatim, 5 words-verbatim
modulo the sources' markdown bold markup, noted; markers each =1, zero
em-dash/placeholders). All 9 register sources fetched live; Latent Space
at the fetch cap and corroborated in-chapter. Next unit: Phase 3 review
of Ch. 10, then Ch. 11 (The transition matrix: T01-T30 survey).

**Chapter 10 REVIEWED (session 23, 2026-11-26):** review passed with 2
findings fixed (the C33 ceilings parenthetical resourced onto the post's
verbatim in-memory framing, dropping an unsupported filtering-absence
claim; the four-trillion-document precision fix) plus one records
correction (row 22's unverifiable "Jul 2026" date for C34 dropped).
Battery: 28/28 quote targets verbatim (4 modulo curly-apostrophe
variants), 9/9 URLs register-copied, markers each =1, 6 numbered
sections, zero em-dash/placeholders. Chapter status: REVIEWED.
Next unit: Ch. 11 (The transition matrix: T01-T30 survey, per BOOK-PLAN;
Part II opens), then its review.

**Chapter 11 AUTHORED + AUTHOR-CHECKED (session 24, 2026-11-26):** manuscript
master `manuscript/ch11-transition-matrix.md` ("The transition matrix: how to
read it"; a survey chapter over T01-T30: what a T-row is, the matrix grouped
by destination, the dense cells with live-verified verbatim anchors, the
empty cells as recorded findings (survivor bias, D3/D5/D6/D8), P19 + P20 as
the reading lenses, spec state flagged, forward pins to Chapters 12-16 and
watch states W1/W2/W5). Spec note: BOOK-PLAN's "T01-T20" spec cell is stale
(pre-R6); the chapter map's "T01-T30 survey" governs and the chapter flags
it per the Ch. 6 convention. Battery: 30/30 T-IDs cited, 7/7 exact URLs
register-copied, 8 source-level quotes live-verified verbatim, corpus-quote
fragments byte-verified against STATE/REGISTER/P-index, markers each =1, 6
numbered sections, zero em-dash/placeholders. Next unit: Phase 3 review of
Ch. 11, then Ch. 12 (Relational exits).

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
- Ch. 8 NewSQL: three triggers, one destination — C25, C26, C27, C28 (T03, T04; BOOK-PLAN spec inputs): P9, P11, P15
- Ch. 9 In-memory: memcached plus your ops wrapper — C29, C30, C31 (+ C57/C60 Aerospike at Indian scale, on-prem, R6): P1, P8 (+ P16 caution per ANALYSIS; T26 preview)
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

Status: Ch. 11 AUTHORED session 24 (Chs. 1-10 done: all reviewed; 11 authored); Chs. 12-23 planned (specs in `manuscript/BOOK-PLAN.md`).

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
| 17 | 2026-11-25 | Phase 3 adversarial review of Ch. 7 | DONE | Two-expert review + fresh machine battery green (6/6 IDs, 7/7 exact URLs in register incl. the added Packt press carry, 6 sections, markers each =1, zero em-dash/placeholders; canvas-manuscript parity exact). Open questions resolved: the ~6.6B figure is the press carry's own verbatim wording ("now houses over 6.6 billion time series", Packt, search-snippet-verified; the chapter's "active series" wording dropped); the bracketed-paraphrase device removed: "fell behind" was not verbatim ("falling behind" is) and "reach[ing] engineers 45 minutes late" replaced by the full verbatim "critical application health alerts reached engineers 45 minutes late" (recovered from the indexed excerpt of the stub-live TechBlog post); the DAG-query-engine detail is the survey's characterization of register T17 (the register row carries no DAG wording), attribution corrected; the "full comparison is Chapter 22's material" pin was wrong (the giants' comparison is §4 of the chapter; Ch. 22 is the playbook): reworded to "the giants' full comparison is §4 below, and the migration-playbook treatment is Chapter 22's material"; FC5's fails-when is a chapter formulation, consistent with Ch. 5's FC3 convention (no fix). Packt URL added to the chapter Sources (byte-matches register). Chapter commit 6d2acee6 (equal: true). Incident at session open: the open-row edit consumed row 16 (anchor matched the row's start); row 16 restored byte-for-byte from the verified fetch (repair commit 42d24c77). Chapter status: REVIEWED. Next: Ch. 8 (NewSQL) |
| 18 | 2026-11-25 | Ch. 8: NewSQL — three triggers, one destination | DONE | Chapter master `manuscript/ch08-newsql.md` (~22.2 KB, canvas rendered): per BOOK-PLAN ("SQL semantics survived every move: the transition is topological, not semantic; build vs. buy across three eras 2012 / 2013-19 / 2022-23"); era-shaped cases: C27 Google F1/Spanner 2012 (FP live-verified: "was resolute in its desire to never do it again", "By 2012... just completed the migration of AdWords frontend traffic", TrueTime; T04's gold standard), C28 Alibaba/Ant OceanBase 2013-19 (FP live: dbdb.io Alipay Oracle exit + MySQL active-standby consistency gap, three principles, TPC-C 2019, external customers 2017; Alibaba Cloud case study 256,000 payments/s, its text renders "0.25.6 million transactions per second", noted; "~10% per-query gap" = SURVEY dossier graded assessment), C25 DoorDash CockroachDB 2022-23 (FP live: "went up to 500GB very quickly", "single PostgreSQL table size is under 500GB", "10X load", facade-first playbook; Cockroach Labs write-up "crash and burn"; T03's three cutover attempts + "literally a lossless cutover, as far as I can tell" + alert-volume decline "despite running dozens more clusters" via The New Stack indexed excerpts, dead-fetch noted in-chapter), C26 Pinterest TiDB 2022-23 (Medium blocked; "picking 10+ datastore technologies" + Spanner "prohibitively high" + Vitess "cross shard queries may not be optimized" via indexed excerpts, tails truncated noted; InfoQ 2026 live: TiDB still in production, "petabyte-scale data and thousands of pipelines"); T03/T04 transitions; patterns P9 (C26 face), P11 (C27/C28 faces), P15 seller's-view reading; three-triggers comparison table; graded exercises + ◆ diagnostic. Battery green: 4 register cases + 2 transitions + 3 patterns, 12/12 exact URLs register-copied, 28/28 quote targets whitespace-normalized-verified, markers each =1, zero em-dash/placeholders, canvas-manuscript parity exact. Two consecutive copy slips around one sentence opener in the C26 paragraph caught by fetch-back reads and fixed (final commit 2cf28014). Chapter commits: 7b25b2d3 (first push), 2cf28014 (final). Next: Phase 3 review of Ch. 8, then Ch. 9 (In-memory) |
| 19 | 2026-11-25 | Phase 3 adversarial review of Ch. 8 | DONE | Two-expert review + fresh machine battery green (23/23 URL occurrences across 12 unique register-copied URLs, 29 quote targets whitespace-normalized-verified, 6 sections, markers each =1, zero em-dash/placeholders; canvas-manuscript parity exact; full fetch-back read). Three fixes: O1 the C27 Demirbas channel label corrected (blog, SR) → (blog, PR): the author is a third-party commentator, not a participant; O2 the P15 seller-side convergence claim downgraded where unsourced: "both converged into products" → the two builds "outgrew their in-house origin", with OceanBase's record (external customers from 2017, commercial + open source, dbdb.io, live-verified) as the register-sourced face and Spanner's managed-product path marked as the study's context/assessment (§4 sourcing note + §6 block); O3 the ~10% per-query assessment pinned to SURVEY dossier 11 for traceability. Register nit noted, no amend: register C26's Vitess wording spells "cross-shard" while the live post has "cross shard"; the chapter quotes the live spelling. Three edit-tool slips this session (a stray "(unused)" insertion, a corrupted new_string in the dossier-pin edit, a spliced "not years of few years of engineering" phrase) all caught by scans/fetch-back reads and repaired pre-close. Chapter commits: 51eb4237 (review fixes), ca8fc165 (final). Chapter status: REVIEWED. Next: Ch. 9 (In-memory) |
| 20 | 2026-11-25 | Ch. 9: In-memory — memcached plus your ops wrapper | DONE | Chapter master `manuscript/ch09-in-memory.md` (~22.5 KB, canvas rendered): per BOOK-PLAN ("nobody runs vanilla memcached at scale; the real-world unit is the hardened fork + proxy; who owns correctness, app vs. infra, is the split"); C29 Netflix EVCache (FP wiki live: "most of the requests are served by the cache", Ketama sharding, zone affinity; 2-9 replicas register-recorded), C30 Facebook Memcache (FP NSDI'13 abstract live: "may lose or evict data without notice", billions req/s, trillions of items; app-owned invalidation per register + survey comparison), C31 Twitter Twemcache (FP post live: 20+ TB, ~2T queries/day, 23M+ QPS, Twemproxy; the post's next-day self-correction of the slab calcification claim noted); R6 extension C57 PhonePe + C60 Flipkart (Aerospike on-prem at Indian scale; PhonePe VP story live: "380 million registered users", "hundreds of Aerospike clusters... three on-premises data centers", "over 160TB per site"; fraud-model FP post live: 2.5B txns/month, 100M in a day Apr 2022, fraud complaints −38%/−50%, P16 caution on vendor-reported figures per ANALYSIS line 67; Flipkart VP live: 50+ use cases, "an aggregate 90 million queries per second across three data centers", 95M TPS press release, "a core engineering team of fewer than ten developers" → P8's strong-scale face, "self-managed clusters into a unified internal service layer"); patterns P1 (discipline reading), P8 (inverted face), P16 caution; T26 contrast preview (Freecharge managed vs PhonePe self-run, Ch. 16/21). Battery green: 5/5 case IDs + P1/P8/P16 + T26, 8/8 exact URLs register-copied, 32/32 quote targets whitespace-normalized-verified, markers each =1, 6 sections, zero em-dash/placeholders, canvas-manuscript parity exact. Live probes: EVCache wiki, NSDI'13 abstract, Twemcache post, PhonePe story + fraud post, Flipkart blog + press release all verbatim; BigDATAwire dead fetch (142 chars, register-recorded, noted in-chapter); one transient fetch-service error retried once successfully. Self-caught fixes pre-push: typos ("wi'th", "women-and-machines" garble, "shows defaults" garble, "on-premis") and one quote-hygiene violation (paraphrase inside quotes in the C30 paragraph, replaced with the verbatim framing); one corrupting edit mistaken (placeholder marker left in the C30 paragraph) caught by read-back and repaired by line surgery. Chapter commits: 5cc317b0 (first push), 7b3ed79d (final). Next: Phase 3 review of Ch. 9, then Ch. 10 (Vector) |
| 21 | 2026-11-25/26 | Ch. 9 Phase 3 adversarial review | DONE | Two-expert review of `manuscript/ch09-in-memory.md` + fresh battery with 7 live sources re-fetched sequentially (EVCache wiki, NSDI'13 page, Twemcache post, PhonePe story + fraud post, Flipkart blog + press release). Findings fixed: O1 the two C30 contract quotes ("may lose or evict data without notice...", "the semantics of the system as a whole are difficult to state precisely") were misattributed to the authors' abstract: they sit in the conference page's Public Summary by Michael Piatek; §2 now separates the abstract (scale sentence, verbatim) from the Public Summary (contract + its verbatim question "Won't such an imprecise system be incomprehensible to developers and impossible to operate?"), §3 bullet, §5 bullet, §4 and the steelman aligned; O2 EVCache's "survives the loss of an entire zone" claim unsupported by the wiki (its actual factual base: reads local, "the writes are done on both the zones", zone-fallback fetch from the other zone) and the "cache handles staleness via its own replication and invalidation paths" clause unsupported and contradicting §3: both rewritten onto wiki verbatim, EVCache's claim rescoped to redundancy; O3 the 2-9 replicas figure now live-sourced from the wiki verbatim ("Any number of copies of data are supported (some clusters run with 2, others with 9)"), register phrasing noted as summary; O4 P1 label in §5 + the ●○○ worked answer framed as this chapter's discipline reading (registered face stays C10); O5 "P8's rule inverted" reworded to "at strong scale" (matches the P-index face reading in §6); O6 table cells cleaned ("the infrastructure for redundancy, the app for staleness"; "real-time fraud checks" replacing the unsupported "in-transaction checks") plus exercise-hint wording; O7 fresh battery catch: the fraud-complaints quote had silently truncated its parenthetical, restored to full verbatim "(50% in the specific segment of high-value transactions that we are targeting)". Final battery: markers each =1, 6 sections, zero em-dash/placeholders, 8/8 URLs register-copied, 31/31 quote targets verbatim on live pages/corpus (conventional notes: wiki bullet integrated lowercase; SURVEY quote verbatim modulo its *application* emphasis markup); canvas-manuscript parity exact. Incident: this session's first close pushes (claimed 087e4bc8 chapter / a4646441 records) never landed; GitHub verified still at open state ea444543 and the close redone with full verification. Chapter commit: a5b699bd (equal: true, 23,251 bytes on GitHub). Next: Ch. 10 (Vector) |
| 22 | 2026-11-26 | Ch. 10: Vector, where the index lives | DONE | Chapter master `manuscript/ch10-vector.md` (~25.5 KB, canvas rendered): per BOOK-PLAN ("if vectors are a feature, a library keeps ops near zero; if retrieval is the product, an engine earns its keep; the unwind is documented both directions"); placement-axis cases: C32 turbopuffer (FP blog live: Readwise shelving arithmetic "~$5k/month... $20k/month+", $2+/GB 2022 in-memory regime named as the founding cost floor, object-storage + NVMe cache architecture; AI Engineer PR profile: "four trillion documents", "over 25,000 searches per second" Apr 2026 + Anthropic/Atlassian/Cursor/Notion customers, run-rate figure = third-party analyst, kept PR; Latent Space PR interview: Cursor 95% cost cut, vendor-reported, P16 caution stated), C33 Spotify Voyager (FP post live: Annoy 2013 → Voyager 2023 HNSW on hnswlib, "More than 10 times the speed of Annoy", "up to 50% more accuracy", "4 times less memory usage than Annoy", statelessness passage "almost entirely removing the maintenance and cost burden of maintaining a stateful database cluster"; GitHub README FP: "Voyager is used extensively in production at Spotify, and is queried hundreds of millions of times per day"; InfoQ PR "10 years on..." trigger; T18), C34 Confident AI (FP: Pinecone→pgvector, "143% more QPS" with "Data taken from Supabase" marked, "PGVector is Not Perfect Either" honesty section, $160/mo p1.x2 at ~60 QPS 0.99; post date not verifiable from the live page fetch (sidebar dates only), so no date is asserted for this source in the chapter), C35 Pinecone counter-flow (VP Apr 2024 post by three of its engineers: "people have come to Pinecone after trying and failing with pgvector", light-workload cost model "on average, 10 queries per minute", "Pinecone Serverless is cheaper for the four tested datasets", 1.1x-2.2x / 1.5x-2.9x vendor-estimated; JCG PR press carry: 40-60% cost reductions at 10-100M vectors; T20 both directions + W1 verdict: no first-party 100M+ migration either way, watch stays open). Patterns: ANALYSIS records no C32-C35 rows, so §6 uses the Ch. 6 convention (family rule "where the index lives" + P16 caution side + W1 watch state). Battery green: 9/9 exact URLs register-copied, 30/30 quote targets (25 verbatim, 5 words-verbatim modulo the sources' markdown bold markup, noted), markers each =1, 6 sections, zero em-dash/placeholders, canvas-manuscript parity exact. Live probes: all 9 register sources fetched live sequentially; Latent Space fetched at exactly the 32,793-char cap. Self-caught fixes pre-push: CJK artifact in the §4 ops-burden cell removed ("absorbed by the existing Postgres ops"), one register paraphrase kept out of quotation marks with attribution. Chapter commit: 685ab953 (equal: true, 25,537 bytes on GitHub). Next: Phase 3 review of Ch. 10, then Ch. 11 (the transition matrix) |
| 23 | 2026-11-26 | Ch. 10 Phase 3 adversarial review | DONE | Two-expert review of `manuscript/ch10-vector.md` (chapter commit 685ab953) + fresh battery with all 9 register sources re-fetched sequentially (all succeeded, one program total). Sourcing checks all clean: the turbopuffer blog header "Simon Hørup Eskildsen (Co-founder & CEO)" and "Updated: March 05, 2026" verbatim (chapter's title + date supported); the AI Engineer run-rate figure is verbatim "Sacra estimated its annualized revenue run rate at $100 million in March 2026" (third-party analyst, chapter's PR-level handling exact); the Pinecone post byline "Dave Rigby, Janum Shah, Ram Sriharsha Apr 17, 2024" verbatim ("April 2024, by three of its engineers" supported); "Readwise Reader (read-it-later app)" launch verbatim on the turbopuffer blog; the C34 143% sentence confirmed inside the post's own hedging clause ("there are benchmarks that shows...") — the chapter already frames it as cited-benchmark, no change; W1 wording matches the register's watch (paraphrase of "no first-person 100M+ migration either direction"); the "zero-new-operations middle" is the chapter's own coinage, not sourced, and not presented as such (no change); the ●●○ "8 queries per second" premise is fiction-by-design and the worked answer directs the cost model to the team's own QPS (no change). Quote battery: 28/28 targets verbatim on live pages (23 exact after whitespace+bold normalization; 5 resolved as: 4 newer curly-apostrophe variants in the sources ("we're", "Cursor's", "pgvector's") and 1 probe typo — all words verbatim; conventional note). Findings fixed (2): O1 §5's C33 parenthetical "(no cross-service filtering story, an in-memory working set)" carried an unsupported feature-absence claim (neither the Spotify post nor the Voyager README mentions filtering): rewritten onto supported ground ("working-set ceiling... nearest-neighbor search in memory is the post's own framing"); O2 §5's "The trillion-document scale" made imprecise against the chapter's own four-trillion figure: corrected to "The four-trillion-document scale". Records correction: row 22's "Jul 2026" date for the C34 post was not verifiable from the live page fetch (the page shows only sidebar article dates); the date is dropped from row 22's notes and no date is asserted in the chapter (which never carried one). Final battery: markers each =1, 6 numbered sections, zero em-dash/placeholders/CJK, 9/9 URLs register-copied, canvas-manuscript parity exact. Chapter commit: 5a9e2d7d (equal: true, 25,691 bytes on GitHub). Next: Ch. 11 (The transition matrix: T01-T30 survey) |
| 24 | 2026-11-26 | Ch. 11: The transition matrix, how to read it | DONE | Chapter master `manuscript/ch11-transition-matrix.md` (~18 KB, canvas rendered): Part II opens per BOOK-PLAN ("which moves have public cases, which don't, and what the gaps mean (survivor bias as a finding)"; patterns P19, P20). §1 what a T-row is (five fields, provenance R1-R6, the managed-vs-self-run third axis, register-summary vs source-quote distinction); §2 the matrix table (30 rows grouped by destination: columnar 6, KV 4 incl. all-DynamoDB, distributed SQL 3, wide-column 5 with 3 engine swaps, singles T12/T22/T24/T25/T26/T27, bidirectional T20; origin census: 9 relational-origin rows; graph on neither side of any row); §3 dense cells (columnar inflow with Uber verbatim "a single ClickHouse node could ingest 300K logs per second, about ten times what a single ES node could achieve" and Zerodha z-connect verbatim "reduced data storage from 20TB to 4TB while storing hundreds of billions of financial records. The query performance has increased 1000x performance in many scenarios", plus the explicit caution that the page's "performance increase topping ~1000%" belongs to the separate Citus refactor row, not T30; KV gravity incl. Zomato verbatim "our billing system's monthly expenses saw a notable 50% reduction due to these changes" = the register's "~50% monthly cost reduction" source sentence; returns with Infisical verbatim "the migration execution turned out smooth with zero data loss"); §4 empty cells (graph band absent per D6 verbatim "found only vendor/marketing material; re-check later", single-occurrence rows, T20's bidirectionality, silence recorded not guessed: D3/D5/D8 fragments verbatim, survivor-bias verdict (assessment)); §5 the two lenses (P19's faces: T03/T07/T19/T23/T27 published mechanics incl. Yelp verbatim staging sequence "switched the write logic to write to MySQL first, and finally (several months later) turned off Postgres writes entirely and cleaned up much of the code we had written"; P20's faces: T12 "deliberately reversible", T13/F09, T05's asterisk "but later billing regression (F08)"; the three-question reading rule); §6 pattern faces, spec state (BOOK-PLAN "T01-T20" cell flagged stale vs chapter map's "T01-T30 survey" per the Ch. 6 convention), forward pins to Chs. 12-16 + W1/W2/W5, part-level verdict; graded exercises (T24 worked example, columnar-capacity plan, reverse migration design, ◆ P20 diagnostic) + 7-URL Sources (all register-copied). Spec note recorded: BOOK-PLAN line 29 says T01-T20 (stale, pre-R6); the PROJECT chapter map ("T01-T30 survey") wins and the chapter is written against it; battery: 30/30 T-IDs cited, 7/7 exact URLs register-copied, 8 source-level quotes live-verified verbatim zomato/yelp/bhoos/infisical/freecharge/uberlog/zconnect(z2), register/STATE/P-index quote fragments byte-verified against the corpus mirrors; markers each =1, 6 numbered sections, zero em-dash/placeholders/CJK/backslashes, canvas-manuscript parity exact. Conventional notes: nested-quote glyph demotion in the P19 fails-when quote (register's inner "..." rendered '...' inside the chapter's quoted span); Zomato's curly apostrophe (system's) rendered straight; z-connect fetch at the 32,793-char cap (target text well inside). Chapter commit: 4e54ac8b (equal: true, 18,188 bytes on GitHub). Next: Phase 3 review of Ch. 11, then Ch. 12 (Relational exits) |

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
