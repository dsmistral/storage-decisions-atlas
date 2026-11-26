# Chapter 15: The engine-swap twins

## 1. Context: the smallest move the architecture allows

Every chapter in this part so far has read moves that changed the database's
public face: Chapter 12's exits re-platformed whole programs, Chapter 13's
counter-program changed the topology while keeping the engine class, and
Chapter 14's absorptions moved customers into engines that speak a new
protocol. This chapter reads the smallest move in the transition matrix: keep
the database, keep the protocol, keep the schema and the administrative
tooling, and replace only what sits below them. The BOOK-PLAN's row 15 names
it as its single idea: "Same protocol, new internals: the lowest-blast-radius
move, independently invented three times". The survey's comparison section
calls the three cases "the engine-swap twins": they kept "the protocol, schema
and operations stack, and replaced only storage internals" and "all three kept
rollback paths" ([SURVEY](#sources), lineages below).

The prescription discipline first: a transplant has an enabling condition.
The P-index states P12's bold rule name as "Engine swap, same protocol" and
its rule text as "replace storage internals, keep schema/protocol/ops",
with the tail phrase "the lowest-blast-radius move" carrying the same claim
the BOOK-PLAN's idea names. Its fails-when draws the boundary the family
chapter drew: "The pain is in the data model or query layer, not the storage
engine."
Chapter 4 already made that diagnostic the cure ladder's hinge for this
family, and it read two of this chapter's movers in depth there: C38 (Discord's
full exit) and C16 (Instagram's Rocksandra transplant). This chapter does not
re-read those cases; it reads the transition rows they anchor (T07, T16) for
what a swap costs and buys, adds the family's relational twin (T08,
Facebook's InnoDB-to-MyRocks program, with cases C41 and C42), and reads the
three side by side as one pattern.

Why "twins", and why "three times"? The name is the survey's; the census is the
ANALYSIS's pattern table: its P12 row reads "C41 (−62.3% instance size,
rollback replicas), T07 (177→72 nodes), C16 (Rocksandra), C42 (HBase→MyRocks)"
with the observation that this is "the matrix's lowest-blast-radius move;
repeated independent invention is the signal". Three different teams,
none coordinating, converged on the same move for the same class of pain:
Instagram and Facebook each independently selected RocksDB as the new internals
under two different databases, and Discord selected a C++ reimplementation of
Cassandra's own protocol. The pattern's repeated invention is the chapter's
evidence that the move is a real option class, not one writer's taste.

## 2. The moves

| Row | Move | Keeper | Swapped | Stated driver | Evidence |
|---|---|---|---|---|---|
| T07 | Discord: Cassandra to ScyllaDB | Cassandra's API and data model | JVM storage side, for a C++ shard-per-core engine | GC pauses, hot partitions, node count | FP |
| T08 | Facebook: HBase to MySQL/MyRocks (Messenger) | none shared; a repatriation to the relational stack | the wide-column stack itself for MyRocks under MySQL | operating the HBase stack was the burden | FP |
| T16 | Instagram: RocksDB inside Cassandra | everything above the storage layer | Cassandra's JVM storage engine for a C++ RocksDB engine | GC-bound read latency | PR |
| C41 | Facebook UDB: InnoDB to MyRocks | MySQL protocol, schema, tools | InnoDB's B-tree for RocksDB's LSM | space-bound on flash | FP |

Two register rows and two cases carry the three transitions. T08 is the
wide-column exit from HBase and its mover is C42 (Messenger); C41 is the
relational engine swap the same success story made possible: the register
records Messenger's move as "Motivated by MyRocks success; operating the
HBase stack was the burden". The register's C41 row carries the program's
window (2016 to 2019) and its headline figures at source; the window applied
to Messenger is not verbatim in any fetchable source (see the mechanics below)
and is not asserted here.

Note what is absent from the driver column: none of the three register rows
records a data-model complaint. The pain is always mechanical: garbage
collection of a managed runtime, space and write amplification on a specific
storage medium, or the operational weight of a parallel stack. That is P12's
enabling condition, stated in the source rows themselves.

## 3. The mechanics of three swaps

### T07, Discord: a protocol-compatible reimplementation (C38's move)

Discord's blog is the matrix's most complete single account of a swap-sized
migration, and its diagnosis of why Cassandra hurt comes verbatim from the
page: "GC pauses affecting latency, all the way to super long consecutive GC
pauses that got so bad that an operator would have to manually reboot and
babysit the node in question back to health". The page's case for the
replacement is equally concrete: "Its promise of better performance, faster
repairs, stronger workload isolation via its shard-per-core architecture, and
a garbage collection-free life sounded quite appealing", and it adds the
candid caveat: "ScyllaDB is most definitely not void of issues, it is void of
a garbage collector, since it's written in C++ rather than Java."

