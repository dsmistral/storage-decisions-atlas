# Chapter 14 - The NewSQL Absorptions

**Governing idea.** Distributed SQL's sales pitch is absorption: bring your
relational semantics, and the database will scale them horizontally without
asking you to leave SQL behind. The register's transition rows record what
happened when that pitch met real workloads. T03 (relational to distributed
SQL) carries DoorDash's Aurora Postgres exit to CockroachDB. T04 (sharded
relational to distributed SQL) carries two flows whose origins are in
different families: Google's sharded MySQL fleet into F1 on Spanner, and
Pinterest's HBase, a wide-column store, into TiDB. And the R6 round added T25,
the first recorded departure: Zomato's billing platform leaving TiDB for
DynamoDB in 2024. Chapter 8 read these four cases as eras of build versus
buy, and the survey's characterization of them is the chapter's spine:
"The same requirement set" (transactions plus horizontal scale) "produced two
builds and two purchases purely because the market matured between them",
which the survey calls the decision framework's "re-check the market before
building" rule, "demonstrated chronologically" (SURVEY comparison paragraph,
graded as the survey's characterization). This chapter reads the moves
themselves: what an absorption actually cost in each era, and what the one
recorded departure says about the destination family's terminal-status claim.

**Cases in this chapter:** C25 (DoorDash on CockroachDB, T03's mover), C26
(Pinterest on TiDB, T04's second face), C27 (Google F1 on Spanner, T04's
first face), C28 (Alibaba/Ant OceanBase), T25 (Zomato billing, the reverse
flow). Patterns: P9 (the adoption learning loop), P11 (latency as the price
of correctness), P15 (build only what converges), with the T25 direction
finding borrowed from the matrix chapter.

## 1. What an absorption is, and why the reading is chronological

An absorption, for this chapter's purposes, is a move into distributed SQL
whose pitch is semantic: unlike the exits of Chapter 12, the absorptions
claim to preserve what the origin gave the application. The claim comes in
gradations. Migrating Aurora Postgres tables to CockroachDB preserves the
SQL surface but changes the wire protocol and the operational model.
Migrating sharded MySQL to F1 preserves SQL and the application's mental
model while dissolving the shard map entirely. Migrating HBase to TiDB
preserves horizontal scale while restoring SQL, the thing wide-column never
promised. In each case the absorbed workload is supposed to come out the
other side believing little has changed.

The chronology is not decoration; it is the finding. The survey's comparison
paragraph runs the era census: Google "built" (2012: its parenthetical says
no product existed), Alibaba "built" (2013 through 2019: peak transaction
rates nothing procurable could meet), DoorDash and Pinterest "bought"
(2022 to 2023: converged products exist). Two builds and two purchases of
the same requirement set, spread across a decade of market maturity. The
survey's decision-framework rule number six is the rule the chronology
demonstrates: "Re-check the market before building. In 2012 Twitter had to
build Manhattan; in 2026 Pinterest could evaluate ten products and adopt one
(TiDB). Building a general-purpose store now requires justification that a
product (or a storage engine swap, à la MyRocks) cannot meet." The rule's
timestamp discipline matters as much as its content: era-specific answers,
read without their timestamps, look like laws for all time.

## 2. The moves, side by side

| Move | Window | Origin | Destination | Trigger | Build or buy | Evidence |
|---|---|---|---|---|---|---|
| T04, flow 1 (C27) | by 2012 | Sharded MySQL (AdWords) | F1 on Spanner (built) | Resharding as a recurring human project | Build | FP |
| T04, flow 2 (C26) | announced 2022 | HBase (wide-column), deprecated | TiDB | HBase end of life; unify on SQL | Buy | FP (dead-fetch source, indexed excerpts) |
| T03 (C25) | vendor write-up Nov 2023 | Aurora Postgres (relational, single primary) | CockroachDB | Single-writer ceiling; weekly-peak outages | Buy | FP (+ vendor VP) |
| T25 (Zomato billing) | 2024 (register date label) | TiDB (distributed SQL), self-managed | DynamoDB (managed KV) | TCO of overprovisioning; self-managed ops bill | Reverse | FP (blog plus AWS co-authored post) |

Four rows, two directions. The first three flow into distributed SQL from
three different origin families, which is the absorption claim made broad.
The fourth flows out of it, and no amount of era framing should obscure what
that does to the word "destination": the register's summary for T25 ends on
the correction, verbatim in two fragments, "unusual reverse direction" and
"NewSQL is not a terminal state", split where the register row's own em dash
sits (the dash itself stays out of this chapter per its conventions).

## 3. The moves, with figures

**T03: DoorDash absorbs its own relational bottleneck (C25).** The trigger
is the single-writer ceiling, stated in the team's own terms: table usage
"went up to 500GB very quickly", past the internal guidance that a "single
PostgreSQL table size is under 500GB", with slower inserts "because all the
updates went through a single writer instance" and peak-hour latency
metrics doubling (C25: DoorDash engineering, FP; live-verified). The
migration mechanics are the playbook in miniature, and Chapter 13's
Property 4 preview applies even though this move crosses engines: a
database facade layer (RFDS) first, then dual writes, then shadow reads
with data comparisons, then a feature-flag cutover (same source, FP). The
vendor's write-up (Cockroach Labs, VP; live-verified this session) adds two
spans the company blog does not carry: the weekly-peak failure, in an
engineer's recollection that "our main database would crash and burn", and
the reversibility engineering, in the tool built "ensuring easy and lossless
reversion in the event something went wrong" before the automated,
"losslessly" migrating successor replaced it. The New Stack's coverage
carries the register-summary details (three cutover attempts, the alert
volume declining "despite running dozens more clusters"); its live fetch
returns cookie boilerplate under this study's fetch cap, so those details
stand on the register's T03 row and indexed excerpts, the same handling as
Chapter 8.

**T04, flow 1: Google absorbs sharded MySQL (C27).** The origin's pain was
not the engine but the resharding cycle: "Google's entire advertiser-facing
interface was sharded across MySQL instances on specially acquired hardware",
and Ads, having "just completed a very challenging resharding project", "was
resolute in its desire to never do it again" (C27: Google Cloud blog, FP;
live-verified). The absorption's character is in one verb: the F1 project
existed "to transparently move the entire AdWords stack from MySQL to
Spanner". Transparency is the absorption promise in its strongest form: the
application was not supposed to notice the topology change underneath it.
"By 2012, when we published the Spanner paper, we had just completed the
migration of AdWords frontend traffic to the new Spanner backend" (same
source). The price is register-recorded: TrueTime's global transactions and
the per-operation latency accepted for them (register C27; the survey's
assessment is that async application patterns hid the latency, and Chapter
8 carries the sourcing note).

