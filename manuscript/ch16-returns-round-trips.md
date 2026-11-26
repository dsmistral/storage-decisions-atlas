# Chapter 16: Returns and round trips

## 1. Context: the moves nobody champions

Part II has so far read transitions in their celebrated direction: the pioneering
exits (Chapter 12), the sharding programs done right (Chapter 13), the absorptions
into distributed SQL (Chapter 14), the engine swaps (Chapter 15). The market does
not celebrate the moves in the other direction, and the corpus records that
asymmetry as a finding in its own right. Ten register transition rows face away
from the dominant flows, and they are the quietest papers in the file. The
BOOK-PLAN's row 16 names the chapter's single idea as: "The quiet
document→relational return (small/mid only), the vector unwind both ways, the
two-week TimescaleDB round trip, and Uber's RDBMS→RDBMS".

Why quiet? The survey's deferral D3 records it: the corpus's first-party
document→relational returns are "First-party returns exist only from small/mid
teams (slice, Infisical); big-tech reversal undocumented", with the deferral's
reservation that this is "likely survivor gap, not absence"
([corpus/STATE.md](#sources), deferral table). A giant returning to a family it
exited would be a story with the exit's numbers reversed, and no such paper is in
the register: the return stories here are small-team and mid-scale first-party
accounts, plus one source-reliant round trip published by a one-person product.
Chapter 2 planted this seed at the document family's edge: its exit-flow preview
carried the reading that "the family's exits are access-shape
exits, not fashion exits", graded as assessment there, and this chapter pays
that preview off.

The chapter reads ten transition rows: T10, T11, T13, T19, T20, T23, T24, T25,
T26, T27. Four moves are returns in the strict sense, a move back into the
relational family the mover (or a mover like it) had left: T19's three movers
and T27. T13 is the round trip: out of MongoDB into a purpose-built engine and
back into MongoDB two weeks later. T20 documents the
vector unwind in both directions, so it is a round trip class rather than a row
with one arrow; T11 and T23 stay relational and carry this chapter's hardest
lesson about a published story; and the outflow cluster the R6 verification pass
added (T24, T25, T26) supplies the managed-cloud counterweight. T25's mechanics
are Chapter 14's by explicit division of labor: it enters here only as the
pointer that the NewSQL exit exists, since the register's own T25 row scores the
direction as "unusual" and adds that "NewSQL is not a terminal state".

## 2. The moves

| Move | Mover | Origin → Destination | The move in one line | Channel |
|---|---|---|---|---|
| T19 | slice (C44) | MongoDB → PostgreSQL | ACID for money flows, a fintech's quiet return | FP |
| T19 | Infisical (C45) | MongoDB → PostgreSQL | self-hosting demand outran the document model | FP |
| T19 | Voucherify (C65) | MongoDB → PostgreSQL | side-by-side sync, triggers + AWS DMS, zero user-visible downtime | FP |
| T10 | Mixmax | MongoDB → DynamoDB | the key-value return of an email-tracking workload | FP |
| T11 | Uber | PostgreSQL → MySQL | the move whose outcome was fine and whose justification aged badly | FP/SR |
| T13 | NexusTrade | MongoDB → TimescaleDB → MongoDB | the two-week round trip, reversed intact | SR |
| T23 | Yelp | PostgreSQL → MySQL | in-place standardization: reads first, Postgres writes off months later | FP |
| T24 | Deliveroo | RDS PostgreSQL → DynamoDB | sharding considered and rejected in favor of a managed KV | VP |
| T25 | Zomato | TiDB → DynamoDB | read whole in Chapter 14; the outflow pointer only | FP |
| T26 | Freecharge | Aerospike → DynamoDB | the managed-cloud answer to PhonePe's on-prem question | VP |
| T27 | Bhoos Games | DynamoDB → PostgreSQL | lazy per-user migration on next connect | FP |
| T20 | Confident AI (C34) | Pinecone → pgvector | the unwind outbound; read whole in Chapter 10 | FP |
| T20 | Pinecone (C35) | pgvector → Pinecone | the counter-flow, documented by the incumbent; read whole in Chapter 10 | VP |

Ten rows, thirteen lines: T19 carries three movers and T20 two directions. The
channel column is the chapter's caution surface: the two vendor-channel rows
(T24, T26) are guest posts and case studies published by the destination's
vendor, and their figures are read under that flag below.

## 3. The mechanics

### The quiet document→relational returns (T19)

Three teams left MongoDB for PostgreSQL, for different published reasons, all at
scales where the document model's distribution story was not the point.

**slice** is a fintech, and its move is the register's C44 row: "ACID for money
flows". The slice post is a Medium publication and its page blocks automated
fetches (142-character stub this session, as registered), so its spine is
corroborated via search snippets and flagged as such: the post's own title reads
"Freaking hard Migration from MongoDB to PostgreSQL: Leveraging Data Platform for
Scalable Success", and its indexed text carries "PostgreSQL's ACID compliance
ensured transactional reliability", followed at the source's em dash by "critical
for fintech applications", then "its open-source model eliminated licensing
costs" and "SQL familiarity also made database operations more streamlined for
our engineering team". A fintech moving money-adjacent flows to a transactional
relational engine is the return the quiet class exists to hold.

**Infisical** is the secrets manager, and its return runs the other way inside
the class: a managed product grew a self-hosted audience, and the document model
did not follow. The page opens at "north of 50 million secrets daily", and names
the diagnosis: the team "often ran into constraints around the capabilities and
usability of MongoDB like the lack of support for transactions". Its
transactions receipt is the next sentence at source: "With MongoDB, setting up
transactions was not trivial because it required running MongoDB in cluster mode
with various configuration overhead". The trigger was a product decision:
"With demand growing for self-hosting Infisical, we found ourselves shipping many
features catered to reducing the learning curve needed to self-host Infisical
and, as part of that, we ended up leaving MongoDB in favor of PostgreSQL". The
result line is the cluster's cleanest migration receipt: "Fortunately, the
migration execution turned out smooth with zero data loss and only a few
non-essential incidents of feature malfunction; we ironed out these bugs out in
the following 36 hours with minimal impact to customers". The register's C45
summary line also calls the destination "One unified SQL+JSONB platform"; the
live page's text carries the migration story above and does not itself spell the
JSONB claim, so the chapter keeps that phrase at register-summary level and does
not attribute it to the page.

**Voucherify** is the promotions engine, and its return is the most engineered of
the three. The banner reason was cost inside a two-database estate: the post
names Compose, "a SaaS platform serving MongoDB", whose price "was very expensive
in comparison to alternatives" (the page's bold on the phrase is source markup),
plus the cascade tax of running two database types, "cascade effect of doubled
codebase, paradigms and concepts". The estate itself: "around five years of data
collected" (source bold again), "Around a terabyte of constantly changing data",
and a schedule of three months plus three: "we spent three months rewriting and
testing new code and next three months migrating all the data". The method is
what the register's T19 row summarizes as "zero downtime via side-by-side sync,
triggers + DMS", and the post's thesis line says it plainly: "By running MongoDB
and Postgres side by side, syncing data gradually, and validating changes as they
happen, you can switch databases without users ever noticing". Two
implementation receipts carry the pattern: the transformation layer ran on
"triggers in Postgres", which "can have variables supplied", and the copy
machinery was AWS DMS, whose "overall cost of using DMS in our case was
marginable". The well-known ~30% figure is the cross-post's claim, not a sentence
in the blog body: the author's dev.to version is titled "How we moved from
MongoDB to Postgres without downtime and cut our costs by 30%"
(search-snippet-verified; the register's C65 row carries both URLs and the ~30%
cut sits in the cross-post's title, marked self-reported at the register).
Dating note: the blog is by Adrian Wilczek, dated November 1, 2024 at the page;
the cross-post's first comment is dated May 6, 2021, so the 30% claim predates
the blog post's publication by about three years.

D3's requirement holds across all three: a fintech, a small open-source company,
a promotions vendor. No giant has published a first-party document→relational
return; the register calls that the likely survivor gap, not an absence, and a
reader who wants to falsify D3 has a concrete hunt to run.

### The small-scale key-value return (T10)

Mixmax's move from MongoDB into DynamoDB is the register's T10 row: "Clean,
well-documented at small scale". Chapter 2 read it as the document family's
exit-flow preview and had to stop at the register's summary because the page
fetched thin at authoring time; this session the page fetches clean, so the
operating details can be read from the paper directly. The post is dated
December 11, 2023, signed Brad Vogel, and frames the project as "migrating 1.5
billion documents from MongoDB to DynamoDB". The load: a tracking system at
"eventually reaching a staggering 1.5 billion records (even after culling old
records)" where queries "became intolerably slow, taking more than 200ms", with
"Our indexes outgrew RAM capacity" and "the financial cost of maintaining such a
large MongoDB instance was becoming unsustainable". Note the shape of the pain:
none of it is a data-model complaint. The cutover is a week of dual writes and a
backfill: "A week-long migration transferred all 1.5 billion documents to
DynamoDB, with the database easily handling 10k insertions per second", followed
by the read switch, "After confirming the data was in sync, we updated our UI to
read from DynamoDB and eventually decommissioned the MongoDB setup", and a
performance surprise the post splits across its own em dash: "reads in Dynamo not
only didn't get slower", "it got faster". The register's verdict supports both
halves, and the same D3 honesty that keeps the returns small keeps this row's
finding small: at a scale where any of several answers would have worked, the team
picked the one whose ops story fit it (assessment).

### The RDBMS→RDBMS pair: the outcome and the mirror (T11, F11, T23)

Two register rows move between relational engines, and they pair as this
chapter's hardest read: the move whose published story is the problem (T11, Uber,
with F11) and the move whose story is the repair (T23, Yelp).

**Uber's Postgres-to-MySQL move** (July 26, 2016, by Evan Klitzke) was fine as
engineering: the register's T11 row says "Outcome fine", and the paper's own
frame is that "in many of the cases where we previously used Postgres, we now use
Schemaless, a novel database sharding layer built on top of MySQL". Its five
published limitation bullets are "Inefficient architecture for writes",
"Inefficient data replication", "Issues with table corruption", "Poor replica
MVCC support", and "Difficulty upgrading to newer releases". The trigger story is
told at source in two sentences worth reading together: "During a routine master
database promotion to increase database capacity, we ran into a Postgres 9.2
bug", followed by the post's own correction, "The bug we ran into only affected
certain releases of Postgres 9.2 and has been fixed for a long time now". Into
that mix the post drops the sentence that puts the piece on the failure
postmortem list's periphery. The scope note says the analysis is "primarily based
on our experience with the somewhat old Postgres 9.2 release series", and then
the next sentence evicts the guardrail: "To our knowledge, the internal
architecture that we discuss in this article has not changed significantly in
newer Postgres releases". That is the generalization that aged badly.

The register's F11 row is the adversarial record of what happened next:
"Engine-mechanics claims (PG 9.2 era) later largely erased by PG10+; conflated
engine limits with ops inexperience; industry took wrong lesson for years". A
critic's post by Oren Eini (Ayende) returns the failure to operations rather than
engine mechanics; that site blocks automated fetches (the 142-character block
stub this session, as registered), so the chapter carries the register's summary
of the critics' position and ANALYSIS's sharper cut, "F11: skill justified the
move; blog blamed the engine", without quoting the post directly.

**Yelp's in-place standardization** (T23) is the same direction, published eight
years later, with the story told right: the register's T23 row calls it "the
honest mirror of T11/F11". The Yelp Reservations service ran on "PostgreSQL
which is not used anywhere else at Yelp, which meant that only a small rotation
of long-tenured employees knew Postgres well enough to do outage response". The
cost of that singularity: "This caused issues in maintenance, visibility, and
outage response times". The decision is an organizational one, stated without
engine-anxiety theater: "So, we switched out this DB in-place with a
Yelp-standard MySQL DB". The playbook is Chapter 11's quoting target and worth
its full span here: "During the release, we first kept reads on Postgres, to keep
identical behavior as the status quo while also writing to MySQL. This let us
cross-check the databases and fix inconsistencies and bugs at our leisure without
affecting customers. We then gradually switched requests to read from MySQL, then
switched the write logic to write to MySQL first, and finally (several months
later) turned off Postgres writes entirely and cleaned up much of the code we had
written". The engine differences are handled as feature-translation work, not as
a verdict on the old engine: "Postgres has a lot of functionality that isn't
supported in MySQL", with the triggers case as the receipt, "Triggers are
supported by MySQL in general, but are not supported by our MySQL
infrastructure". ANALYSIS's P8 row reads the pairing for what it is: "C62 Yelp:
the honest mirror of F11", where Yelp "moved a product DB to org-standard MySQL
because the team lacked deep Postgres expertise and said so plainly", while
"Uber's same-direction move aged badly because it conflated engine mechanics with
ops inexperience".

### The outflow cluster and the managed-cloud counterweight (T24, T26, T27; T25 by pointer)

The R6 pass added the outflow flows T23 through T27, and Chapter 11 read them as
matrix rows; this chapter reads the three not yet read in depth, T24, T26, T27,
as one cluster, because they share a question: who runs the small
operationally-critical store, you or your vendor? Chapter 11 read T26 and T27 as
"the same question answered twice, once in each direction".

**Deliveroo's dispatcher** (T24, vendor channel) considered the self-run answer
and wrote down why it lost: "We considered sharding the existing RDS for
PostgreSQL database to reduce the number of records", then "However, we decided
against this approach for the following reasons", and the reasons are operational
before they are architectural: "Because new major database engine versions
sometimes introduce breaking changes, Amazon RDS doesn't perform major engine
version upgrades automatically". The chosen line: "Therefore, we decided to
migrate our data store to DynamoDB, which is a fully managed, serverless, NoSQL
key-value data store", with "consistent single-digit millisecond performance for
reads and writes regardless of table size" as the stated sell. The subtle part of
the move is semantic, not mechanical: the dispatcher's after-commit Kafka
semantics had to be rebuilt, and the paper says how: "To mimic this after-commit
behavior, we decided to set up a DynamoDB stream and use an AWS Lambda trigger to
invoke a Lambda function that publishes messages to Kafka", with ordering handled
by "a version attribute in the DynamoDB items". Flag: this is a guest post
published by AWS (the register's VP channel), so the destination's virtues are
the vendor's account of them; the sharding-rejection reasons are the post's own
and stand regardless.

**Freecharge's identity service** (T26, vendor channel) is the direct contrast
with Chapter 9's PhonePe section, where the register's T26 row says the two are
"the managed-cloud answer to the same workload class PhonePe (C57) runs on-prem",
and closes that summary with the register's own phrase: "one decision, two
answers". The post's summary line: "DynamoDB helped effectively manage traffic
spikes while lowering total cost of ownership (TCO)" at an estate that "regularly
encounters traffic spikes resulting from marketing campaigns, promotions offers,
and live events". Its headline number is vendor-reported and read here under the
P16 caution: "DynamoDB's pay-as-you-go pricing effected a 60% lower cost of
ownership for this workload". The similarity to PhonePe's record (C57, Aerospike
on-prem, hundreds of clusters, three data centers) is the point of the row; the
difference is who owns correctness, and Chapter 9's correctness-owner split holds
on both sides of this pair.

**Bhoos Games** (T27, first-party) closes the loop back into relational: the
register's row says "20M+ items" moved "Lazy per-user migration on next connect",
and the post reads: "migrating the production database consisting of more than 20
million user items from DynamoDB to PostgreSQL". Why lazy rather than bulk:
"Since there were more than 20 million user items, it was unfeasible to carry out
offline migration", and the mechanism is the post's own named technique: "When
the user connected to our server the next time, we carried out what we call 'The
Lazy Migration Technique', migrating their data automatically to Postgres DB on
first connection". The motive list is a small-team inventory: no indexes beyond
the id ("executing any non-id query was both time-consuming and expensive"), no
analytics, and an encoded schema where "$a$1 is diamond", "$a$2 is coin", "$a$3
is XP". T27 is the return that F09's round trip only gestured at: a deliberate,
per-user, hold-the-old-path-steady move into PostgreSQL, published by a small
team, done for query freedom and cost legibility rather than for any engine's
fashion.

### The vector round trip, both ways (T20)

The vector family's unwind is this chapter's round-trip class: the index moved
out of a dedicated engine into an embedded one, and the register documents
traffic in both directions. Chapter 10 read both papers whole, so this section
reads only the class finding. The outbound face is Confident AI's Pinecone to
pgvector move, whose decision line reads "after weeks of experimentation, we made
the decision to replace it entirely with pgvector". The counter-flow faces it
from the engine side: Pinecone's own post opens the pump on users arriving the
other way, "Over the past several months, people have come to Pinecone after
trying and failing with pgvector and other bolt-on vector databases". The
register's T20 row holds the verdict and refuses the direction: "Workload-
dependent, not directional", with "both directions documented" as the evidence
statement. That is why this chapter reads T20 as the round-trip class: the
honest form of the finding is not "engines win" or "libraries win" but that the
vector index's home is a placement decision, revisit-able in either direction,
which is exactly what a round trip with surviving paths looks like when nobody's
plan required dismantling one.

## 4. Returns versus wrong choices: what the old path buys you

The chapter's two risk collisions are a round trip that worked and a
justification that did not.

**The round trip that worked.** NexusTrade's Mongo to TimescaleDB to Mongo move
(T13, F09) is the register's only source-reliant return, and its author's title
is the receipt: "I went through hell and back migrating to TimescaleDB. I didn't
last two weeks". The premise: facing performance problems, the author "assumed it
was because it wasn't 'purpose-built'" and undertook what the post calls "a crazy
migration to TimescaleDB". The reversal: "Two weeks into the trial, I returned to
MongoDB and avoided the planned $590 monthly separate Timescale instance". The
verdict line at the trail's end: "My old caching solution would've been a lot
cheaper than this, and it worked!". Chapter 7 read the time-series mechanics of
this failure (the continuous aggregates and compression that fought the
workload); Chapter 20 owns its failure-class reading. What this chapter adds is
the enablement: the move was reversible because the old path was intact, and the
survival was real because of it. ANALYSIS's P20 row says exactly this on its
face: "C39, C41, T13/F09 (NexusTrade survived because old path intact)".

**The justification that did not.** Uber's T11 is the pairing row: outcome fine,
story wrong, and the story's error strengthened a wrong industry lesson for
years per the register's F11 row. The mechanics story was half right and wholly
overstated; the ops story (a small rotation of people, upgrades avoided for
years, a bug at a routine promotion) pointed at team size and process, and the
paper's own two sentences carry both signals without reconciling them. Yelp's
T23 is what the corrected version looks like: the team factor said out loud, the
engine differences treated as translation work, the cutover read-first and
months-long.

**Verdict (assessment).** The class's teaching is not that returning is safe or
that staying is virtuous; it is that both stories get truer when the old path
survives the cutover and the paper's claims are scoped to the engine's actual
diff. Every row in this chapter's table that ends well did both. The one row
read for its failure of communication did neither.

## 5. Patterns: team size, reversibility, measured claims

**P8, team size is an architectural input.** The P-index's rule reads "Team size
is an architectural input: the engine your small team can operate wins", faces
"C05 (Instagram, 3 engineers), C02, C03", and fails when "Used to excuse
avoidable learning: skill justified a move whose published mechanics were
overstated (F11, Uber)". This chapter supplies the pair's negative and positive
faces at the pattern's exact grain. Uber's engine-mechanics story is what P8's
fails-when carves off from the honest version of the same decision; Yelp's row
is the honest version, and ANALYSIS's P8 cell marks it "the honest mirror of F11"
with a line the chapter now quotes whole enough to steer by: Yelp "moved a
product DB to org-standard MySQL because the team lacked deep Postgres expertise
and said so plainly". The return movers score on the same axis without needing
the word: Infisical's transactions receipt is a small-team story in operational
terms, and Voucherify's two-times-three-months schedule is capacity accounting.

**P20, reversibility is the survival requirement.** The P-index's rule: "if your
plan cannot return to the old system on day 60, it is not a plan", fails when
"The old path is dismantled at cutover", the clause after the P-index's em dash
reading "the first post-migration regression becomes unrecoverable". The chapter
round trip is the face: NexusTrade survived, per ANALYSIS's P20 row, "because old
path intact"; Yelp's months-long read overlap is the same discipline at
migration scale; Bhoos's lazy per-user move keeps the old store in place until
each user's next connect proves the new one. T20's both-ways record is
reversibility as a standing option: the vector index's home stays a decision precisely
because nobody burned the bridge behind the unwind.

**FC5, purpose-built is marketing until measured.** The failure-class table's
line reads "'Purpose-built' is marketing until measured on your queries", and
ANALYSIS's cell specifies the mechanism: "FC5 purpose-built marketing: F09
(continuous aggregates fought the workload)". NexusTrade's premise sentence is
the class in one clause, "it wasn't 'purpose-built'", a marketing word doing the
deciding. The quiet returns invert the same discipline from the positive side:
Voucherify validated "changes as they happen" against its own traffic, and
Mixmax moved because its own queries were slow, not because a category said so.

**D3, the honesty flag.** The deferral holds this chapter's evidential boundary:
returns are documented from small/mid teams only, big-tech reversal is
undocumented, and the corpus records the likely gap rather than asserting an
absence. Both halves of that sentence are findings, and both shape how far this
chapter's lesson generalizes: the return class's lesson is price-proven at scales
where one team owns the decision, and its extension to fleets the size of the
exit chapters' is a hypothesis, not a record.

## 6. Spec state and what the chapter hands forward

**Spec state.** BOOK-PLAN's row 16 schedules inputs "T10, T11, T13, T19, T20;
C44, C45, F09, F11" with patterns "P8, P20, FC5". The chapter map's Ch. 16 line
goes further, adding "+ T11 Uber;" explicitly and the R6 outflow flows running
from T23 to T27, whose map text reads "Yelp standardization,
Deliveroo/Zomato/Freecharge outflows, Bhoos return". The divergence is the
by-now-standard one (recorded at Chapters 6, 11, 13, and 15): the plan's case
columns predate R6, and the map governs, so the chapter reads the outflow
cluster and flags the plan cell here. Two smaller notes in the same spirit: the
plan's line places T11 inside its main list while the map adds it
parenthetically, and the plan's case column carries no T25 because Chapter 14
owns that row's mechanics, which the section above honors by pointer.

**Forward pointers.** Chapter 17 opens Part III with the backup-failure class
(F02, F04, F05, F06 under FC1), the failures this chapter's survivors avoided by
keeping old paths intact. Chapter 20 owns F09's and F11's full failure-class
readings, "wrong transition" and "wrong communication". Chapter 21's driver
census will count T23's standardization driver and T26's traffic-spike driver
alongside the exits' drivers. And Chapter 22's playbook draws replay mechanics
from the rows this chapter read first-hand: T19's side-by-side sync, T23's
read-first stagger, T27's lazy per-user return.

## Exercises

●○○ (read the evidence column). Four rows in the moves table carry figures that
need a caution flag before generalizing: Voucherify's ~30% cut (self-reported,
cross-post title), Freecharge's 60% lower cost of ownership (vendor-reported,
destination vendor's blog), Pinecone's cost model (vendor post), and Zomato's
50% (first-party, with one channel co-authored by the vendor). Classify each by
evidence level and name the pattern whose fails-when covers vendor-reported
cost claims. (Answer: FP self-reported, VP, VP, FP with a co-authored channel;
P16's fails-when, which reads "Cost is vendor-reported and unaudited" and then,
after the P-index's em dash, "verify before generalizing".)

●●○ (apply the mirror). Write the one-sentence engineering summary of T11 and
T23 that survives P8's fails-when test. (Worked answer: Uber moved because its
operators could run MySQL's mechanics and could not run its Postgres
configuration, and said so in engine terms; Yelp moved because only a small
rotation of long-tenured employees knew its Postgres well enough for outage
response, and said so in team terms. The second survives the fails-when; the
first is the conflation.)