The requirement was stated by the team itself: "we need to migrate trillions
of messages with no downtime". The mechanics ran the playbook in Chapter 4's
summary and will be P19's faces there: the team first measured an estimate
that "It requires a lot of tuning, and once we get it set up, we have an
estimated time to completion: three months", then rebuilt the migrator and
reported, "We hook up our new and improved migrator and get a new estimate:
nine days!" The migrator itself is the checkpoint discipline in miniature:
"It reads token ranges from a database, checkpoints them locally via SQLite,
and then firehoses them into ScyllaDB." The outcome spans are the register
row's figures at source: "It's a much more efficient database" and "we're
going from running 177 Cassandra nodes to just 72 ScyllaDB nodes." InfoQ's
independent account carries the same census and a sharper tooling sentence:
the team "decided to implement a bespoke solution in Rust with SQLite used
for checkpointing, which allowed them to shorten the migration time from
three months to nine days" ([InfoQ](#sources)).

Two confirmed spans strengthen the same-protocol reading. The team's
suspicion quote: "We were suspicious that slapping a new database on our
system wasn't going to make everything magically better." And the sequencing:
"by 2020, we had migrated every database but one to ScyllaDB", messages last,
exactly the incremental-then-flagship order the P-index's P19 faces record.

### C41, Facebook UDB: LSM internals under the same MySQL

The program's origin is one sentence of arithmetic on the migration post:
"Our objective was to migrate one of our main databases (UDB) from compressed
InnoDB to MyRocks and reduce the amount of storage and number of servers used
by half." The diagnosis is the enabling condition in miniature: "Even though
we used InnoDB compression and there was excess CPU and random I/O capacity,
it was unlikely that we could enhance InnoDB to use less space." The intro
post's opening carries the workload and the win in the same paragraph: "we
were able to use 50 percent less storage for the same amount of data compared
with compressed InnoDB. Ultimately this will allow us to use half as many UDB
servers."

The swap's blast radius is the point, and the post states it: "No application
change was needed because MySQL client protocols and schema definitions were
the same. (There were actually some minor differences, like Gap Lock, but
they were relatively easy to address.)" The VLDB paper's abstract says the
same in one phrase: "The transition was mostly transparent to client
applications." A swap is not a re-platform: the register's C41 row records
the components that made that true (same schema, same replication, same
administration tools), and the post's mechanism list is the receipt: "MyRocks
could replicate from InnoDB" and "InnoDB could also replicate from MyRocks".

That bidirectional replication is where this chapter's P20 reading lives, and
the post's own words are the record: "We could deploy MyRocks on primary
without destroying InnoDB. This allowed us to demote MyRocks and promote
InnoDB again if an issue arose in production." Around it sits the verification
batter the survey summarizes as "careful stages with double-write consistency
checks": a primary-and-secondary-key checksum tool, a cross-instance
consistency check pinned by GTID snapshots, and a shadow layer: "We created a
tool to shadow (replay) captured queries to multiple instances and compare
results. This made sure that production queries were seeing exactly the same
results between InnoDB and MyRocks." The paper's completion sentence anchors
the timing: "Facebook completed the UDB migration from InnoDB to MyRocks in
2017", with the headline figures in its results paragraph: "The InnoDB to
MyRocks UDB migrations were completed in August 2017. For the same data sets,
MyRocks and modern LSM-tree structures and compression techniques reduced the
instance size by 62.3% compared to compressed InnoDB" and "These gains enabled
us to reduce the number of database servers in UDB to less than half, saving
significant resources."

### C42, Facebook Messenger: the exception that proves the boundary

Messenger's HBase-to-MyRocks move shares the paper, and the register rows its
motivation: "Motivated by MyRocks success; operating the HBase stack was the
burden". The paper's own discussion of it sits beyond this environment's
fetch cap on the PDF, so its mechanism detail here rests on indexed excerpts
of the same paper, noted as such: the Messenger migration "was more complex
than UDB since HBase and MySQL were very different database products", the
data moved via "MapReduce jobs to extract the data from HBase and loading it
into MySQL", and "During the transition phase, consistency was checked by
double writing and reading." The snippet also records that the project ran
"in conjunction with refactoring the Facebook Messenger application."

That last detail is the boundary made explicit. Facebook's own record sizes
the Messenger move as a different product class from UDB's: not a
protocol-compatible swap but a cross-product migration with an application
refactor attached. The survey's engine-swap comparison lists C42 among the
twins because its register row records the same motivation class (ops burden,
not data model), and the P12 reading holds; but its mechanics belong to the
heavier end of the ladder, and the paper itself says so. Messenger is the
chapter's within-cases caution: the "same protocol" discount is earned only
when the protocol really is the same.

### T16, Instagram: RocksDB inside Cassandra, demonstrated in public