**T04, flow 2: Pinterest absorbs its wide-column estate (C26).** The
trigger is deprecation: HBase end of life, and with it the question of what
absorbs the workloads it ran. The team's 2022 evaluation is the re-check
rule executed in public: "we started the evaluation by picking 10+ datastore
technologies that were potential candidates for our workloads" (C26:
Pinterest engineering, FP; the page blocks automated fetches, quotes
verified in indexed excerpts per the register's handling). The evaluation's
exclusions are as instructive as its winner. Spanner was "appealing in many
dimensions but is not open source", and "the migration cost from AWS to GCP
could be prohibitively high at our scale"; Vitess's "cross shard queries may
not be optimized" (same source, indexed excerpts; Chapter 8 carries the
truncation caution). TiDB won among finalists including TiDB, YugabyteDB,
and DB-X. The current-state anchor is freshness-grade: as of October 2026,
Pinterest's ingestion framework still moves "database changes from MySQL,
TiDB and KVStore into analytical tables" at "petabyte-scale data and
thousands of pipelines" (C26: InfoQ, FP; live-verified). The absorption has
held for roughly four years without a public regret line; the register's W2
watch is explicitly still open on the long-term verdict.

**T25: Zomato leaves the destination family (the register's R6 reverse
flow).** The full arc is in one first-party post, co-published by Zomato and
AWS (the AWS page records "This post is co-authored with Neha Gupta &
Kanica Mandhania from Zomato"; the Zomato blog's own credit names the same
two authors, FP). Zomato chose TiDB in the relational-expertise mold:
"The decision to use TiDB was based on the expertise of Zomato's engineering
team in the relational model and MySQL, as well as its ability to
horizontally scale and meet scaling needs" (same source, FP). The exit's
drivers are the ones Chapter 21 will call the ops-burden and cost drivers,
and they are unusually explicit: the cluster grew from 5 nodes to 25, "in
both the primary and replica clusters", and on peak days "the infrastructure
was manually scaled up and kept overprovisioned to prevent performance
issues"; TiDB was self-managed, taken up "when their cloud solutions were
not available", so "our team took on the majority of heavy lifting to ensure
its reliable operation" (same source, FP; live-verified). The exit's
mechanics are the playbook run in reverse: a phased approach with "dual
write, replication, data synchronization, and failover mechanisms" for zero
downtime (same source). Leaving SQL had a semantic bill, and the post pays
it openly: the relational schema of separate tables with foreign-key joins
was redesigned into "a unified DynamoDB table using the adjacency list
design pattern", with an inverted index for the many-to-many side (same
source, FP). The outcome figures: an average decrease of 90% in
microservice response time, throughput up to "8,000 RPM", four times more
traffic handled at the same scale, and the money line the register's T25 row
summarizes: "In the new solution with DynamoDB, we pay only for what we
use, unlike the previous approach with TiDB, which involved
over-provisioning the database for sudden traffic peaks. As a result, our
billing system's monthly expenses saw a notable 50% reduction due to these
changes." (The page renders Zomato's scale in Indian digit grouping, "more
than 2,00,000 restaurants"; the register's summary carries the cost figure
as "~50% monthly cost reduction".)

## 4. The comparison: what the chronology demonstrates, and what the departure corrects

Read the four rows as one experiment repeated across a decade. The constant
is the requirement set: transactions plus horizontal scale. What varies is
the market, and the moves diverge exactly where the market does.

In 2012 through 2019 there was nothing to buy, at any acceptable
conformance. Google built F1 and paid P11's price knowingly; Alipay's ledger
left Oracle for a build whose early per-query gap the survey's dossier 11
grades at roughly 10% (assessment, carried from Chapter 8 with its sourcing
note), and whose 2019 TPC-C result is the register's audited-the-hard-way
coda. The builds were forced, and their trigger lists do not mention any
product, because there was none to mention.

By 2022 the market had converged products, and the buyers put them through
adoption-grade scrutiny rather than gratitude. DoorDash's move went through
a facade, dual writes, shadow comparisons, and three cutover attempts before
it held; Pinterest evaluated more than ten candidates and documented why
each rejected candidate lost. Neither buyer treated absorption as free, and
both kept a fallback: DoorDash's tooling was built for "easy and lossless
reversion" first. Verdict (assessment): this is the quiet maturation of the
corpus itself, the difference between the pioneering exits of Chapter 12
(improvise the semantics) and the buyer era (the checklist exists, in
public, before the move).

Then T25 corrects the frame. If distributed SQL were a terminal state, the
buyer era would end the story: build nothing, buy the absorber. Zomato's
2024 departure records the opposite arithmetic, and the corpus's direction
finding is explicit about it (ANALYSIS, Ch. 11 matrix direction findings
row), quoted in two fragments split where its em dash sits: "T25 (Zomato
TiDB→DynamoDB) and T27 (Bhoos DynamoDB→PG) show neither distributed SQL nor
managed KV is a terminal state" and "both reverse flows now first-party".
What made the
departure rational was not a failure of distributed SQL's semantics; Zomato
gave TiDB three years of production billing. It was the mismatch between
the store's operating model and the workload's economics: peak-shaped
traffic, overprovisioned capacity, self-managed infrastructure. The
departure price was semantic (joins denormalized into an adjacency list),
and Zomato's record shows it was paid knowingly, as a purchase decision, not
as an accident of drift.

## 5. The patterns

**P9, the adoption learning loop, on C26.** The registered face is
precisely this chapter's second flow: ANALYSIS's P9-row face cell reads
"C26: NoSQL 'broke catastrophically' → ShardDB → 2022 evaluation of 10+ →
TiDB", its inner quote marks demoted to single glyphs by this study's
conventional note (the arrows are the ANALYSIS text). The 2022 evaluation is
the re-check rule in loop form: a team that had been through the NoSQL era
and an intermediate in-house stop ran a fresh, documented, multi-candidate
evaluation rather than defaulting to either its own history or the era's
marketing. P9's fails-when ("The honest re-evaluation step is skipped")
is the counter-face the ANALYSIS names: internal builds that never re-ran
the evaluation.

**P11, latency as the price of correctness, on the build era.** ANALYSIS's
P11 row reads: "C27: F1 accepted higher per-op latency, hidden by
async app patterns; C28: ~10% per-query gap accepted" (the ANALYSIS
pattern table's P11 cells, quoted; the sourcing split for the ~10% figure
is Chapter 8's, pinned to SURVEY dossier 11). Both builds paid the same
bill, correctness purchased with latency, and paid it deliberately. The
fails-when ("The workload can't hide the latency") is precisely what T25's
traffic profile shows on the cost side: Zomato's billing could not hide a
latency-above-capacity event on peak days, and its answer was architectural
pay-per-use rather than transactional renegotiation.

**P15, build only what converges, read from the buyer era.** The P15
lens explains the title of this chapter's story: across the book, the
builds that paid off became platforms or open source, and the ones that
did not stayed internal. Of this chapter's two builds, OceanBase's record
is the register-sourced face of convergence: "From 2017, OceanBase started
to serve for external customers" (C28: dbdb.io, FP; live-verified), plus
the 2019 TPC-C result in the same source. Spanner's product path is
handled as the study's context per Chapter 8's review discipline, not
asserted from the register rows. The buyer side completes the reading
(assessment): buying a converged engine is the cheap half of P15, and T25
shows even that purchase stays open to re-evaluation, which is P9's loop
wearing P15's outcome label.

## 6. Spec state and what the chapter hands forward

