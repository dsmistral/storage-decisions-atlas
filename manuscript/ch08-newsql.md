# Chapter 8 - NewSQL: Three Triggers, One Destination

**Governing idea.** In every move this chapter reads, the SQL semantics
survived intact: transactions, joins, and a familiar query surface carried
over to the new engine. What changed was the topology underneath them. The
moves are therefore topological, not semantic: nobody rewrote the meaning of
their data, everybody rebuilt where the writes live. The case set also spans
three eras of the same destination. In 2012, Google built it (F1 on Spanner)
because Ads' sharded MySQL could not be re-sharded again. From 2013 to 2019,
Alibaba built it (OceanBase) because nothing procurable combined
consistency, peak, and cost. By 2022 and 2023, the destination existed as a
product, and buyers arrived: DoorDash to CockroachDB, Pinterest to TiDB.
Three eras, three triggers, one destination.

**Cases in this chapter:** C27 (Google F1 on Spanner, first-party), C28
(Alibaba/Ant OceanBase, first-party/press), C25 (DoorDash on CockroachDB,
first-party via the company blog plus a vendor's conference write-up), C26
(Pinterest on TiDB, first-party), with transitions T03 (relational to
distributed SQL) and T04 (sharded relational to distributed SQL). Register
IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`.

---

## 1. Context: the semantics stayed, the topology changed

The chapters so far read families where leaving SQL was part of the deal:
wide-column stores traded transactions for write scale, key-value stores
traded query power for simplicity. The NewSQL family is the counter-move:
engines that keep ACID transactions and a SQL surface while distributing
writes across many nodes. The corpus records four teams at that counter-move
and, read together, they carry a specific lesson: in every one of these
migrations the data model survived unchanged, and the pain was always about
topology, who can write where, how shards are rebalanced, and how far apart
the copies sit.

The chapter's organizing read, from the book plan: three triggers, one
destination. Each era has its own trigger. In 2012 the trigger is the
resharding wall: a sharded fleet that works until the business grows past
its split. From 2013 to 2019 the trigger is the procurable-system gap: money
flows that need both Oracle-class consistency and commodity-class economics,
with no product that offered both. By 2022 and 2023 the trigger is the
generalist ceiling: single-writer clusters tripling their own latency at
peak, and a NoSQL platform era ending in deprecation. The destination is the
same every time: distributed SQL that keeps the transactional semantics.

## 2. The cases, each against its time

### 2012: Google builds the destination (C27, T04)

Google's AdWords system ran on sharded MySQL: "Google's entire
advertiser-facing interface was sharded across MySQL instances on specially
acquired hardware" ([C27: Google Cloud blog, reflecting on Spanner's
SIGOPS award](https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award),
FP). The trigger is written in one sentence of regret: Ads "had just
completed a very challenging resharding project" and "was resolute in its
desire to never do it again" (same source). So the Spanner team and Ads
partnered on F1, to move "the entire AdWords stack from MySQL to Spanner",
and by the time the Spanner paper was published in 2012, "we had just
completed the migration of AdWords frontend traffic to the new Spanner
backend" (same source). The register's summary holds the price: TrueTime's
global transactions, and a per-operation latency cost accepted as the price
of correctness (register C27).

The trigger here is pure topology. Nothing was wrong with MySQL's semantics;
what was wrong was that the shard count was fixed by human resharding
projects. Google's answer removed the semantic question entirely: keep the
SQL, distribute the writes.

### 2013 to 2019: Alibaba builds its own way out (C28)

Alipay reached the same destination from the other direction, and its
trigger was the procurable-system gap. The Database of Databases entry
records the moment: "In 2013, Alipay decided to stop using Oracle. Since the
alternative choice MySQL can't ensure strong consistency between active
server and standby server, OceanBase got its first opportunity" ([C28:
dbdb.io, OceanBase](https://dbdb.io/db/oceanbase/revisions/24), FP). The
founder's three stated principles are the requirement list in miniature:
"(1) distributed (2) low cost (3) high reliability" (same source). Scale
evidence is press-grade and audited the hard way, by nine consecutive Double
11 shopping festivals: the Alibaba Cloud case study records the 2017 peak at
256,000 payments per second (its text renders the figure as "0.25.6 million
transactions per second"), states that "All of Alipay's core business data"
is stored in OceanBase, and credits the engine with "lower costs and
higher scalability" against Oracle ([C28: Alibaba Cloud case studies](https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies),
FP). The survey's dossier adds an honest assessment: early OceanBase was
meaningfully slower per query than Oracle, a roughly 10% gap the team took
on deliberately (SURVEY dossier, graded as assessment). In 2019, "OceanBase
beat Oracle and won the first place in TPC-C test" (dbdb.io, same source),
and from 2017 the database "started to serve for external customers" (same
source).

The trigger is again procurement, not semantics: Oracle had the consistency,
MySQL's active-standby pairing could not hold it, and the economics of the
ledger demanded commodity servers. The build was the only available route to
the destination in 2013.

### 2022 and 2023: the buyers arrive (C25, C26, T03, T04)

DoorDash (C25, T03). The trigger is the single-writer ceiling. DoorDash's
new-verticals fulfillment backend outgrew its Aurora Postgres cluster: table
usage "went up to 500GB very quickly", past DoorDash's internal
recommendation that a "single PostgreSQL table size is under 500GB", and
inserts slowed
"because all the updates went through a single writer instance"; peak-hour
latency metrics doubled (C25: [DoorDash
engineering](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/),
FP). The company blog documents the migration in the playbook's own shape: a
database facade layer, dual writes, shadow reads with data comparisons, then
a feature-flag cutover, with all legacy retrieval flows behind the facade
first (same source, FP). The results are stated plainly: "we can handle 10X
load as we continue to scale our new verticals business" (same source). The
vendor's conference write-up and its press coverage add the part the
company blog softens: at weekly peaks, the old cluster would "crash and
burn", the cutover "took three attempts to succeed", an engineer watching
the migration video called it "literally a lossless cutover, as far as I
can tell", and afterward the team reported a decline in the volume of alerts
about incidents "despite running dozens more clusters than it did before"
([C25: Cockroach Labs](https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/),
and [C25: The New Stack](https://thenewstack.io/how-doordash-migrated-from-aurora-postgres-to-cockroachdb/);
the New Stack page's live fetch returns cookie boilerplate before its text
under this study's fetch cap, so those three quotes are verified in indexed
excerpts and noted here).

Pinterest (C26, T04). The trigger is the end of the NoSQL era: HBase
deprecation, and with it the search for a unified successor. Pinterest's
2022 evaluation is the transition matrix in one post: "we started the
evaluation by picking 10+ datastore technologies that were potential
candidates for our workloads", including both in-house options (Rockstore,
ShardDB) and every distributed-SQL contender on the market ([C26: Pinterest
engineering, TiDB adoption](https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10),
FP; the page blocks automated fetches, quotes below are verified in indexed
excerpts). There were fewer
options than the era's marketing suggested, and the post records the
assessments that excluded them: Spanner was "appealing in many
dimensions but is not open source", and "the migration cost from AWS to GCP
could be prohibitively high at our scale"; Vitess's "cross shard queries
may not be optimized" (all same source, indexed excerpts; the tails of
these one-line assessments are truncated in every
retrievable excerpt, and the register's record carries the fuller wording). The finalists "were TiDB, YugabyteDB, and DB-X" (same
source, indexed excerpt), and TiDB won. The freshness anchor matters: as of October
2026, Pinterest's ingestion framework still "move[s] database changes from
MySQL, TiDB and KVStore into analytical tables" at "petabyte-scale data and
thousands of pipelines" ([C26: InfoQ](https://infoq.com/news/2026/10/pinterest-event-time-partition/),
FP; live-verified). The zero-downtime HBase-to-TiDB migration mechanics the
register records are in the third Pinterest post of the series (register
C26, sources; same fetch limitation, register-recorded).

## 3. What happened (sourced outcomes, with evidence levels)

- **Google (C27, T04, FP).** Sharded MySQL on special hardware replaced by
  F1 on Spanner; the AdWords frontend traffic migration was complete by the
  2012 paper publication; TrueTime-backed global transactions accepted at a
  latency cost (register C27;
  [Google Cloud blog](https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award)).
- **Alibaba/Ant (C28, FP/PR).** Oracle exited at Alipay in 2013; OceanBase
  rebuilt the ledger role from scratch on the stated principles of
  distributed, low cost, high reliability; 256,000 payments per second at
  the 2017 Double 11 peak with all core business data on OceanBase; first
  place in TPC-C in 2019; external customers from 2017 (register C28;
  [dbdb.io](https://dbdb.io/db/oceanbase/revisions/24);
  [Alibaba Cloud](https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies)).
- **DoorDash (C25, T03, FP/VP).** Aurora Postgres replaced after the
  single-writer ceiling (500GB guidance, doubled peak latency); facade
  layer, dual write, shadow reads, feature-flag cutover; three cutover
  attempts, final lossless; 10X load handled; alert volume down despite
  dozens more clusters (register C25 and T03; DoorDash blog;
  Cockroach Labs; The New Stack, quotes via indexed excerpts).
- **Pinterest (C26, T04, FP).** 10+ candidates evaluated in 2022 against
  the HBase deprecation; TiDB selected over YugabyteDB and an in-house
  option; zero-downtime migration mechanics published; production use
  confirmed at petabyte scale as of October 2026 (register C26; Pinterest
  engineering via indexed excerpts; InfoQ live-verified).

## 4. The comparison: three triggers, one destination

| Test | Google F1 (2012, build) | Alibaba OceanBase (2013-19, build) | DoorDash CockroachDB (2022-23, buy) | Pinterest TiDB (2022-23, buy) |
|---|---|---|---|---|
| Trigger | the resharding wall: never again | nothing procurable: consistency + cost + peak | single-writer ceiling (500GB, doubled latency) | the NoSQL era's end (HBase deprecation) |
| What the semantics did | survived, moved "transparently" | survived, MySQL-compatible | survived (tabular SQL model kept) | survived (MySQL-compatible) |
| What actually changed | write topology, implicitly, via Spanner | everything below the SQL: storage, replication, consistency | write topology, shard rebalancing | write topology, unified storage layer |
| Transition record | T04's gold standard | T04's build-era twin | T03: three attempts, final lossless | T04's most honest re-evaluation |
| Evidence level | FP | FP/PR | FP/VP | FP |

**Cross-case comparison paragraph.** The register's T03/T04 pair and the
chapter's four cases agree on the shape of the thing: nobody left SQL
semantics behind on the way in, because nobody had to. Google's trigger was
a resharding project fresh enough to be loathed; Alibaba's was a market that
did not sell what its ledger needed; DoorDash's was a cluster whose writes
funneled through one instance; Pinterest's was a platform generation ending
in deprecation. Four different triggers, and in all four cases the engine
that answered kept transactions and the SQL surface. The differences are in
the build-versus-buy economics: in 2012 and from 2013 to 2019 the
destination had to be built, and both builds later converged into products
(the register's P15 convergence shape, read from the seller's side); by 2022
and 2023 DoorDash and Pinterest simply bought it, and their migration
stories are measured in cutover attempts and reversion tools, not years of
engineering. That is the destination's proof: the same move that once took
Google's partnership with the Spanner team became, within a decade, a
procurement decision with a feature flag.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Google (C27):** the founding case, and the one with the clearest
  scheduling honesty: the migration finished just as the paper in 2012 went
  out, which means the "first serious customer" framing is literal. The
  accepted latency price, per the register, is the honest cost of the
  destination, and AdWords is the proof the price is payable.
- **Alibaba/Ant (C28):** the build-era twin, validated the only way
  benchmarks cannot fake: nine consecutive Double 11 peaks, then a TPC-C
  first place taken from Oracle. The deliberate ~10% per-query gap, per the
  survey's assessment, is the honest entry fee of building early.
- **DoorDash (C25):** the buying era's operating picture, warts and all:
  three cutover attempts, a reversion tool built first, and alerts that
  quieted down only after the topology moved. The most transferable detail
  in the case is that the facade layer came first; the database choice
  followed the seam, not the reverse.
- **Pinterest (C26):** the era's conscience. Evaluating 10+ candidates and
  publishing the mechanics afterward is what the adoption learning loop
  looks like when it is not skipped. The zero-downtime series is the
  buying-era migration documentation standard for the whole corpus.
- **The family's rule:** when the semantics survive, budget the audit of
  the topology change instead: cutover attempts are where the risk lives,
  not the query language.

## 6. The pattern

- **P9 - The adoption learning loop.** adopt, break, build simple, evaluate
  honestly, buy the converged engine. Registered face: C26 (Pinterest: the
  NoSQL era broke, ShardDB and Rockstore were internal candidates, the
  honest evaluation landed on TiDB). Fails-when: the honest re-evaluation
  step is skipped, so one bad era becomes permanent infrastructure.
- **P11 - Latency as the price of correctness.** accept higher per-op
  latency rather than give up transactions. Registered faces: C27 (F1 on
  Spanner, TrueTime's latency cost) and C28 (OceanBase's deliberate
  per-query gap against Oracle). Fails-when: the workload cannot hide the
  latency, because the application has no asynchronous patterns to absorb
  it.
- **P15 - Build only what converges (the seller's view).** This chapter is
  the pattern read from the other side of the transaction: the two builds
  (Spanner, OceanBase) both converged into products, which is why the 2022
  and 2023 buyers could arrive at the destination with a feature flag
  instead of a multi-year program. The registered faces for the buyer's
  contrast (Espresso, Manhattan, Schemaless are the internal-only builds)
  are reads for Chapter 12. Fails-when: no open-source or platform plan, so
  the team bears full maintenance forever.

## Exercises

**●○○ (identify the pattern).** A team's Postgres cluster is hitting its
cloud vendor's recommended single-instance size, and peak-hour write
latency doubles whenever batch jobs run. A vendor offers a MySQL-compatible
distributed SQL product with a migration tool. Which two of this chapter's
triggers are absent from their diagnosis, and which pattern is the risk of
skipping the honest evaluation?

*Hint:* one trigger is about a wall they have not hit yet; the other is
about a detour they never took.

*Worked answer (assessment):* the missing triggers are the resharding wall
(nobody has been burned by a manual resharding project yet, so the move is
optional rather than forced) and the procurable-system gap (nothing in their diagnosis says no existing product fits: they have a product in hand). The risk class is P9's fails-when: skipping the honest re-evaluation step and buying on pressure rather than on the loop.

**●●○ (apply the framework).** A payments platform is leaving Oracle in
2026. Apply this chapter's three-era reading: what must the plan contain
that Alibaba's 2013-era plan could never have had, and what must it still
contain that Alibaba's did?

*Worked answer (assessment, judgment):* it escapes the build: commercial
distributed SQL now exists as a product category, so the build-versus-buy
economics differ (buy, and budget cutover attempts and reversion tooling, as
DoorDash's three attempts suggest). What it must still contain is everything
orthogonal to the era: P11's latency price (measure the per-query cost
against Oracle before committing to the number), P19-style verification
tooling (dual writes, shadow reads, diffing: the facade-first ordering from
DoorDash's case), and the ledger's own load-bearing semantics: correctness
requirements do not get cheaper because the engine is purchasable.

**●●● (steelman, then weigh).** Write the strongest case that Alibaba
should not have built OceanBase in 2013: the build consumed a dedicated
database team for three years, shipped a deliberate per-query performance
gap, and the commercial distributed-SQL market that materialized a decade
later now serves the exact need. Then weigh it against the register's
record, and end with the transferable rule.

*Steelman and verdict (assessment):* (i) the steelman: waiting would have
saved three years of team focus and ended at the same destination; the
"first commercial DBMS which supports distributed transactions" was also an
experimental bet, and the 10% gap was a tax paid on every query in the
meantime. (ii) the weigh: the register's record defeats the timeline
argument, because nothing procurable in 2013 combined strong consistency
across active servers with commodity economics, and Alipay's ledger could
not wait a decade; the nine Double 11s are the demand-side validation, and
the TPC-C record plus external customers from 2017 converted the build into
the product the steelman says they should have waited for. The transferable
rule: build only when no procurable engine meets a load-bearing requirement
and the wait itself is unaffordable; if the build converges into a product,
that is a bonus, not a plan (P15's fails-when is the negative space here).

**◆ (diagnostic: no hints).** One sentence each: which era's trigger, and
buy or build? (i) A 40-terabyte Postgres estate with a single writer and
300% peak growth. (ii) A ledger where active-standby consistency is
inadequate and the market sells nothing acceptable. (iii) An HBase estate
with rising maintenance cost and no experts left. (iv) A fleet approaching
its fourth manual resharding project.

*(Diagnostic answers, assessment: (i) the generalist ceiling, buy; the
single-writer topology is the trigger, the product exists. (ii) the
procurable-system gap, build only if the wait is unaffordable; measure
P11's price first. (iii) the NoSQL era's end, buy, but run the honest
evaluation, Pinterest's loop, before converging. (iv) the resharding wall:
the trigger is already firing; Google's case says the next resharding
project is the moment to change topology instead.)*

## Sources

All from the register; URLs copied exactly.

- C25 DoorDash engineering, How We Scaled New Verticals Fulfillment Backend with CockroachDB (DoorDash, FP): https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/
- C25 Cockroach Labs, Why DoorDash migrated from Aurora Postgres to CockroachDB (Cockroach Labs, VP): https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/
- C25 The New Stack, How DoorDash Migrated from Aurora Postgres to CockroachDB (The New Stack, PR): https://thenewstack.io/how-doordash-migrated-from-aurora-postgres-to-cockroachdb/
- C26 Pinterest engineering, TiDB Adoption at Pinterest (Pinterest, FP): https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10
- C26 Pinterest engineering, Sharding Pinterest: How we scaled our MySQL fleet (Pinterest, FP): https://medium.com/pinterest-engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f
- C26 Pinterest engineering, Online data migration from HBase to TiDB with zero downtime (Pinterest, FP): https://medium.com/pinterest-engineering/online-data-migration-from-hbase-to-tidb-with-zero-downtime-43f0fb474b84
- C26 InfoQ, Pinterest Uses Event Time Watermarks to Finalize Petabyte Scale Data Partitions (InfoQ, PR, 2026): https://infoq.com/news/2026/10/pinterest-event-time-partition/
- C27 Google Cloud blog, Reflecting on Spanner paper's SIGOPS Hall of Fame Award (Google, FP): https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award
- C27 The Morning Paper, Spanner: Google's Globally-Distributed Database (ACM/Alarm Clock, PR): https://blog.acolyer.org/2015/01/08/spanner-googles-globally-distributed-database/
- C27 Murat Demirbas, F1: Distributed SQL Database that Scales (blog, SR): http://muratbuffalo.blogspot.com/2023/07/f1-distributed-sql-database-that-scales.html
- C28 dbdb.io, OceanBase database entry, revision 24 (community wiki, PR): https://dbdb.io/db/oceanbase/revisions/24
- C28 Alibaba Cloud, ApsaraDB for OceanBase case studies (Alibaba Cloud, FP/PR): https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies
