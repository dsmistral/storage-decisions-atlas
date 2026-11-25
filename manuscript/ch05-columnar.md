# Chapter 5 - Columnar: Fan-In and Fan-Out

**Governing idea.** A column-oriented store reads a few columns across very
many rows without touching the rest, which is why the same engine family ends
up serving three shapes of client: a content-delivery network drinking its own
telemetry (fan-in), a financial terminal with thousands of readers each
sweeping an entire market (fan-out), and a particle detector cataloguing the
metadata of ten billion collision events (science's version of both at once).
The cases in this chapter all chose ClickHouse, and the lesson is not that
ClickHouse is special; it is that the column store's fit is defined by one
question: does the interesting work touch few columns but many, many rows? Its
documented costs are the shadow side of that bargain: constant background
merges, careful ingest batching, and (in the 2019-2022 era) a ZooKeeper
dependency that one of the three cases was already pushing toward a cure.

**Cases in this chapter:** C17 (Cloudflare: HTTP analytics and the log-pipeline
move, T05 and T06 together, first-party), C18 (Bloomberg: the fan-out case,
vendor-published), C19 (CERN LHCb: metadata for 10 billion events, press).
Register IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`. The chapter also opens failure post-mortem F08 as the
FC3 teaser: a migration that went right and still regressed, years later, in
the layer nobody profiles.

---

## 1. Context: one question defines the family

Row stores read a row and bring all of it along. Column stores read a column
and bring nothing but it. For a dashboard that aggregates one field over a
billion rows, the difference is not 20 percent; it is the difference between a
scan the hardware can finish before the coffee cools and a scan that never
finishes, because the row store paid for columns it never opened. Every claim
in this chapter traces to deployments of one column store, but the family's
bargain is general: massive read aggregation, at the price of an ingest path
that must be batched and merged.

The bargain's operational bill is worth naming before the cases, because it
is what the operators in them actually fight. Append-only parts accumulate and
must be merged in the background, so ingest size is a tuning surface and
"too many small inserts" is a documented anti-pattern. Mutations are
expensive rewrites, so the schema you design at ingest is close to the schema
you are stuck with. And the replication and coordination story in the era
these deployments were built leaned on ZooKeeper, a separate distributed
system with its own operational load. None of this is disqualifying; two of
this chapter's three operators live with it at extraordinary scale. But it is
the honest list of what columnar asks in return for its scans.

## 2. The cases, each against its time

### The fan-in case: Cloudflare (C17)

Cloudflare's HTTP analytics pipeline ingests logs from its global edge
network; the pre-ClickHouse pipeline scaled out PostgreSQL via CitusDB, with
its Kafka tier ingesting Cap'n Proto logs at "6M logs per second" ([C17:
Cloudflare HTTP analytics](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/), FP).
The stakes are self-analytics: Cloudflare runs its own product dashboarding
and its own operational analytics on the platform. The move to ClickHouse is
first-party documented and pairs two distinct workloads, which the register
records as two transitions:

- **T05, Postgres-based OLAP to columnar:** the CitusDB-Postgres pipeline was
  replaced by ClickHouse for the HTTP-analytics path, at 6M requests per
  second ([C17: Cloudflare HTTP analytics](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/), FP).
- **T06, search engine to columnar:** a critical log-producing application
  moved "from Elasticsearch to a much smaller ClickHouse cluster", with CPU
  and memory consumption on the inserter side "reduced by eight times" ([C17:
  Cloudflare log analytics](https://blog.cloudflare.com/log-analytics-using-clickhouse/), FP).

Read the pair together and P2 (workload-match the migration) appears in an
unusual, disciplined form: Cloudflare did not declare a flagship "analytics
database migration"; it moved the HTTP-analytics workload and the logs
workload as separate projects, each against its own access pattern, each with
its own engine. The vendor's later showcase places the platform at
quadrillion-row scale ([C17: ClickHouse blog, Cloudflare case](https://clickhouse.com/blog/cloudflare), VP).
This is P2's rule working in the gentle direction: because the workloads were
matched separately, each migration could fail or succeed on its own terms.

### The fan-out case: Bloomberg (C18)

Bloomberg runs ClickHouse on-prem for market data, and the meetup report the
register carries distills the access pattern into one sentence: optimizing
for "querying about a hundred million securities all at once", and then
"what if then a thousand people do the same thing" ([C18: ClickHouse blog,
Bloomberg meetup report](https://clickhouse.com/blog/nyc-meetup-report-large-scale-financial-market-analytics-with-clickhouse), VP).
The data is immutable; the reads are mercilessly wide; there is no write
amplification problem because there are almost no updates. On cloud-database
fashion terms this deployment is unfashionable on purpose: on-prem, done for
latency and control, serving the flagship product.

Where Cloudflare's pain arrives with writes, Bloomberg's arrives with
coordination: the register records ZooKeeper stress during clone
operations pushing the team toward disaster-recovery drills and the ClickHouse
Keeper direction (register C18, VP). The case is vendor-published rather than
first-party, and the chapter's grading holds that line: the access-pattern
sentence is a talk report, not an audited account.

### The science case: CERN LHCb (C19)

CERN's LHCb experiment implemented ClickHouse to store and process metadata on
10 billion events with over 1000 attributes per event ([C19: Wikipedia,
ClickHouse](https://en.wikipedia.org/wiki/ClickHouse), PR). The fit is
architectural poetry: detector events are append-only, analysis queries sweep
a few of the thousand attributes across all events, and nobody ever updates
last month's collision. The case is press-recorded rather than first-party in
the corpus, so it functions as the family's breadth proof rather than as an
operations case study: the same shape serves a physics
collaboration that will never read a vendor blog post.

### Companion: the teaser from Part III (F08)

The register carries a Cloudflare billing incident worth opening here because
it reverses the direction of the chapter's optimism: after the ClickHouse
migration, the jobs that produce customer bills slowed dramatically while
every usual hardware metric stayed normal ([F08: Cloudflare billing
pipeline](https://blog.cloudflare.com/clickhouse-query-plan-contention/), FP).
The live page's own diagnosis: "The problem wasn't CPU-bound work; it was
massive lock contention", a mutex inside the engine, visible only through
query-plan forensics. Cloudflare instrumented, diagnosed, and contributed
fixes upstream. F08 is failure class FC3, migrations regress in the
unprofiled layer, and this chapter reads it as a preview: the column store
modernized a bill-producing system, and the regression that followed came
from inside the engine, not from the migration's data logic. The full
post-mortem lives in Chapter 19.

## 3. What happened (sourced outcomes, with evidence levels)

- **Cloudflare (C17, FP).** Postgres-OLAP pipeline replaced by ClickHouse at
  6M requests per second (T05); logs moved from Elasticsearch to a much
  smaller cluster with inserter CPU and memory reduced by eight times (T06)
  ([C17: Cloudflare HTTP analytics](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/); [C17: Cloudflare log analytics](https://blog.cloudflare.com/log-analytics-using-clickhouse/));
  the cutover was completed by a one-time ETL of over 60 billion rows with
  consistency checks, after which the old pipeline shut down
  ([C17: Cloudflare HTTP analytics](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/), FP).
- **Bloomberg (C18, VP).** On-prem ClickHouse for immutable market data,
  sized for thousands of concurrent whole-market sweeps; ZooKeeper stress
  during clones pushed DR drills and the Keeper direction ([C18: ClickHouse
  blog](https://clickhouse.com/blog/nyc-meetup-report-large-scale-financial-market-analytics-with-clickhouse)).
- **CERN LHCb (C19, PR).** ClickHouse implemented for metadata on 10 billion
  events with over 1000 attributes per event ([C19: Wikipedia,
  ClickHouse](https://en.wikipedia.org/wiki/ClickHouse)).
- **Cloudflare again (F08, FP).** The post-migration billing aggregation
  slowed with normal-looking hardware metrics; the cause was lock contention
  in the engine, fixed and contributed upstream ([F08: Cloudflare blog](https://blog.cloudflare.com/clickhouse-query-plan-contention/)).

## 4. The comparison: one engine, three client shapes

| Test | Cloudflare (fan-in) | Bloomberg (fan-out) | CERN LHCb (science) |
|---|---|---|---|
| The workload | edge telemetry, 6M/s | immutable market data | detector event metadata |
| The access pattern | few columns, endless rows, millions of writers | wide sweeps, thousands of readers | few of 1,000+ attributes, all events |
| What moved in (or why this engine) | CitusDB-Postgres pipeline; Elasticsearch logs | engine adopted on-prem from the start | metadata store beside the detector |
| The pain | ingest design, small-batch merges | coordination under clones (ZooKeeper) | recorded thin (PR) |
| The shadow price | F08: bill pipeline regressed years later | DR drills + Keeper as the cure | none on record |
| Evidence level | FP | VP | PR |

**Cross-case comparison paragraph.** The three cases share the engine and
split on the direction of their data flow, which is the chapter's real axes:
fan-in (Cloudflare) buys throughput with ingest discipline and pays in
background merging, fan-out (Bloomberg) buys read breadth with coordination
tooling and pays in ZooKeeper-era operational drills, and the science case
inherits both bargains with none of the publicity. The family's decision rule
survives across all three: if the interesting work is aggregation over few
columns of very many rows, the column store's read arithmetic wins even when
the operational bill looks alarming. What varies is where the bill lands, and
that is knowable in advance: count your writers against your mergers
(Cloudflare's problem), count your coordinators against your readers
(Bloomberg's problem). If neither count is extreme, the case for columnar is
thin, and the F08 teaser adds the standing warning: the regression that comes
*after* the migration can come from the engine's internals, where your
profiling has never looked.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Cloudflare (C17):** exemplary, and the corpus's cleanest two-workload
  columnar adoption. The P2 discipline (separate moves for separate access
  patterns) made each migration legible, and the company's willingness to
  publish the later F08 regression is what makes this corpus trust its
  first-party record. Verdict: correct then, still correct now.
- **Bloomberg (C18):** exemplary but vendor-lensed. The access-pattern
  sentence is the family's best articulation of fan-out, yet everything
  comes through the vendor's meetup report; the operational specifics
  (ZooKeeper stress, Keeper) are direction-grade, not audited. Verdict:
  believable, evidence level honestly cap-limited at VP.
- **CERN LHCb (C19):** the right case for breadth, the wrong case for depth.
  One press-recorded line is all the corpus holds, and this book does not
  inflate thin sources; its value is demographic: the family reaches
  users who buy nothing and blog nothing. Verdict: correct, unverifiable in
  detail, and that is the finding.
- **The F08 teaser:** a migration that did everything right and still
  produced a multi-year bill-pipeline regression. Verdict: adopt columnar
  for the scans, but profile the engine's internals when the workload
  becomes load-bearing for money; FC3 is not a corset that only fits other
  people's migrations.

## 6. The pattern

- **P2 - Workload-match the migration.** Split by access pattern, not one
  flagship product. Register's recorded face: C46, Amazon's consumer Oracle
  exit (ledger to DynamoDB, transactional to Aurora, analytics to Redshift).
  Cloudflare's T05/T06 pair is the same rule in disciplined form: two
  independent workloads moved as two independent projects (proposed here as
  a second face for the survey pass). Fails-when: semantic translation, in
  the register's wording savepoints and PL/SQL, is assumed free; C46's
  savepoint-timeout friction at 92% done is the cautionary template, read
  fully in Chapter 12.
- **FC3 - Migrations regress in the unprofiled layer** (failure-mode class,
  previewed via F08). The billing jobs that slowed shared nothing with the
  migration's own verification surface: hardware metrics looked fine while a
  single engine mutex serialized the critical path. Fails-when: the
  verification plan only measures the layers the migration touched.

## Exercises

**●○○ (identify the pattern).** An analytics team replaces a Postgres OLAP
pipeline with a column store and, in a separate project the next quarter,
moves its log search from Elasticsearch to the same engine. Name the pattern
the team executed and what it protected them from.

*Hint:* the two moves failed or succeeded independently by design.

*Worked answer (assessment):* P2, workload-match the migration (Cloudflare's
T05/T06 pair is the face). Splitting the two moves meant each could be sized,
tested, and rolled back against its own access pattern; a single flagship
migration would have coupled the engine choice for both workloads and made
one bad fit escape into both.

**●●○ (apply the framework).** A payments company wants to move its
transaction ledger into the same column store it uses for analytics, arguing
that "aggregation is aggregation". Apply the chapter's decision rule: what
counts should be taken before the move, and what does the F08 teaser predict?

*Worked answer (assessment, judgment):* count writers against mergers and
coordinators against readers: a ledger is update-heavy and consistency-bound,
the exact inverse of immutable append-only analytics, so the columnar read
arithmetic never gets to pay for its operational bill. Merging mutation
semantics into an append-tuned engine couples the ledger's correctness to the
engine's most-manual layers, and F08's lesson is that the regression arrives
in precisely the unprofiled interior when the workload becomes load-bearing
for money. Keep the ledger where verification is strongest; fan the OLAP out
of it.

**●●● (steelman, then weigh).** Write the strongest case that Bloomberg
should have run its market analytics in the cloud on a managed columnar
service, then weigh it against the register's record. End with one
transferable rule about when the on-prem choice is rational.

*Steelman and verdict (assessment):* (i) the steelman: managed columnar
services now sell the same scans with keeper and backup tooling included, and
Bloomberg carries the ZooKeeper-era cost structure precisely to retain
control it could rent; (ii) the weigh: the register records the constraint
set as latency, control, and a flagship product whose competitive identity
includes that control, and vendor-published evidence cannot reverse a
decision premised on not owning the vendor's incentives. The rational on-prem
zone is exactly where the workload IS the product: when the data and its
latency characterize what customers pay you for, renting the engine means
renting the product's defining property. Cloudflare, whose analytics are
load-bearing but not customer-facing, works equally well either way, which is
why its case is instructive and Bloomberg's is distinctive.

**◆ (diagnostic: no hints).** Three teams, one sentence each: is the columnar
bargain actually the bargain this team is making? (i) A team reads five
columns across two billion rows to compute hourly funnels. (ii) A team reads
one row by key, sometimes forty columns at once, microseconds matter. (iii) A
team appends a few thousand small batches per second and wonders why
background merges dominate CPU.

*(Diagnostic answers, assessment: (i) the textbook fit, columnar pays
immediately; (ii) not the family's bargain, that is a point-lookup workload,
and columnar buy-in would ship the ingest bill without ever exercising the
scan arithmetic; (iii) the family's documented anti-pattern, ingest discipline
is the price of the scans, and the batch-size lesson is Cloudflare's
log-pipeline post in one failure.)*

## Sources

All from the register; URLs copied exactly.

- C17 Cloudflare HTTP analytics (CF blog, FP): https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/
- C17 Cloudflare log analytics (CF blog, FP): https://blog.cloudflare.com/log-analytics-using-clickhouse/
- C17 Cloudflare case showcase (ClickHouse blog, VP): https://clickhouse.com/blog/cloudflare
- C18 Bloomberg market analytics meet-up report (ClickHouse blog, VP): https://clickhouse.com/blog/nyc-meetup-report-large-scale-financial-market-analytics-with-clickhouse
- C19 CERN LHCb deployment (Wikipedia, PR): https://en.wikipedia.org/wiki/ClickHouse
- F08 Cloudflare billing query-plan contention (CF blog, FP): https://blog.cloudflare.com/clickhouse-query-plan-contention/