Chapter 4 read C16 in depth; this chapter's addition is the transition row's
own lesson, and it comes with the corpus's weakest evidence level (PR) and a
dead-fetch caveat. The register's only source URL is The New Stack's interview
page, which this session confirmed again as cookie boilerplate before any text
(fetched at the cap with no article body); the quotes here therefore rest on
indexed excerpts of that page and of the interview's native home, noted as
such per the Ch. 4/8 convention. The New Stack excerpt sizes the artifact: "Rocksandra is a fork, said Francois Deliege, Instagram engineering manager,
but not one designed to compete with Cassandra itself", and the interview's
native record (the Cassandra project's own case page, which is not in the
register and is noted here only as corroboration) carries the diagnosis
verbatim: "read latency was a concern due to garbage collector (GC) issues,
with a P99 read latency in the range of 25ms to 60ms, depending on client
traffic. To address this, the team created Rocksandra, a C++ storage engine
based on RocksDB."

What T16 records that Ch. 4 deferred is the demonstration effect. The same
excerpt trail records the team open-sourcing the engine ("We have open
sourced our Rocksandra code base and benchmark framework, which you can
download from Github"; the team's own announcement post is not in the
register, so the quote is snippet-corroborated only and noted for the survey
pass), and the experiment feeding Cassandra's pluggable-storage-engine
direction, which Ch. 4 already carried. An engine
swap designed as a demonstration, not a war, is the smallest evidence for the
pattern's claim that the move is among the options any Cassandra operator
holds; the register keeps the case at PR for the same reason.

## 4. What the three swaps share, and where the rule breaks

The shared spine first, from the sources themselves. Same motivation class:
GC on the JVM (T07, T16) or space and write amplification on flash (C41),
with C42's ops burden as the outlier that still names an operational, not a
semantic, driver. Same conservation law: the API, schema, and operational
surface stay; the register's C41 row's "open-sourced (Percona)" and the paper's
"mostly transparent to client applications" are the same fact seen from the
two sides. Same playbook, at three scales of ceremony: Discord's checkpointed
migrator (measure, re-measure, checkpoint, cut over), Facebook's
replication-in-both-directions plus three verification tools, Instagram's
engine built behind the existing surface with results published for
inspection. The survey's comparison section reads the trio as "the
lowest-blast-radius move in the matrix, and its repeated independent
invention is the strongest signal that 'swap the engine, keep the database'
deserves a place in every team's option set" (assessment, survey).

Where the rule breaks is the migration's cost curve. The three swaps are not
equally cheap, and the corpus's own rows price them: T16 is a patch-level
change to a single system (a fork of one engine's internals), C41 is a
two-year staged program that still required building and maintaining a new
storage engine first, and T08's Messenger leg was complex enough that the
team refactored the application around it. The blast radius shrinks, but the
work concentrates into the parts of the swap that no product gives you: the
new internals themselves (C41, C16) or the bespoke migration and verification
tooling (T07, C41). The survey's P15 lens reads the same cases the other way:
"Build only what converges", and MyRocks (C41/C42) is one of its registered
faces, the build that became Percona's shipping engine.

Verdict (assessment). The engine-swap twins are the matrix's quiet master
class because each team looked at a problem that could have justified a
risky destination change and asked the smaller question first: is the model
wrong, or is the engine wrong? In all three cases the model was right and
the engine was wrong, and in all three the fix fit inside the existing
contract. The pattern's one sentence of warning, from the P-index, is that
the diagnosis must be honest: an engine swap cannot fix a data-model problem,
and the twins' own C42 shows what happens when the replacement engine is
different enough that the application must follow.

## 5. Patterns: P12 with its playbook and reversibility guards

**P12 (engine swap, same protocol).** The P-index's face list is "C41
(InnoDB→MyRocks), T07 (Discord→ScyllaDB), C16 (Rocksandra)" and its
fails-when is "The pain is in the data model or query layer, not the storage
engine." The ANALYSIS pattern table's P12 row carries four faces: "C41 (−62.3%
instance size, rollback replicas), T07 (177→72 nodes), C16 (Rocksandra), C42
(HBase→MyRocks)", adding C42 to the P-index's three.
The distinction matters for the reader who quotes the face list: ANALYSIS's
table, not the P-index, is where C42's twin membership is registered.

**P19 (the migration playbook).** The P-index's P19 face list includes two of
this chapter's movers: "C41 (MyRocks staged)" and "T07 (Discord checkpointed
migrator)", with the fails-when split at its dash into two contiguous
fragments: "Any step skipped" and "especially the diff/verification tooling,
'where migrations die'." ANALYSIS's P19 row adds
the precision that C41's face is "staged + consistency checks" and T07's is
"checkpointed Rust migrator, 9 days". The three verification tools in the C41
post are the diff/verification step in the most documented form in the corpus,
because the engine-swap shape makes continuous diffing cheap: while both
engines speak the same protocol, both can serve the same shadow queries.

