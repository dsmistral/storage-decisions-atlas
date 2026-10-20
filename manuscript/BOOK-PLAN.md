# manuscript/BOOK-PLAN.md — Chapter specs

**Approved (Phase 1 user gate, session 1). Primary audience: working engineers /
architects** — technical depth favored: access patterns, operational detail,
failure mechanics; decision framing serves the engineer who runs the system.

One chapter = one working session (split-on-trouble allowed). Each spec lists:
scope, register inputs, patterns taught, and the chapter's single governing idea.

## Part I — The families

| Ch | Title | Single idea | Inputs | Patterns |
|---|---|---|---|---|
| 1 | Relational: three ways to stay relational | Same family, three topologies (split / scale-up / monolith): keep SQL semantics, change where rows live — "scale forces distribution" is falsified | C01, C02, C03 (+C47 Vitess history; C37 Uber as the family's cautionary twin) | P4, P6, P7, P8 |
| 2 | Document: three reasons, one aggregate | The fit test is the aggregate (listing, player, document set) accessed whole and rarely joined — not "unstructured data"; plus the quiet return preview (T19) | C06, C07, C08, C09; T10, T19 | P2, P15 |
| 3 | Key-value: one pattern, three scales | The design is invariant (single-key, no joins, availability over read-modify-write); the cost lever changes — call discipline, not schema discipline | C10, C11, C12, C13 | P1, P2, P16 |
| 4 | Wide-column: three fleets and the JVM cure | The family's recurring pain is operational (GC, repair, hot partitions); the recurring cure is an engine transplant, not a new database | C14, C15, C16, C38 (+C36 Twitter's Cassandra era) | P3, P9, P12 |
| 5 | Columnar: fan-in and fan-out | Columnar's fit is scanning few columns across many rows — why the same engine serves a CDN, a trading terminal, and a particle detector | C17, C18, C19; T05, T06 | P2, FC3 preview (F08) |
| 6 | Graph: three jobs | The common denominator is the multi-hop query; the three cases form a maturity ladder from exploration to production ML on graphs | C20, C21, C22 | P2 |
| 7 | Time-series: the family both giants built | At giant cardinality, cardinality+downsampling+retention economics aren't solved together by products — both built and open-sourced; below it, the generalist wins (F09) | C23, C24, F09, T17 | P15, FC5 |
| 8 | NewSQL: three triggers, one destination | SQL semantics survived every move — the transition is topological, not semantic; build vs. buy across three eras (2012/2013–19/2022–23) | C25, C26, C27, C28; T03, T04 | P9, P11, P15 |
| 9 | In-memory: memcached plus your ops wrapper | Nobody runs vanilla memcached at scale; the family's real-world unit is the hardened fork + proxy; who owns correctness (app vs. infra) is the split | C29, C30, C31 | P1, P8 |
| 10 | Vector: where the index lives | If vectors are a feature, a library keeps ops near zero; if retrieval is the product, an engine earns its keep; the unwind is documented both directions | C32, C33, C34, C35; T14, T18, T20 | P16, W1 |

## Part II — The transitions

| Ch | Title | Single idea | Inputs | Patterns |
|---|---|---|---|---|
| 11 | The transition matrix: how to read it | Which moves have public cases, which don't, and what the gaps mean (survivor bias as a finding) | T01–T20 survey | P19, P20 |
| 12 | Relational exits | The pioneering exits and their honest afterlives: right for their time, and later left behind | T01, T02; C15, C36, C46 | P1, P2, P3, P10 |
| 13 | Scale-out relational done right | The four properties shared by every successful migration; the seductive wrong key | C01 (Slack/GitHub), C04, C05, C47 | P4, P5, P6, P7, P19 |
| 14 | The NewSQL absorptions | From relational and wide-column into distributed SQL across three eras of market maturity — the re-check-the-market rule demonstrated chronologically | T03, T04; C25, C26, C27, C28 | P9, P11, P15 |
| 15 | The engine-swap twins | Same protocol, new internals: the lowest-blast-radius move, independently invented three times | T07, T08, T16; C41, C16, C42 | P12, P19, P20 |
| 16 | Returns and round trips | The quiet document→relational return (small/mid only), the vector unwind both ways, the two-week TimescaleDB round trip, and Uber's RDBMS→RDBMS | T10, T11, T13, T19, T20; C44, C45, F09, F11 | P8, P20, FC5 |

## Part III — The failures

| Ch | Title | Single idea | Inputs | Patterns |
|---|---|---|---|---|
| 17 | Backup failures: the class that kills | Half the catastrophic cases are backup failures, not database failures; restore-path diversity is the deliverable | F02, F04, F05, F06 | FC1 |
| 18 | Distribution's own failure modes | Adopting distribution means adopting its failure model: unmonitored shard growth, split-brain during brief partitions | F01, F03; F10 | FC2, P5 |
| 19 | Process and verification failures | Runbooks are code (guardrails, interlocks); post-migration verification must profile the new engine's internals | F07, F08 | FC3, FC4 |
| 20 | Wrong choices, wrong transitions | The same species of mistake inside the dossiers: wrong design key, wrong published justification, wrong "purpose-built" assumption, wrong architecture | F09, F10, F11, F12 | FC5, P8, P20 |

## Part IV — Synthesis

| Ch | Title | Single idea | Inputs |
|---|---|---|---|
| 21 | The five real drivers | Availability/blast radius, ops burden, cost at scale, correctness requirements, team skill — technology is downstream of these | All drivers across corpus |
| 22 | The migration playbook | Dual-write → backfill → shadow-read diff → incremental cutover → warm rollback; budget as much for verification tooling as the data move | P19, P20 firings |
| 23 | The decision framework | Seven rules: start boring; vertical first; natural tenant key; consistency matched to money; ops cost first-class; re-check the market before building; migrate reversibly | P-index whole |

## Appendices

- **A. The case register** — from `corpus/REGISTER.md` (C01–C47, T01–T20, F01–F12)
- **B. Methodology and the evidence ladder** — FP/VP/PR/SR, survivorship bias, unaudited figures, conflicts and caveats (from `CONVENTIONS.md` + `corpus/SURVEY.md` methodology and open-questions sections)
- **C. Open questions, deferrals, watch-lists** — from `corpus/STATE.md` (D1–D5, W1–W3)

## Exercise system note

Per `CONVENTIONS.md`: graded ●○○ / ●●○ / ●●● per chapter; ◆ diagnostics at least one per Part (reader diagnoses which pattern applies, no hints); exercises built only from real constraints (no invented company claims).
