# Chapter 1 - Relational: Three Ways to Stay Relational


**Governing idea.** "Scale forces distribution" is a falsified claim. Across the
documented record, teams facing the largest relational workloads picked three
different answers from the same family: split the data, scale the machine up, or
refuse to grow the problem at all. All three kept SQL semantics. The decision
that mattered was never "relational or not" but *where the rows live*, and one
of the three topologies is right for a far larger share of workloads than the
industry reflex assumes (P4, P6, P7, P8).

**Cases in this chapter:** C01 (GitHub), C02 (Stack Overflow), C03 (37signals),
C47 (YouTube/Vitess, the enabling history), C56 (Zerodha) and C61 (OpenAI) as
modern counter-weights, C37 (Uber) as the cautionary twin. Register IDs resolve
in Appendix A; every source URL is copied from `corpus/REGISTER.md`.

---

## 1. Context: the reflex, and the three topologies

The load-bearing sentence in much storage advice is: "we grew, so we had to
shard." It sounds like physics. It is a choice. The public record gives at
least three topologies for the same problem, an online product whose
transactional data is growing:

- **Topology A: split.** Keep the relational engine, divide the data, first by
  domain (vertical), later by rows (horizontal), behind a routing layer.
- **Topology B: scale up.** Buy bigger machines, replicate for reads, and never
  split.
- **Topology C: the monolith.** Design the product so the largest single node
  keeps working; one database, joins included.