**P20 (reversibility is the survival requirement).** The P-index's P20 face
list includes "C41 (InnoDB rollback replicas)", and ANALYSIS's row groups it
with "C39, T13/F09 (NexusTrade survived because old path intact)" with its
fails-when cell noting "F06: no out-of-domain backup = nothing to return to."
The C41 mechanism is the pattern's cleanest implementation: the old engine kept
replicating from the new one so that "This allowed us to demote MyRocks and
promote InnoDB again if an issue arose in production." Discord's plan kept
the Cassandra fleet warm through the cutover window, and messages moved last
for the same reason. The chapter map's Ch. 22 owns the playbook as a whole;
this chapter's contribution is the observation that P12's same-protocol
constraint is what makes P19's diffing and P20's rollback cheap enough to
actually run.

## 6. Spec state and what the chapter hands forward

**Spec state.** BOOK-PLAN's row 15 reads "The engine-swap twins | Same
protocol, new internals: the lowest-blast-radius move, independently invented
three times | T07, T08, T16; C41, C16, C42 | P12, P19, P20". The chapter
map's line reads "The engine-swap twins: T07, T08, T16 (P12)". The pattern
columns differ: the plan carries P19 and P20 alongside P12, the map names
only P12. This chapter follows the plan (the playbook and reversibility rows
are load-bearing in the mechanics and patterns sections above) and records the
divergence per the Ch. 6/11/13 convention. The case-column relation needs one
sentence too: the plan's C41, C16, C42 sit behind the transition rows as
their movers (C41 is T08's enabling program and a transition in itself, C16
is T16's mover, C42 is T08's mover), and C38, Discord's case row, was read
whole in Chapter 4 and enters here only through T07.

**Forward pointers.** Chapter 16 owns the returns-and-round-trips cluster
(T10, T11, T13, T19, T20 plus the R6 outflows T23 through T27), and per Ch.
14's division of labor it owns the outflow reading while this chapter hands
it the swap-side contrast: a round trip is what happens when the keeper of
the old path fails and P20's discipline is the difference between an
expensive lesson and an unrecoverable one. Chapter 21's driver census will
find this trio reporting operational pain and cost, one class, three engines;
Chapter 22's playbook will find its cheapest implementation here, where
diffing costs a shadow query instead of a dual-write stack. The survey's
watch-list W2 (Pinterest's long-term TiDB verdict) and W1 (large-scale vector
unwinds) are unaffected by this chapter.

## Exercises

●○○ (read the evidence column). Three register rows document engine swaps:
one FP, one PR, one FP. Name each row's evidence level, and for the PR one,
state in one sentence what the dead-fetch of its only source URL means for
how a reader should weight its figures.

●●○ (apply the enabling condition). Your Cassandra cluster's p99 reads are
climbing and the on-call rotation dreads repairs, but your team also wants
lightweight transactions across partitions. Write the two-column diagnosis
P12's fails-when requires (data model vs storage engine) and name which
column an engine swap can address; use one quote from this chapter's sources
in each column.

●●● (price the consistency check). Facebook ran three verification tools
(key-checksum, cross-instance GTID comparison, shadow query replay) during
C41. For each tool, write one sentence on what failure it catches that the
other two cannot, then identify which one would survive if the destination
engine could not replicate from InnoDB at all, and why.

◆ (pattern diagnostic). A team wants to replace self-managed Cassandra with
a managed Cassandra-compatible service but their pagination code relies on
Cassandra's exact compaction-trigger behavior, which the managed service's
engine does not reproduce. Which of P12's fails-when clause binds, and what
would you check next? Answer: the pain has moved into the app-visible layer
(query/pagination semantics), which is P12's fails-when ("The pain is in the
data model or query layer, not the storage engine"); check whether the
behavior gap is a tunable or a semantic difference before swapping (P19's
diff step exists precisely to catch this before cutover).

## Sources

All URLs byte-copied from `corpus/REGISTER.md`:

- C38/T07 Discord (Discord blog, FP): https://discord.com/blog/how-discord-stores-trillions-of-messages
- C38/T07 Discord (InfoQ, PR): https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/
- C41 Facebook (FB intro, FP): https://engineering.fb.com/2016/08/31/core-infra/myrocks-a-space-and-write-optimized-mysql-database/
- C41 Facebook (FB migration, FP): https://engineering.fb.com/2017/09/25/core-infra/migrating-a-database-from-innodb-to-myrocks/
- C41/C42 Facebook (VLDB 2020, FP): https://www.vldb.org/pvldb/vol13/p3217-matsunobu.pdf
- C16/T16 Instagram Rocksandra (The New Stack, PR): https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/