**Spec state.** BOOK-PLAN's row 14 inputs cell reads "T03, T04; C25, C26,
C27, C28"; the chapter map's line reads "T03, T04, C27, C28 (+ T25 Zomato
reverse flow, R6)". Two divergences, both recorded per the Ch. 6/11/13
convention. First, the plan's cell predates the R6 round: it omits T25,
which the chapter map carries and the chapter is written against. Second,
the map's line names only C27 and C28 as case inputs, but T03's mover and
T04's second face are C25 and C26 in the register rows themselves, so they
enter as the T-rows' movers, their full era-case reading owned by Chapter
8 (this chapter cross-references rather than re-reads: Ch. 8 owns what the
market offered each buyer, this chapter owns the mechanics, price, and
reversibility of the moves). The plan's single idea ("from relational and
wide-column") is carried by T04's two faces, whose origins are Google's
sharded MySQL and Pinterest's HBase.

**Forward pointers.** Chapter 15 holds the contrast class: the engine-swap
twins keep the protocol and swap the internals (P12), the lowest-blast-radius
move, against this chapter's protocol-changing absorptions. Chapter 16 owns
the returns-and-round-trips cluster, and T25 reappears there among the R6
outflow flows (T23 through T27); the division of labor is that this chapter
reads T25's mechanics and its NewSQL-terminal-state finding, while Chapter
16 reads the outflow cluster as returns. Chapter 21 owns the driver census,
and T25's record is a clean entry there: the drivers its post names are the
ops burden and cost, the same two Chapter 9's T26 preview set up. W2 stays
open on Pinterest's long-term TiDB verdict; its TiDB-exit side is documented
only by T25's Zomato flow.

**Part-level verdict (assessment).** The absorptions are the corpus's proof
that the SQL family's center of gravity moved, in one decade, from
something teams built when they outgrew sharding to something teams buy
with a documented evaluation. The chronology also warns the other way: T25
recorded, three years after the buyer era opened, that the absorber's
operating model is itself a decision with a price. The central
nomenclature discipline this chapter leaves the reader: distributed SQL is
a routing decision, not a destination.

## Exercises

●○○ (read the direction column). A billing platform that spent three
years on a distributed SQL engine moves to a managed key-value store,
cutting monthly database spend roughly in half by replacing statically
overprovisioned capacity with usage-based billing. Name the register's
transition row for it, the direction it runs relative to the matrix's
inbound distributed SQL rows, and the ANALYSIS row that states the
finding.

●●○ (apply the re-check rule). Your team, in 2026, needs transactions plus
horizontal scale for a workload you estimate at two terabytes and a peak
of six thousand writes per second. Write the three-item checklist the
survey's rule number six requires before any build decision, carry the
build era's price tag from C27 and C28 into the comparison, and state
which two register rows you would read first and why.

●●● (steelman the reverse flow). An architect dismisses T25 as "Zomato
chose TiDB badly and then bought marketing". Using the Zomato post's own
record (why TiDB was chosen, the 5-to-25-node scaling, the self-managed
ops description, the adjacency-list redesign), write the strongest case
that the departure was a rational re-evaluation under P9's loop, then
identify which single documented fact most weakens your steelman.

◆ (diagnostic, no hints). Two teams leave their databases the same year.
Team A migrates sharded MySQL tables into a distributed SQL database,
keeping application SQL unchanged behind a facade. Team B migrates a
distributed SQL billing schema into DynamoDB, redesigning joins into an
adjacency list. Name each move's register row, the pattern registered
against the era each belongs to, and the one word in Team B's architecture
bill that would not have appeared in Team A's.

## Sources

The chapter's factual base is the register's rows C25, C26, C27, C28, T03,
T04, T25, plus P9/P11/P15 of the ANALYSIS pattern table and the register's
P-index in `PROJECT.md`. The register and ANALYSIS quotes were byte-checked
against the corpus mirrors; ten source URLs were re-fetched live this
session (eight fully, Cockroach Labs including the crash-and-burn and
lossless spans; the Pinterest Medium post and The New Stack carry the
registered dead-fetch and cookie-wall limitations, noted where used):

- https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/ (C25, T03)
- https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/ (C25, vendor write-up)
- https://thenewstack.io/how-doordash-migrated-from-aurora-postgres-to-cockroachdb/ (C25, T03 register-summary details; dead-fetch noted)
- https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award (C27, T04)
- https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10 (C26, T04; dead-fetch noted)
- https://infoq.com/news/2026/10/pinterest-event-time-partition/ (C26 current state, R5)
- https://dbdb.io/db/oceanbase/revisions/24 (C28)
- https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies (C28)
- https://blog.zomato.com/switching-from-tidb-to-dynamodb (T25)
- https://aws.amazon.com/blogs/database/unlocking-performance-scalability-and-cost-efficiency-of-zomatos-billing-platform-by-switching-from-tidb-to-dynamodb/ (T25, co-authored)