Three of the best-documented relational fleets on the internet chose one
topology each ([C01: GitHub partitioning](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/); [C02: Craver 2013](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/), [C02: Craver 2016](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/); [C03: 37signals dev](https://dev.37signals.com/the-rails-delegated-type-pattern/)).

## 2. The choices, each against its time

### Topology A: split (GitHub C01, on Vitess foundations C47)

GitHub began as a Rails monolith on one MySQL database. Fifteen years on, the
fleet is 1,200+ MySQL hosts, 300+ TB, 5.5M queries/second across 50+ clusters
([C01: GitHub partitioning](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/); [C01: MySQL 8.0](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/), FP).

The interesting fact is the *order*: GitHub split by domain first, moving whole
table groups (repositories, users) into separate databases, and only then
split rows for the largest domains, with automated linters blocking any query
that would join across domains ([C01: GitHub partitioning](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/), FP). The tooling underneath is
Vitess, the routing layer YouTube built in 2010 when its "write traffic became
too high for the master database to handle" and application-level sharding
plus replicas stopped being enough ([C47: Vitess history](https://vitess.io/docs/21.0/overview/history/), FP). Vitess co-creator Sugu Sougoumarane described the project's first resharding: "It took an entire night of master downtime, but we survived" ([C47: Percona interview](https://www.percona.com/blog/percona-live-featured-talk-with-sugu-sougoumarane-vitess-the-complete-story/), FP; register amended session 5); the pain that later motivated online resharding (assessment; [C47: PlanetScale](https://planetscale.com/blog/horizontal-sharding-for-mysql-made-easy), VP).

Judged against its time: by 2021, GitHub's choice was conservative. The
horizontal-sharding machinery was 11 years old, battle-tested at YouTube, Slack
and JD.com (35M QPS peak), and free to operate as open source ([C47: Vitess history](https://vitess.io/docs/21.0/overview/history/), FP).

### Topology B: scale up (Stack Overflow, C02)

One of the highest-traffic sites on earth ran on roughly ten servers, with two
SQL Server masters plus replicas: 334M SQL queries and 148M HTTP requests in a
day, average write time "0 milliseconds," SQL boxes with 384 GB RAM and SSDs
kept at 5–10% CPU ([C02: Craver 2013](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/); [C02: Craver 2016](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/), FP). No sharding. Ever.

The workload made it possible: read-mostly (each page hit is one question and a
handful of answers), writes that stay small and hot, and honest caching. But
read-mostly describes huge parts of most consumer sites. What made Stack
Overflow unusual is that the team *measured* instead of following the default
cloud growth path, and spent the avoided scale on engineering attention
instead ([C02: Craver 2013](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/), FP).

Currency caveat (R5): Nick Craver has since moved to Microsoft Azure; the
2013/2016 posts remain the canonical first-party record of the self-hosted
topology and must be cited with their dates. Verdict language below stays tied
to those dates.

### Topology C: the monolith (37signals, C03)

Basecamp and HEY run as "the majestic monolith … It's one app with one big
database, and there's a lot of efficiency in terms of workflow that you get
from just having one database … Stay in the majestic monolith as long as you
can. We've been running this architecture for 10 years" ([C03: 37signals dev](https://dev.37signals.com/the-rails-delegated-type-pattern/), FP).

No topology example should be judged by company fame, and this one deliberately
optimizes for a different metric: iteration speed of a small team. Joins across
one database are cheap, not cross-service calls; schema changes are one
migration, not ten; and there is no consistency boundary to police, because
there is only one database. This is deliberate restraint rather than neglect,
and 37signals itself proved the discipline is conscious: when its Fizzy product
attempted SQLite-per-customer tenancy in 2026, it reverted to a single MySQL
database before launch after hitting global-replication edge cases (C53,
recorded for the P18 fails-when discussion, whose chapter home is a logged
planning gap from session 5).

## 3. What happened (sourced outcomes, with evidence levels)

- **GitHub (C01, FP).** Domains split; row-level sharding applied where needed;
  an entire fleet upgraded to MySQL 8.0 under SLOs, done incrementally the same
  way ([C01: MySQL 8.0](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/), FP). The relational semantics survived all of it: GitHub's engineers treat
  the database layer as a fleet to be maintained, not a platform to be escaped.
- **Stack Overflow (C02, FP).** The measured outcome: sub-millisecond average
  writes, 5–10% CPU headroom on the masters, at day-peak volumes of 334M SQL
  queries ([C02: Craver 2013](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/), FP). The topology carried the site for over a decade.
- **37signals (C03, FP).** One database, over ten years of operation, with the
  small team's throughput as the product of the simplicity ([C03: 37signals dev](https://dev.37signals.com/the-rails-delegated-type-pattern/), FP).
- **YouTube/Vitess (C47, FP).** The enabling move became an industry utility:
  CNCF project, adopted at Slack, GitHub, Square, JD.com (35M QPS) ([C47: Vitess history](https://vitess.io/docs/21.0/overview/history/); [C47: PlanetScale](https://planetscale.com/blog/horizontal-sharding-for-mysql-made-easy), FP/VP).

### The modern counter-weights (2023–2026)

Two recent first-party cases push the same lesson into the 2020s, one from
each side of the read/write line.

- **Zerodha (C56, FP).** India's largest stock broker runs self-hosted
  PostgreSQL on bare EC2 ("not moving out of PostgreSQL") at ~trillion-row
  scale, with a dedicated Postgres hot-cache instance serving reports via
  ~7 million ephemeral result tables per day (DungBeetle, open-sourced); when
  sharding became necessary for one 20 TB database it was done "in two hours
  with zero schema changes" ([C56: Zerodha tech](https://zerodha.tech/blog/working-with-postgresql/); [C56: DungBeetle](https://github.com/zerodha/dungbeetle); [C56: FOSS United talk](https://platform.fossunited.org/c/indiafoss/indiafoss24/cfp/oqtefvp0pj), FP). Scale-up and mastery, taken further than Stack Overflow took them, at a fintech whose write peak is the trading day.
- **OpenAI (C61, FP).** ChatGPT reached 800M users on **unsharded** PostgreSQL:
  one Azure PostgreSQL Flexible Server primary plus nearly 50 read replicas
  across multiple regions, millions of QPS. Reads scaled by replication, not
  sharding; primary load was minimized because a single writer cannot scale
  writes. One SEV-0 PostgreSQL incident in 12 months: the ImageGen launch,
  a >10× write surge and 100M new users in a week. The honest boundary is
  first-party too: remaining write-heavy workloads are being migrated to
  sharded systems, and third-party coverage of the post reports a
  "no new tables in PostgreSQL" rule ([C61: OpenAI](https://openai.com/index/scaling-postgresql/), FP; [C61: ByteByteGo](https://blog.bytebytego.com/p/how-openai-scaled-to-800-million), [C61: VentureBeat](https://venturebeat.com/data/how-openai-is-scaling-the-postgresql-database-to-800-million-users), PR, watch-list W5).

These two cases are the reason Chapter 1 exists as a comparison rather than a
sharding manual. Zerodha's "reads, caches, and one node" and OpenAI's "replicas
against 800M users" bracket what unsharded PostgreSQL really buys in 2026, and
OpenAI names its own fail line: writes.

### The cautionary twin: Uber (C37)

The family's worst public outcome is not a sharded disaster. It is a *narrative*
disaster. In 2015, Uber moved from Postgres 9.2 to MySQL with a home-grown proxy
called Schemaless, publishing Postgres-specific claims: page-level MVCC causing
write amplification between replicas, replication-layer demands, DDL locking
behavior ([C37: Uber Eng](https://www.uber.com/us/en/blog/postgres-to-mysql-migration/), FP). The independent technical response, most prominently Ayende Rahien's,
re-derived the same workloads and argued the casualties were operational
inexperience and Postgres tuning, not Postgres itself ([C37: Ayende](https://ayende.com/blog/175137/re-why-uber-engineering-switched-from-postgres-to-mysql), SR).

Two afterlives matter for this chapter. First, "PG doesn't scale" became
industry folklore it did not earn; every later relational decision in this book
had to be made against that background hum. Second, Uber's own arc continued:
Schemaless evolved into Docstore, "a general-purpose transactional database"
with strict serializability at the partition level (C48, T22; [C37: Schemaless→Docstore](https://www.uber.com/in/en/blog/schemaless-sql-database/), FP): 
which is to say Uber eventually rebuilt SQL characteristics on top of its
sharded escape. The family's strongest critic eventually wrote the strongest
argument for the family.

## 4. The comparison: the conditions under which each topology works

| Condition | A: split (GitHub) | B: scale up (Stack Overflow, Zerodha) | C: monolith (37signals) |
|---|---|---|---|
| Write rate | Highest; the only topology that absorbs write growth past one node | Low to moderate; bounded by one primary's write capacity | Low; the product design keeps it there |
| Read rate | Solved by replicas per shard | Solved by replicas; the leverage point (C61) | Solved by caching |
| Natural tenant key exists? | Required eventually; chosen deliberately (P4) | Not needed | Not needed |
| Joins across domains survive? | Blocked by linters; app must reassemble (P6) | Unchanged, all joins available | Unchanged, all joins available |
| Team that can operate it | A platform team's worth | Deep ops mastery, small team (P8) | Any Rails-class team (P8) |
| Reversibility | Highest: splits are staged and incremental (P7, P19) | Not applicable | Not applicable |

Reading the table by row, the real decision procedure falls out:

1. **Is the workload write-bound?** If yes, no amount of replicas will help;
   the single-writer limit is first-party physics (C61, W5). Plan the split,
   and choose the key before you think you need it (P4).
2. **If read-bound:** replication plus caches covers 10M and 800M users alike
   (C02, C56, C61). The variable is not scale but ops depth (P8): either a team
   with Craver-class measurement discipline or a workload benign enough not to
   demand it.
3. **Either way, vertical before horizontal** (P6): GitHub's domain splits were
   the low-risk, reversible step that bought years before row splitting, and
   the same sequencing reappears at Figma in 2020–22 (C54, held for Chapter 13).
4. **Split along a real-world boundary** (P4: domain, shop, workspace, channel),
   keep the units movable (P7: Shopify's whole-shop moves; Notion's divisible
   480-shard fleet), and never let "the tenant is the shard" pass without
   checking the hot unit (P5: Slack's workspace-per-shard, held for Chapter 13).

**Cross-case comparison paragraph.** The three topologies are the same thesis
at different write rates: SQL semantics are worth defending everywhere, only
the defense differs. GitHub defended them with a routing layer and a decade of
incremental surgery; Stack Overflow and Zerodha defended them with measurement,
headroom and caches; 37signals defended them by refusing the growth that would
threaten them. The outlier is Uber, which abandoned the family, then spent
years rebuilding what it left. The corpus contains no case of a team that chose
its topology from this decision procedure and reported regret; the documented
regrets (F10, C37's folklore) all came from key choice or narrative, not from
staying relational.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **GitHub (C01):** exemplary sequencing, and the most imitable case in the
  book. Domain-first splits, linter-enforced seams, fleet upgrades done
  incrementally under SLO. Could they have done better? Nothing evident at this
  distance; the debate is whether vertical sharding without Vitess-class
  routing would have sufficed longer, and their own accounts suggest the
  routing layer paid for itself.
- **Stack Overflow (C02):** the counter-example that works, with boundary
  conditions stated by its own author: read-mostly load, exceptional
  measurement culture, and a business model that rewards hardware efficiency.
  Could they have done better? By their own analysis, cloud/distributed storage
  would have cost more for no user-visible gain. The Azure move changes the
  hosting story, not the 2013–2016 record.
- **37signals (C03):** deliberate restraint, honestly labeled as strategy.
  Could they have done better? Their Fizzy episode (C53) shows the restraint is
  active, not nostalgic: they tried a fancier tenancy and walked it back before
  launch.
- **Zerodha (C56):** Stack Overflow's kin at fintech scale; the two-hour sharding
  of one database is the maturity dividend of mastery plus PostgreSQL tooling.
  Watch DungBeetle: if the ephemeral-table pattern converges into a product, it
  is the P15 test applied to a cache pattern (assessment).
- **OpenAI (C61):** the strongest modern "you may not need to shard yet"
  datapoint, and the only one that names the wall (writes). The "no new tables"
  third-party-reported rule, if accurate, is a frontier management tactic worth
  watching via W5, not a documented first-party policy (assessment).
- **Uber (C37):** the least defensible *publishing* decision, not necessarily
  the least defensible *engineering* decision. Migrating off Postgres 9.2 for
  operational reasons the team knew well was defensible; publishing
  engine-mechanics claims that independent re-derivation contested, and that
  ossified into "PG doesn't scale," set the industry back years (assessment).

Could any of them have done better in the largest sense? The comparison says:
only in choosing the key earlier (P4/P5) and in what they told the rest of us.
No case in this chapter argues for leaving the family.

## 6. The pattern

- **P4, Natural tenant key sharding.** Keep SQL; split along a real-world
  boundary. Faces here: GitHub's domains ("users," "repositories"). Fails when
  no natural tenant exists, or the key is chosen late (F10).
- **P6, Vertical before horizontal.** Split table groups before splitting
  rows. Face: C01's ordered split. Fails when the app still needs cross-domain
  joins, which is why the linter gate is part of the decision and not an
  implementation detail.
- **P7, Movable tenants beat rehashing.** Make the tenant relocatable. Faces
  here are forward-references to Chapter 13 (Shopify's shop moves, Notion's
  divisible shard count); Chapter 1 plants the idea because it explains *why*
  GitHub's domain splits stayed reversible (P19/P20 tie-in).
- **P8, Team size is an architectural input.** The engine your team can
  operate wins. Faces: C02, C03, C56 all chose the topology their teams could
  run brilliantly. Fails when skill is used to excuse avoidable learning, the
  conflation Chapter 20 returns to via F11 and Yelp's honest mirror (C62).

## Exercises

**●○○ (identify the pattern).** A B2B SaaS with 40,000 company tenants runs one
Postgres primary; its fastest-growing table is an event log written by an
ingestion service at 20,000 rows/second, while its human-facing read traffic is
modest. The team proposes read replicas. Name the chapter's decision step this
skips, and the pattern that applies.

*Hint:* which row of the comparison table decides this?

*Worked answer (assessment):* the write row. Replicas only scale reads; a
20k rows/s ingestion primary is write-bound, so the single-writer limit binds
(C61's stated wall, W5). The applicable pattern is P4 in its forward-looking
form: split the write-heavy data along its real-world boundary (the tenant, or
better, the event stream) *before* the load forces a rushed key choice.

**●●○ (apply the framework).** You are the staff engineer at a 25-engineer
company running a 400 GB Postgres database for a project-management product.
Traffic: 95% human, 5% API. Growth: 1.5× data per year, reads growing faster
than writes after a new offline client shipped. Your options this quarter are
(a) two read replicas and a cache review, (b) vertical split of the document
storage domain into its own database, (c) horizontal sharding by workspace.
You may pick at most one this quarter. Use the chapter's decision procedure to
argue for one, and name each pattern you relied on.

*Worked answer (assessment, stated as judgment):* (a) first, with (b) as the
pre-decided next step. The workload is read-heavy, so replication attacks the
actual growth (C02, C56, C61); (c) spends P5 risk on a 400 GB fleet no shard
has yet needed, and sharding keys are irreversible decisions made cheaply only
before they are needed; (b) is correct sequencing (P6) but buys little while
the whole database is one node's size. The discipline is to document the
intended key now (workspace, plausibly, if workspaces are small and numerous)
so that P4's key is chosen deliberately rather than under duress, and to keep
the split reversible per P7/P20.

**●●● (critique a case against its critics).** The Uber 2015 migration (C37)
claimed Postgres-specific write amplification due to page-level MVCC. Ayende's
re-derivation argued the evidence showed operational issues instead. Write a
200–400 word critique that (i) states which claims are contested by this
chapter's record, (ii) uses C48 (Docstore's strict serializability) as the
internal evidence Uber itself later supplied, and (iii) takes a position on
whether the migration or the *account* of it caused more industry harm. Support
(i)–(ii) from the cited sources; (iii) is a Verdict (assessment).

*Suggested answer sketch (assessment):* the contested part is the generalization
("PG doesn't scale"), not the anecdote (their replication lag was real); C48 is
Uber re-entering SQL semantics territory, which undercuts the engine-fault
narrative; the account did more harm, because 400+ words of blog post echoed
for a decade in decisions the blog's own authors did not have to live with.

**◆ (diagnostic: no hints).** Three teams describe their moment. For each,
decide the topology and the one pattern whose fails-when they are closest to:
(i) a wiki company, 2 TB database, 60/40 read/write, single primary at 40% CPU;
(ii) a marketplace, 30 TB, clear shop-level isolation in the data model, three
hot tenants growing off-scale relative to the rest; (iii) a budgeting app, one
node, 12 engineers, write rate flat for two years, roadmap entering
cross-entity reporting. Answer in one sentence each; there is no credit for
tool names.

*(Diagnostic answers, assessment: (i) scale-up, P6 isn't violated yet but the
write share makes P4's "natural tenant" check the one to run before growth
picks; (ii) split, with P5's fails-when live, "the tenant is the shard" fails
when hot tenants exist, so plan movers or finer keys (P7); (iii) monolith, and
the P6 fails-when to respect is the reporting need: vertical splits must not
break joins the reports rely on.)*

## Sources

All from the register; URLs copied exactly.

- C01 GitHub partitioning: https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/
- C01 GitHub MySQL 8.0: https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/
- C02 Craver 2013: https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/
- C02 Craver 2016: https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/
- C02 Craver bio (R5 currency caveat): https://nickcraver.com/
- C03 37signals: https://dev.37signals.com/the-rails-delegated-type-pattern/
- C37 Uber: https://www.uber.com/us/en/blog/postgres-to-mysql-migration/
- C37 Ayende critique (SR): https://ayende.com/blog/175137/re-why-uber-engineering-switched-from-postgres-to-mysql
- C37/C48 Schemaless to Docstore: https://www.uber.com/in/en/blog/schemaless-sql-database/
- C47 Vitess history: https://vitess.io/docs/21.0/overview/history/
- C47 PlanetScale (VP): https://planetscale.com/blog/horizontal-sharding-for-mysql-made-easy
- C47 Percona interview, Sugu Sougoumarane (session 5 register amendment; first-resharding quote): https://www.percona.com/blog/percona-live-featured-talk-with-sugu-sougoumarane-vitess-the-complete-story/
- C56 Zerodha tech: https://zerodha.tech/blog/working-with-postgresql/
- C56 FOSS United talk abstract: https://platform.fossunited.org/c/indiafoss/indiafoss24/cfp/oqtefvp0pj
- C56 DungBeetle: https://github.com/zerodha/dungbeetle
- C61 OpenAI: https://openai.com/index/scaling-postgresql/
- C61 ByteByteGo (PR): https://blog.bytebytego.com/p/how-openai-scaled-to-800-million
- C61 VentureBeat (PR): https://venturebeat.com/data/how-openai-is-scaling-the-postgresql-database-to-800-million-users
