# Chapter 11 - The Transition Matrix: How to Read It

**Governing idea.** A transition row in the register is a claim about a
family-to-family move: where the data came from, where it went, who moved it,
what they said it cost them, and how good the evidence is. The register holds
thirty of them (T01 through T30, the last eight added by the R6 verification
pass). Read together, they form a matrix, and the matrix has a shape: some
moves are documented many times over, some exactly once, and some not at all.
The gaps are data too. The survey ledger's standing note puts it plainly:
nobody blogs "we migrated and it was fine"; Stack Overflow / 37signals
matter as the silent majority done well. Survivor bias, in other words, is
a finding about where evidence comes from, not an absence in the world. This
chapter teaches you how to read the matrix before Part II walks its dense
cells chapter by chapter.

## 1. What a transition row is

Every T-row in the register (`corpus/REGISTER.md`) carries five fields: the
from-family and to-family, one or more named movers, a claimed outcome
summary, an evidence level, and exact source URLs copied at gather time. The
rows came from six research rounds (R1 through R6): R1 through R4 built the
base twenty rows, R3 deliberately hunted the quiet directions (document to
relational returns, the vector unwind in both directions), R5 added T21 and
T22, and R6, a check against a third-party report's case list, added T23
through T30.

Three things to notice about the rows as a class of evidence.

