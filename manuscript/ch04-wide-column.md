# Chapter 4 - Wide-Column: Three Fleets and the JVM Cure

**Governing idea.** The wide-column family (Cassandra and its Bigtable-lineage
relatives) keeps one interface: a partitioned row key, sparse wide rows, and
per-query tunable consistency. Joining the family settles the data-model
question; what it opens instead is an operational bill: JVM garbage-collection
pauses, replica repair at scale, and hot partitions. The family's documented
record shows the cures, in ascending order of effort, are fleet discipline, an
engine transplant behind the same protocol (P12), a bridge over a managed
primitive (P3), and the honest loop of adopt, break, evaluate, and buy the
converged engine (P9). Four cases, one per posture toward the family: a fleet
operator (Apple), a transitional adopter (Netflix), an engine modifier
(Instagram), and a family exiter (Discord), with Twitter's Manhattan (C36) as
the companion.

**Cases in this chapter:** C14 (Apple on Cassandra, the planetary fleet), C15
(Netflix: Oracle, SimpleDB, Cassandra, the transitional stop), C16 (Instagram
"Rocksandra", the engine transplant), C38 (Discord: Mongo, Cassandra,
ScyllaDB, the full-treatment exit), plus C36 (Twitter's Manhattan) as the
companion. Register IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`.

---

## 1. Context: one interface, one operational bill

A wide-column store keeps rows under a partition key, and each row is a sparse
map of columns that can be arbitrarily wide. Writes are appends to a
log-structured store, so the write path is cheap and linear; reads and
maintenance carry the bill, because an append-only store must be compacted
behind the writer, and because in the dominant open-source implementation the
whole thing runs on the Java virtual machine. Consistency is tunable per
query: you choose, at call time, how many replicas must agree.

That is the family's bargain, and it explains the shapes of the cases in this
chapter. Nobody here is choosing a data model; they all already have one.
What varies is where the pain lands (the JVM, the primitive, the partition)
and what the operator does about it: bear it, bridge it, transplant it, or
replace it. The family is unusual in how much of that decision can be taken
without changing the interface, which is why half this chapter's cures keep
the same API.

## 2. The cases, each against its time

### The planetary fleet: Apple (C14)

The largest documented Cassandra fleet on record is credited to Apple by the
Apache Cassandra project itself: "more than 160,000 instances storing over
100 petabytes of data across 1,000-plus clusters" ([C14: DevPro Journal,
Cassandra 4.0 release](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/), PR).
At this scale the unit of work is not the node or even the cluster; it is the
fleet. The published record is correspondingly thin: no first-party
engineering account of Apple's Cassandra operations exists in the corpus,
which is itself a finding. The biggest fleet keeps the quietest records.

The one open thread is press-reported: since 2015, accounts have described
parts of iCloud and CloudKit moving toward FoundationDB ([C14: The Dulin
Report](https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/), [C14: Engineers' Codex](https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions), PR;
held as deferral D1, watch-list W1's transition question). The register's
caution stands: press-only, direction not completed fact, and Apple still
ships Cassandra patches.

### The transitional stop: Netflix (C15)

Netflix launched streaming in 2007 on Oracle behind one data center: "We had
a single data center, which meant we had a single point of failure"
([C15: InfoWorld](https://www.infoworld.com/article/2171162/big-movies-big-data-netflix-embraces-nosql-in-the-cloud.html), PR).
After a three-day database outage in 2008 (the register-recorded driver), the
company decided on the cloud: "We knew we had to get out of the data center,
so we could keep running and keep growing."

The move came in steps the register records as Oracle, then a stop at Amazon's
managed SimpleDB, then Cassandra in 2010 onward. SimpleDB proved unreliable
at Netflix's volumes (the survey's verdict on the stop, dossier 3) and was
itself replaced. This is P3's
canonical face, and it is worth reading precisely: the stop failed
the way P3 says stops fail, at volume, for a workload the primitive was not
built to carry. What makes Netflix's case the good version of P3 is what
happened next: the stop was treated as a bridge, the re-evaluation was
honest, and Cassandra became the backbone at "more than 10,000 instances
storing 6 petabytes across 100-plus clusters, with over 1 trillion requests
per day" ([C15: DevPro Journal](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/), PR;
Netflix's own account of life in the trenches is first-party
[C15: Cassandra from the trenches slides](https://www.slideshare.net/jasedbrown/cassandra-from-the-trenches-migrating-netflix-10586521)).

### The engine transplant: Instagram Rocksandra (C16)

Instagram runs Cassandra beside its sharded Postgres for feed-scale reads.
When the JVM's read path was the problem (read latency concern traceable to
garbage-collection issues), its engineers took the smallest move the
architecture allows: they replaced Cassandra's storage engine internals with
a C++-based RocksDB engine, keeping the same database around it. The result
was dubbed "Rocksandra", and the experiment fed Cassandra's later
pluggable-storage-engine direction ([C16: The New Stack](https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/), PR).

This is P12's purest face in the family: same data model, same protocol, same
operational surface, new storage internals. The diagnosis behind it is the
pattern's enabling condition: Instagram's pain sat in the storage engine
(read latency traced to GC issues), not in Cassandra's data model, so
a transplant could address it. That condition matters; it is what separates
this case from moves that had to leave the family entirely.

### The full-treatment exit: Discord (C38)

Discord outgrew MongoDB for messages in 2015 and moved to Cassandra in 2017
for linear write scale. By its own account the Cassandra years were an
operational grind: "from GC pauses affecting latency, all the way to super
long consecutive GC pauses that got so bad that an operator would have to
manually reboot and babysit the node" ([C38: Discord blog](https://discord.com/blog/how-discord-stores-trillions-of-messages), FP).
The cure, adopted after evaluating the options, was ScyllaDB, "a
Cassandra-compatible database written in C++" with a shard-per-core
architecture and faster repairs: "we're going from running 177 Cassandra
nodes to just 72 ScyllaDB nodes", each with 9 TB of disk against an average
of 4 TB per Cassandra node, with tail latencies that "improved drastically"
([C38: Discord blog](https://discord.com/blog/how-discord-stores-trillions-of-messages), FP).
The historical-message move itself was carried by a bespoke, checkpointed
migrator and went from three months to nine days ([C38: InfoQ](https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/), PR;
the migration machinery is T07, read in full in Chapter 15, and its playbook
is P19 in Chapter 22).

Read the whole arc and it is P9's loop enacted at scale: adopt, break,
evaluate honestly, and buy the converged engine. The survey's synthesis adds
the transferable nuance, marked as assessment: only about one-third of
Discord's fix was "a new database"; the rest was keeping load off the
database (data services, load management). The engine swap was the cheapest
leg of the cure, not the whole cure. A caution for scale honesty: the survey
account also shows the pattern's appeal is that ScyllaDB kept Cassandra's
data model and protocol, so years of operational knowledge survived the move.

### Companion: Twitter's Manhattan (C36)

The register's Twitter entry records Manhattan as a built store: built for
millions of QPS at low latency, and in 2016 retrofitted with manageable
strong consistency because uniqueness semantics were load-bearing
([C36: Manhattan](https://blog.x.com/engineering/en_us/a/2014/manhattan-our-real-time-multi-tenant-distributed-database-for-twitter-scale), [C36: strong consistency](https://blog.x.com/engineering/en_us/a/2016/strong-consistency-in-manhattan), FP).
What Manhattan replaced was an availability-first Cassandra era, which the
survey's dossier 12 records as the ladder MySQL → Cassandra → Manhattan.
For this chapter the companion's lesson is narrow but sharp: even a company
that ended up building its own store first went through the availability-first
interface and carried its costs; the consistency bought back (P10, the
2016 retrofit) is the same bill Chapter 3's C36 preview identified. The full
Manhattan arc lives in Chapter 12.

## 3. What happened (sourced outcomes, with evidence levels)

- **Apple (C14, PR).** Largest documented fleet per the Apache project's own
  credits: 160,000+ instances, 100+ PB, 1,000+ clusters ([C14: DevPro
  Journal](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/));
  FoundationDB direction press-only and partial (D1; [C14: Dulin Report](https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/)).
- **Netflix (C15, FP/PR).** Oracle exited to the cloud; SimpleDB stop bridged
  and left behind; Cassandra backbone at 10,000+ instances, 6 PB, over 1
  trillion requests per day ([C15: InfoWorld](https://www.infoworld.com/article/2171162/big-movies-big-data-netflix-embraces-nosql-in-the-cloud.html); [C15: DevPro Journal](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/)).
- **Instagram (C16, PR).** Rocksandra engine swap shipped behind the same
  Cassandra surface; the experiment fed the pluggable-engine direction
  ([C16: The New Stack](https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/)).
- **Discord (C38, FP).** Trillions of messages moved Cassandra→ScyllaDB in a
  staged migration with messages migrated last; 177 nodes to 72; the bespoke
  migrator went from three months to nine days
  ([C38: Discord blog](https://discord.com/blog/how-discord-stores-trillions-of-messages); [C38: InfoQ](https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/)).

## 4. The comparison: the cure ladder

| Test | Apple (fleet) | Netflix (stop) | Instagram (transplant) | Discord (exit) |
|---|---|---|---|---|
| The workload | iCloud-class storage | streaming service | activity feed | historical messages |
| The pain | borne quietly (no public ops account) | a managed primitive at volume | GC-bound JVM reads | GC pauses and on-call toil |
| The cure | fleet discipline | bridge, then adopt | engine transplant, same API | protocol-compatible replacement |
| What stayed | Cassandra (watch D1) | Cassandra | Cassandra's model and ops | Cassandra's data model, via ScyllaDB |
| Evidence level | PR | FP/PR | PR | FP |

**Cross-case comparison paragraph.** The four cases share an interface and
differ only in posture, which is the family's operator's-eye lesson: the cure
ladder is ordered by how far the pain reaches. Bear it when the pain is
maturity you can buy with discipline (Apple, whose record remains too thin to
praise concretely). Bridge it when the pain is an engine you never should
have married (Netflix's SimpleDB stop). Transplant it when the pain is inside
the storage engine and the data model is fine (Instagram's Rocksandra, P12's
condition). Replace the implementation but keep the model when the pain is
chronic and operational (Discord's ScyllaDB, the same P12 shape one rung
up). What decides your rung is a diagnostic question, not a market survey:
where, exactly, does the latency graph hurt? The two PR-light cases (Apple,
Instagram) also pace the reading: the family's biggest operators document the
least, so its testimonial weight rests on the ones who left or who fix it.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Apple (C14):** exemplary scale, opaque record. The Apache credits make
  the fleet figures credible but vendor-flattered is not the right frame;
  they are project credits, not audited measurements, and the FoundationDB
  thread (D1) is direction, not outcome. Verdict: the right storage choice
  for its workloads by revealed practice; a case we cannot grade deeply
  because nobody published.
- **Netflix (C15):** correct at every step against its moment. The 2008
  outage made availability the binding driver; SimpleDB was a reasonable
  bridge country that failed exactly where P3 predicts; the honest switch to
  Cassandra turned the family into the backbone. Verdict: the corpus's
  cleanest P3 story, because the stop did its job as a stop.
- **Instagram (C16):** the family's best demonstration that the interface
  can outlive its engine. The transplant targeted the right layer and paid
  forward into Cassandra's own architecture. Verdict: exemplary, with the
  standing caveat that Rocksandra's outcomes are press-reported, not
  first-party.
- **Discord (C38):** the corpus's most complete wide-column arc, and the
  chapter's verdict anchor: the move was right, the migrator for it was the
  real engineering (Chapter 15's subject), and the honest reading is that
  the database was one leg of a three-legged cure. Verdict: exemplary, and
  the anti-drama poster: years of Cassandra investment were preserved, not
  abandoned.

## 6. The pattern

- **P3 - Managed primitive is not a production database.** A transitional
  stop on a managed primitive can be unreliable at volume. Face: C15's
  SimpleDB stop. Fails-when: the stop is used as a destination rather than a
  bridge (no one on the team owns the exit).
- **P9 - The adoption learning loop.** Adopt, break, build simple, evaluate
  honestly, buy the converged engine. Faces: C15's Oracle/SimpleDB/Cassandra
  ladder and C38's Cassandra-to-ScyllaDB arc. Fails-when: the honest
  re-evaluation step is skipped, and one bad era becomes permanent
  infrastructure (Pinterest's counter-case is Chapter 8's).
- **P12 - Engine swap, same protocol.** Replace storage internals, keep
  schema, protocol, and ops: the lowest-blast-radius move in the matrix.
  Faces: C16 (Rocksandra) and C38/T07 (ScyllaDB, full read in Chapter 15).
  Fails-when: the pain is actually in the data model or query layer, which a
  transplant cannot touch.

## Exercises

**●○○ (identify the pattern).** A team on Cassandra sees p99 latency spike
during daily compaction windows; investigation shows GC pressure and
compaction backlog, while the data model and query patterns fit well. Name
the pattern that applies and the two structural options it offers.

*Hint:* ask where the pain sits: model, engine, or discipline.

*Worked answer (assessment):* P12, engine swap, same protocol. Options:
transplant a different storage engine behind the same API (Instagram's
Rocksandra path), or move to a protocol-compatible engine with different
internals (Discord's ScyllaDB path). Rebuilding the application would answer
a problem the team does not have.

**●●○ (apply the framework).** A mid-size service runs an 8-node Cassandra
cluster, mostly idle, moved there years ago "for scale". Pain: weekly GC
firefights and one operator who is the only person who understands the JVM
flags. The team proposes a ScyllaDB migration. Apply the chapter's patterns:
is the pain engine-shaped? What would P12 predict about the migration, and
what should the team measure before choosing any rung of the ladder?

*Worked answer (assessment, judgment):* the fit question comes first: at 8
mostly idle nodes the genuine story may be operational maturity, not engine
speed, so P12's precondition (pain is in the engine, not the model or the
discipline) is not obviously met. Measure: where time goes in the p99 path
(GC vs. IO), whether the data model is being used as designed, and the team's
real operator capacity (P8: the engine your team can operate wins). If the
pain is engine-shaped, ScyllaDB preserves the model and protocol and is
defensible; if it is discipline-shaped, an engine move ships the pain intact.
The honest possible answer is "stay and fix operations", which the chapter's
ladder lists as the first rung.

**●●● (steelman, then weigh).** Write the strongest case against Discord's
ScyllaDB move, then weigh it. (i) Steelman: argue Cassandra's problems were
solvable in place (tuning, version upgrades, newer garbage collectors) and
the team took on an engine migration to avoid an operations problem. (ii)
Weigh: apply P9 and P12, the moving-parts evidence (177→72 nodes, migrator
cost, latency outcomes), and say whether the move was worth it for a
2019-2022 Discord, ending with one transferable rule.

*Steelman and verdict (assessment):* (i) the strongest anti-case leans on
Cassandra 4.0's JVM improvements and on the fact that Discord's own blog
blamed operations, not the model; a well-tuned Cassandra could plausibly have
carried the load. (ii) But P9's honest-evaluation step is exactly what
Discord ran, and the blog reports it continued to operate Cassandra before
deciding; the deciding evidence is operational (repair times, GC toil,
on-call load), which no tuning plan reversibly answers. The transferable
rule: swap engine implementations before swapping data models, when the
model still fits.

**◆ (diagnostic: no hints).** Three teams, one sentence each: name the
family pattern at work or at risk. (i) A team adopted a cloud vendor's
managed key-value primitive two years ago; it now holds 3 TB of hot data and
its latency crawls; leadership wants to make it the permanent architecture.
(ii) A team's Cassandra cluster shows tail latencies traced to garbage
collection; the data model fits; the team is considering an engine swap that
requires no application changes. (iii) A team moved off Cassandra to a
protocol-compatible engine and reports the data move was boring because the
tooling checkpointed everything.

*(Diagnostic answers, assessment: (i) P3 at its fails-when: the managed
primitive has become a destination, and the exit nobody owns is now the
costliest architecture in the company; (ii) P12, correctly preconditioned
(model fits, engine is the complaint); (iii) P12 executed well, with P19
implied: the checkpointed tooling is the playbook, and boring is the point.)*

## Sources

All from the register; URLs copied exactly.

- C14 Apple on Cassandra (DevPro Journal, PR): https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/
- C14 Apple on Cassandra (Dulin Report, PR): https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/
- C14 Apple on Cassandra (Engineers' Codex, PR): https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions
- C15 Netflix on Cassandra (InfoWorld, PR): https://www.infoworld.com/article/2171162/big-movies-big-data-netflix-embraces-nosql-in-the-cloud.html
- C15 Netflix on Cassandra (Netflix slides, FP): https://www.slideshare.net/jasedbrown/cassandra-from-the-trenches-migrating-netflix-10586521
- C16 Instagram Rocksandra (The New Stack, PR): https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/
- C38 Discord (Discord blog, FP): https://discord.com/blog/how-discord-stores-trillions-of-messages
- C38 Discord (InfoQ, PR): https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/
- C36 Twitter Manhattan (blog.x.com, FP): https://blog.x.com/engineering/en_us/a/2014/manhattan-our-real-time-multi-tenant-distributed-database-for-twitter-scale
- C36 Twitter strong consistency (blog.x.com, FP): https://blog.x.com/engineering/en_us/a/2016/strong-consistency-in-manhattan