●●● (plan the return). Your 1 TB MongoDB estate must return to PostgreSQL with
zero user-visible downtime, in the spirit of Voucherify's record. Draft the
side-by-side plan: the origin of the copy, the sync path, the validation loop,
the switch order, and where the triggers live. (Worked answer at the register's
level of detail: dual writes into a JSONB-staged schema, AWS DMS for the
backfill, Postgres transform triggers for document-to-row mapping that can carry
variables, sanity checks on both sides before the read switch, application
cutover flags per workload, DMS tasks stopped and removed only after anomaly
detection is quiet. The schedule Voucherify published is the realistic prior:
the write-and-test months first, then the data-movement months.)

◆ (pattern diagnostic). Which move in this chapter's table would have ended
unrecoverable, not merely failed, if the mover had dismantled the old path at
cutover? Name the pattern and its two clauses. (Answer: T13's NexusTrade round
trip; P20's fails-when reads "The old path is dismantled at cutover", then,
after the P-index's em dash, "the first post-migration regression becomes
unrecoverable", and ANALYSIS's P20 row records why that did not happen here:
"NexusTrade survived because old path intact". Uber's T11 is the contrast: the
old path was abandoned by attrition, which is why its error could only be
published, not repaired by rolling back.)

## Sources

All URLs byte-copied from `corpus/REGISTER.md`:

- C44/T19 slice (Medium, FP): https://engineering.slice.bank.in/seamlessly-migrating-from-mongodb-to-postgresql-leveraging-data-platform-for-scalable-success-404e2577a47b?gi=dd706d6a831e
- C45/T19 Infisical (FP): https://infisical.com/blog/postgresql-migration-technical
- C65/T19 Voucherify blog (FP): https://www.voucherify.io/blog/how-we-moved-from-mongodb-to-postgres-without-downtime
- C65/T19 Voucherify author cross-post (FP, the ~30% title): https://dev.to/msedzielewski/how-we-moved-from-mongodb-to-postgres-without-downtime-and-cut-our-costs-by-30-7fd
- T10 Mixmax (FP): https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb
- T11/F11 Uber (FP): https://www.uber.com/us/en/blog/postgres-to-mysql-migration/
- T11/F11 Ayende rebuttal (SR): https://ayende.com/blog/175137/re-why-uber-engineering-switched-from-postgres-to-mysql
- T13/F09 NexusTrade (SR): https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003
- T20 Confident AI (FP): https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector
- T20/C35 Pinecone (VP): https://www.pinecone.io/blog/pinecone-vs-pgvector/
- T23 Yelp (FP): https://engineeringblog.yelp.com/2024/10/migrating-from-postgres-to-mysql.html
- T24 Deliveroo (VP, AWS Database Blog): https://aws.amazon.com/blogs/database/how-deliveroo-migrated-their-dispatcher-service-to-amazon-dynamodb/
- T26 Freecharge (VP, AWS Database Blog): https://aws.amazon.com/blogs/database/freecharge-lowered-their-identity-management-system-cost-and-improved-scaling-by-switching-to-amazon-dynamodb/
- T27 Bhoos (FP): https://bhoos.games/blog/migrating-from-dynamodb-to-postgresql/