First, the evidence ladder tilts toward first-party. Of the thirty rows,
the large majority carry at least one FP source; the older the round, the
more PR and SR rows (T09 Apple's Cassandra-to-FoundationDB direction is
press-reported and says so in the row itself: "direction, not completed
fact").

Second, a third axis hides inside the family labels: managed versus
self-run. T24 (Deliveroo RDS to DynamoDB), T25 (Zomato TiDB to DynamoDB),
T26 (Freecharge Aerospike to DynamoDB), and T27 (Bhoos DynamoDB to
PostgreSQL) are all moves whose real variable is not the data model but who
operates the machine. Part II's Chapter 16 reads that axis explicitly.

Third, the outcome field is a summary written by the survey, not a source
quote. Where this chapter quotes a register summary, the wording is the
register's; where it quotes the underlying post, the quote is verbatim from
the source and named as such. The distinction matters whenever a number is
in play.

## 2. The matrix: thirty rows, grouped by destination

| Destination | Rows | Named movers | The short version |
|---|---|---|---|
| Columnar (ClickHouse-class) | T05, T06, T21, T28, T29, T30 | Cloudflare, Cloudflare logs, Langfuse, Uber, Zerodha, Zerodha | Six rows point at one engine class; the densest destination in the corpus |
| Key-value (including managed) | T01, T10, T24, T26 | Amazon ledger, Mixmax, Deliveroo, Freecharge | DynamoDB appears in all four |
| Distributed SQL | T03, T04, T22 | DoorDash, Google, Pinterest, Uber Docstore | Three eras of the same destination |
| Wide-column (family internal or into) | T02, T15, T07, T08, T16 | Netflix, Twitter, Discord, Facebook Messages, Instagram | Five rows, three of them engine swaps |
| Managed KV to relational | T27 | Bhoos | The reverse of the whole right column |
| Distributed SQL back out | T25 | Zomato | NewSQL is not a terminal state |
| Vector internals | T14, T18, T20 | turbopuffer, Spotify, Confident AI vs Pinecone | The only rows whose evidence runs both directions |
| Time-series platforms | T13, T17 | NexusTrade, Uber M3, Netflix Atlas | One round trip, two builds |
| RDBMS to RDBMS | T11, T23 | Uber, Yelp | Same family, opposite tales (F11) |
| Object storage, self-run | T12 | Dropbox | The one move where the "database" is the destination |
| Document to relational | T19 | slice, Infisical, Voucherify | The quiet return |

Read the shape before the cells. The origin side is dominated by one family:
relational or near-relational stacks (T01, T03, T04, T05, T11, T21, T23,
T29, T30 are nine rows whose origin is Postgres, MySQL, Oracle, or a
Postgres-based architecture). The destination side is dominated by another:
six rows land in columnar engines and four in one managed key-value store.
Graph, strikingly, appears on neither side of any row. Section 4 deals with
what that means.

## 3. What the dense cells say

**The columnar inflow (T05, T06, T21, T28, T29, T30).** Six of thirty rows
land in ClickHouse-class engines, and five of the six were added or rewritten
in R5 and R6, which means the direction was getting more common, or better
documented, as the corpus closed. The origins fan out: a Postgres OLAP
extension (Cloudflare's CitusDB, T05), a search engine (Cloudflare's logs,
T06), a Postgres-based product architecture (Langfuse, T21), another search
engine (Uber's logging platform and Zerodha's ELK stack, T28), a PG
TimescaleDB deployment (Zerodha's candles, T29), and plain Postgres back-
office analytics (Zerodha again, T30). The claimed outcomes share a shape:
fewer machines or fewer terabytes for the same scan work.

The two heaviest rows carry verbatim numbers. Uber's logging post: "a single
ClickHouse node could ingest 300K logs per second, about ten times what a
single ES node could achieve." Zerodha's z-connect infrastructure-upgrades
document: "reduced data storage from 20TB to 4TB while storing hundreds of
billions of financial records. The query performance has increased 1000x
performance in many scenarios." Two cautions. First, Zerodha's document is
first-party but written as a business-facing summary, and its numbers are
unaudited; the register records the caveat in its notes. Second, the same
z-connect document also contains "performance increase topping ~1000%"
elsewhere in the page, but that sentence belongs to the separate Postgres
(Citus) refactor project, not to the ClickHouse move; do not cross-pollinate
figures between rows, which is exactly the kind of slip the matrix's
row-by-row reading is designed to prevent.

**The key-value gravity (T01, T10, T24, T26, with T27 as the counter-row).**
DynamoDB is the destination in four of the five managed-or-not KV rows. The
movers range from Amazon's own Financial Ledger (T01, FP giant scale) to
Mixmax (T10, clean at small scale) to Deliveroo's Dispatcher service (T24,
sharding considered and rejected in favor of a managed KV) to Freecharge's
identity system (T26, "lowered their identity management system cost and
improved scaling", per the AWS blog's title, a co-authored vendor-published
account per the register's provenance note). Zomato's billing row (T25, the register's
"~50% monthly cost reduction" summary) carries it in the company's own words:
"our billing system's monthly expenses saw a notable 50% reduction due to
these changes". T27 runs the other way: Bhoos
Games took "more than 20 million user items from DynamoDB to PostgreSQL",
first-party, with a lazy per-user migration on next connect. Chapter 16
reads T26 and T27 as the same question answered twice, once in each
direction.

**The exits and the returns.** The origin side's heaviest movers are
relational exits (T01, T02, T03, T04, Chapter 12 and Chapter 14's material),
and the corpus's quietest cell is the return: T19's document-to-relational
rows are first-party but only from small and mid-size teams. Infisical's
account of its PostgreSQL migration reports that "the migration execution
turned out smooth with zero data loss"; the register pairs it with slice and
with Voucherify's zero-downtime side-by-side sync. The registered deferral
D3 bites here: big-tech reversals are undocumented, "likely survivor gap,
not absence". Nobody at large scale has published a document-to-relational
return; that is a statement about publishing, not about practice.

## 4. What the empty cells mean

Three kinds of gap live in the matrix, and the register records each one
instead of guessing over it.

No graph rows at all. Thirty transitions, and graph appears on neither side
of any of them. This is not because nobody migrates graph databases; it is
because graph moves are rarely published first-party. The registered
deferral D6 says exactly this: the R5 sweep for a first-party production
graph case beyond ICIJ, Zurich, and ShopBot "found only vendor/marketing
material; re-check later", and adds that graph companies rarely blog
first-party. The matrix therefore has an empty graph band, and
Chapter 6's cases (C20, C21, C22) are draws from a thin cell, which their
evidence levels already reflect.

Single-occurrence rows. T12 (Dropbox's S3 to Magic Pocket), T25 (Zomato's
TiDB to DynamoDB), T27 (Bhoos's DynamoDB to PostgreSQL), T22 (Uber's
Schemaless to Docstore), T26 (Freecharge), T24 (Deliveroo) each stand alone
in their direction. A single row proves the move is possible and
documented; it cannot prove the move is typical. When Part II leans on one
of these rows, it leans with the row's own evidence level attached.

The absence pattern that is a positive finding: T20 is the register's only
genuinely bidirectional row ("both directions documented" is the row's own
summary distinction between vector and the older families; every other
family's counter-flow is missing). The survey's honest reading, per watch
W1, is that the vector family's unwind is simply easier to publish: the
engines are young, the stakeholders are fewer, and the cost arithmetic is
public. Watch W1 remains open because the decisive datum, a first-party,
large-scale migration in either direction, still does not exist.

Verdict (assessment): read the matrix as a map of published practice, and
treat every empty cell as three possibilities compressed together: the move
is rare, the move is unpublished, or the move is unwise. Only the third
reading, the one the industry loves, is not supported by the register. The
deferrals (D3, D5, D6, D8) exist precisely to keep this honest: D5 says of
the Foursquare venue-database claim that it should not be claimed "without
a better source"; D8 records that for Razorpay and Swiggy "no genuine
engineering write-ups found". Silence is not evidence of absence; in this
corpus, silence is recorded and dated.

## 5. Reading the matrix with the playbook and reversibility

Two patterns from the P-index govern how to read rows: P19, the migration
playbook (dual writes, backfill, shadow reads with diffing, incremental
cutover per tenant or shard, rollback kept warm), and P20, reversibility as
the survival requirement. Every row in the matrix can be read on two
questions. Did the movers do it in stages? Could they have gone back?

**P19's faces across the matrix.** The register's rows split cleanly on
playbook publication. Rows with published mechanics: T03 (DoorDash's three
cutover attempts, the last "lossless"), T07 (Discord's checkpointed
migrator, three months of planned work compressed to nine days), T19
(Voucherify's side-by-side sync with triggers and a data-migration service),
T23 (Yelp's staged reads, then staged writes), T27 (Bhoos's lazy per-user
migration on next connect). Yelp's account is the cleanest staging sequence
in the whole matrix: "switched the write logic to write to MySQL first, and
finally (several months later) turned off Postgres writes entirely and
cleaned up much of the code we had written." Rows with outcome-only
summaries: most of the pre-R5 rows name the move and the claimed outcome but
publish no mechanics at all. The gradient is itself a finding: the newer the
row, the more likely the playbook is public. Teams in 2024 to 2026 publish
verification tooling; teams in 2013 to 2017 published victory laps. When a
row predates the playbook era, its mechanics must be treated as unknown, not
as healthy.

**P20's faces across the matrix.** Reversibility shows up wherever the move
could have failed without killing the mover: T12 (Dropbox's Magic Pocket was
"deliberately reversible" per the register's summary, with bidirectional S3
replication), T13 (NexusTrade's two-week round trip, which survived because
the old MongoDB path was still intact, failure F09's moral), and the
rollback-kept-warm discipline the register's P20 faces list. The matrix also
shows what non-reversibility costs: T05's ClickHouse migration was exemplary
until a later billing regression (F08) surfaced in the unprofiled query
layer, which is Chapter 19's material but is visible from the matrix as
T05's asterisk. The register's one-line summary even carries it: exemplary,
"but later billing regression (F08)".

The reading rule this chapter hands to the rest of Part II: for any
transition row, ask (a) what evidence level does the row carry, (b) is the
playbook published, (c) was the old path kept warm. A row strong on all
three is a load-bearing case; a row strong on none is a caution about how
the move reads, not proof the move works.

## 6. Patterns, spec state, and what Part II does with the matrix

**The patterns.** This chapter introduces no machinery of its own; it names
the two reading lenses. P19's registered faces are C01 (Slack read-diffing),
C41 (MyRocks staged), T07 (Discord checkpointed migrator), and C04
(per-shop moves); its fails-when is any step skipped, "especially the
diff/verification tooling, 'where migrations die'". P20's registered faces
are C39 (Magic Pocket's bidirectional replication), T13 and F09 (the
NexusTrade round trip), and C41 (InnoDB rollback replicas); its fails-when
is dismantling the old path at cutover. Both patterns' faces sit inside
rows this chapter has already touched, which is the point: the patterns are
extracted from the matrix, not imposed on it.

**Spec state.** Per the Chapter 6 convention, a spec discrepancy is flagged
in the chapter rather than silently resolved: this chapter's BOOK-PLAN line
still says "T01-T20 survey", written when the register held twenty rows;
R5 and R6 added ten more, and the PROJECT chapter map's line, "Ch. 11 The
transition matrix: how to read it; T01-T30 survey", is the current spec.
This chapter surveys all thirty rows and is written against the chapter
map's line.

**Where each cell goes.** The dense columns are pre-assigned: T01 and T02
lead into Chapter 12 (relational exits and their afterlives); the P4-P7
tenancy patterns organize Chapter 13; T03 and T04 are Chapter 14's
NewSQL absorptions; T07, T08, and T16 are Chapter 15's engine-swap twins;
and T10, T11, T13, T19, T20, plus the R6 flows T23 through T27, are
Chapter 16's returns and round trips. Watch states condition three cells:
W1 (vector, open, no first-party large-scale move either way), W2 (Pinterest
TiDB long-term outcome, the inverse of T25's Zomato flow), and W5 (OpenAI's
write-scaling trajectory, the corpus's live test of the unsharded
counter-case).

Verdict (assessment): the matrix is the book's spine because it shows that
the industry's public migration record is dense in exactly the cells where
the pain is chronic (relational exits, logs-to-columnar, managed KV) and
empty where practice is newest or quietest (graph, big-tech returns, NewSQL
exits). Chapters 12 through 16 read the dense cells one family at a time;
Chapter 21 then re-reads the whole matrix for drivers, and Chapter 22 for
mechanics.

## Exercises

●○○ (read a row, state its level). Take the register's T24 row (Deliveroo's
Dispatcher service, RDS PostgreSQL to DynamoDB): what evidence level does it
carry, what does the row's summary say was rejected in favor of managed KV,
and which chapter of Part II owns this row? Work the example: check the
evidence ladder in the register's header, name the axis hiding inside this
move, then write the one-line summary you would put in the matrix cell of
your own architecture notebook.

●●○ (plan from the dense cell). Your org ships 2 TB of logs per day through
Elasticsearch and is evaluating the columnar cell (T06, T28). Using this
chapter's verbatim anchors, write the capacity paragraph for a proposal:
what does a single ClickHouse node ingest in the cited experiment, what does
the storage outcome look like in the Zerodha row, and what must the plan
contain that the register's P19 fails-when names before any node is
installed?

●●● (design the reverse). Your team is on managed DynamoDB at roughly 25
million items in a game-ledger shape and wants the relational query power a
managed KV does not give; the counter-move (T27) is the closest published
case. Design the migration plan: Bhoos's lazy per-user trigger, the staging
sequence from Yelp's row that fits, and the P20 requirement the round trip
in T13 made visible. Then state, using W1's open verdict, how much comfort
the register can and cannot offer about published precedent, and which
absence in the matrix your plan's risk register must record.

◆ (diagnostic, no hints). An org migrated off its old store, hit a relapse
in a layer nobody profiled, and returned in days because the old write path
had never been dismantled. Which registered pattern did the survivors run
that their peers did not, and what is the pattern's registered fails-when?
(You will need both the playbook pattern and the reversibility pattern; the
answer is one pattern, name it and its face.)

## Sources

The chapter's factual base is the register's transition table
(`corpus/REGISTER.md`, T01-T30), the deferral and watch rows of
`corpus/STATE.md`, and the P-index in `PROJECT.md`; register rows cited in
this chapter were verified live during the survey rounds (R1-R6, sessions
2-3), and the eight source-level quotes were re-fetched and verified verbatim
from the live pages in this session:

- https://www.uber.com/us/en/blog/logging/ (T28)
- https://zerodha.com/z-connect/business-updates/zerodhas-infrastructure-upgrades (T29, T30)
- https://blog.zomato.com/switching-from-tidb-to-dynamodb (T25)
- https://engineeringblog.yelp.com/2024/10/migrating-from-postgres-to-mysql.html (T23)
- https://bhoos.games/blog/migrating-from-dynamodb-to-postgresql/ (T27)
- https://infisical.com/blog/postgresql-migration-technical (T19)
- https://aws.amazon.com/blogs/database/freecharge-lowered-their-identity-management-system-cost-and-improved-scaling-by-switching-to-amazon-dynamodb/ (T26)
