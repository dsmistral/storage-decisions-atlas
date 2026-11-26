# Chapter 13 - Scale-Out Relational Done Right

**Governing idea.** Chapter 12 read the exits, the migrations that treated
relational as a thing to leave. This chapter reads the counter-program: the
fleets that heard the same verdict and kept the database anyway. Six
first-party cases, all relational, all still relational today: GitHub, with
1,200+ MySQL hosts holding 300+ TB and serving 5.5 million queries per second
across 50+ clusters (C01); Shopify, sharded by shop since the mid-2010s, with
whole shops moving between shards online (C04); Instagram, sharding Postgres
with three engineers in 2011 by encoding the shard into the ID itself (C05);
Pinterest, a sharded-MySQL fleet operator whose 2022 re-evaluation is the
precursor to Chapter 14's distributed-SQL story (C26); Figma, which spent
2022 on vertical partitioning and then nine months horizontally sharding
Postgres in place (C54); Notion, which sharded a monolith into 480 logical
shards and then re-sharded the fleet without a second migration (C55). The P-index rows behind the register's successful migrations agree on
four properties: a
natural tenant key (P4), vertical partitioning before horizontal (P6),
tenants that can keep moving after the big split (P7), and verification
tooling budgeted as part of the move (P19's playbook, previewed here and
taught in full in Chapter 22). One famous failure shows what happens when
the first property is missed: Slack's workspace-per-shard assumption, which
its own former chief architect later called "the original sin of early Slack
infrastructure" (F10, P5).

## 1. The counter-program

The pioneering exits of Chapter 12 were made under pressures this chapter's
fleets never faced in the same form: Amazon's ops bill ran across nearly
7,500 Oracle databases; Netflix's single Oracle box failed hard enough to
stop shipping DVDs. But the industry lesson drawn from those exits was often
blunter than the exits themselves: relational does not scale. The six cases
here are the falsification of that blunter lesson. None of them left the
relational family. All of them changed the topology instead, and the
fascinating part is how much agreement there is in how they did it.

Two of the six were also read in earlier chapters, and the readings there
were deliberately different. Chapter 1 used C01 (alongside C47, the Vitess
build history) as topology evidence: the same database family kept, under
three different shapes. Chapter 8 used C26 as one of its 2022-era NewSQL
buyers. This chapter reads the same cases as migration programs: not what
the final shape was, but how the fleet got there with the business running
on it. The distinction matters
because the register's successful migrations do not succeed at the schema
level; they succeed at the sequencing level, and the sequencing is what
this chapter teaches.

One figure discipline note before the fleet table. C01's register row
compresses two posts into one summary line, and the same month carried
different numbers for different questions. The MySQL 8.0 upgrade post states
the fleet total: "We store 300+ TB of data and serve 5.5 million queries per
second across 50+ database clusters". The partitioning post states an
average for the clusters it measured: "All hosts of these clusters combined
answer 1,200,000 queries/s on average (1,125,000 queries/s on replicas,
75,000 queries/s on the primaries)". Both are first-party; neither
contradicts the other; quoting either number alone would invite exactly the
wrong comparison.

## 2. Six fleets, side by side

| Case | Era | Scale at decision | Topology move |
|---|---|---|---|
| C01 GitHub | 2021-23 | 1,200+ MySQL hosts, 300+ TB, 5.5M queries/s, 50+ clusters (FP) | Schema domains, vertical shards, then row-level sharding via Vitess |
| C04 Shopify | since ~2014 | MySQL sharded by shop, pods isolate MySQL+Redis+Memcached (FP) | Whole-shop moves between shards, online |
| C05 Instagram | 2011 | Thousands of logical shards as Postgres schemas; 25 photos/s at decision time (FP) | Shard key embedded in the 64-bit ID |
| C26 Pinterest (precursor) | 2022 evaluation (fleet sharded earlier) | Sharded MySQL fleet; deprecated HBase (FP) | App-managed ID-to-shard mapping, later re-evaluated |
| C54 Figma | 2022-24 | Stack grown almost 100x since 2020 (FP) | Vertical partitions, then nine-month horizontal sharding in place |
| C55 Notion | 2021, re-shard later | Monolithic Postgres; then 480 logical shards on 32 physical databases (FP) | Workspace as shard key; re-shard as a redirect |

Every row is first-party evidence (FP). Two of the six sources are
Medium-hosted engineering posts (C05, C26); the checker's fetches return a
near-empty page for Medium, so their anchors are corroborated at summary
level from the register rows and, where marked, from search-snippet text,
following Chapter 8's convention for C26's introduction there. The other
posts (both GitHub posts, Shopify, both Figma posts, both Notion posts, and
the HN thread) all fetched live while writing this chapter.

## 3. The four properties, with figures

**Property 1: the shard key must be natural to the workload (P4).**
ANALYSIS's pattern table lists P4's faces as C01 (domain), C04 (shop_id), C05
(user id in ID), and C26 (app-managed ID-to-shard), and the R6 verification
pass added C55 to that list as its Postgres-at-product-scale member. The
range of what counts as natural is worth reading closely, because it is
wider than "pick a customer column". Shopify's key is the shop: "each pod
consists of its own shard", and "We can attach a `shop_id` to all
shop-owned tables and have it serve as a sharding key". Notion's is the
workspace, and the post's own section heading states it as a decision:
"Partition block data by workspace ID". Instagram's is the most aggressive
version: the key is not stored in a column but encoded into the 64-bit ID
itself, generated per shard inside Postgres (the register records the
PL/pgSQL generation; the Medium account is search-snippet-corroborated in
this chapter, with the post's opening sentence, "With more than 25 photos
and 90 likes every second", marking the scale at which the design was
chosen). Notion's post
also records the road not taken, which is this chapter's family boundary in
one line: "switching to another database system such as DynamoDB (deemed
too risky for our use case)". Pinterest's
app-managed ID-to-shard mapping is the same idea maintained at a fleet that
later outgrew its own shards, which is exactly why it anchors Chapter 14.
ANALYSIS's fails-when for P4 points at what all of these avoided: the
tenant exists but was chosen coarse.

**Property 2: vertical partitioning before horizontal (P6).** ANALYSIS's
pattern table records P6's face as C01: "domain table moves before row
splitting; linters prevent cross-domain joins". GitHub's own words are the
mechanism: "we group
database tables that belong together into schema domains and enforce
boundaries between the domains with SQL linters. This allows us to safely
partition data later without ending up with queries and transactions that
span partitions." The linters are the point. They encode the partition
boundary as a reviewable, always-on check, so the later splitting inherits
boundaries the codebase has already learned to respect. GitHub then moved
the domains with Vitess: "We use its vertical sharding feature to move sets
of tables together in production without downtime. To do that, we deploy
Vitess' VTGate in our Kubernetes clusters."

The R6 verification pass added the second first-party face, and Figma's
post states the sequencing as a running total: "In 2020, we were running a
single Postgres database hosted on AWS's largest physical instance, and by
the end of 2022, we had built out a distributed architecture with caching,
read replicas, and a dozen vertically partitioned databases." The same post
closes the sequence: "Figma's database stack has grown almost 100x since
2020", with the nine-month horizontal sharding project as the payoff: "Our
nine month journey to horizontally shard Figma's Postgres stack, and the
key to unlocking (nearly) infinite scalability." Vertical first was not a
fallback at Figma; it was the explicit 2020-22 stepping stone that kept the
horizontal project small enough to finish in nine months.

The trade the key choice buys is stated plainly in the same post: "we chose
not to support atomic cross-shard transactions because we could work around
cross-shard transaction failures", and "We picked a colocation strategy that
minimized the changes required at the application layer." Figma's logical
shards live as Postgres views grouped per key (the register's summary row
calls the groupings "colos"), so the colocation the application already had
is carried onto the router rather than rebuilt.

**Property 3: tenants must keep moving after the split (P7).** The
P-index's pattern row records P7's face as C04, and the post's own framing
is the property: Shopify balances
"our MySQL shards by migrating shops across shards", with the moves
"entirely online and with virtually zero consumer-facing downtime". The pod
architecture is what makes
the moves routine: a pod carries its own MySQL, Redis, and Memcached, and
the shop is the identifying entity, so moving one is a bounded data move,
not a re-schema. Notion's re-shard is the macro version of the same
property: once the workspace became the key, capacity planning became a
server count, and the fleet later went from 32 machines by "tripling the
number of instances in our fleet from 32 to 96 machines", with traffic
migrating slowly and verifiably, reusing the original logical-shard
structure rather than paying for a second migration.

**Property 4: verification tooling is part of the move (P19, previewed).** The
register's P19 faces already include C01's three-year double-write and
read-diffing program at Slack; the R6 pass added two more from this
chapter's own cases. Notion: "Before migrating read queries, we added a
flag to fetch data from both the old and new databases (known as dark
reading). We compared these records and discarded the sharded copy, logging
discrepancies in the process." And then the post's most quotable line of
discipline: "As a precaution, the migration and verification logic were
implemented by different people. Otherwise, there was a greater chance of
someone making the same error in both stages, weakening the premise of
verification." Figma automated the same instinct: "We added a 'shadow
application readiness' framework capable of predicting how live production
traffic would behave under different potential sharding keys." And the
property has a peacetime form: once the move was done, Figma kept the
guardrails. "Since the full rollout, Figma's database SLO, measured against
core user experience availability, has held at above 99.99%. In Q4 2025
alone, PGKeeper prevented more than 20 incidents that would otherwise have
caused user-visible outages."

## 4. The seductive wrong key

P5's registered face is F10, and ANALYSIS's pattern table quotes it from
Slack's own people: the "original sin of early Slack infrastructure". The
full sentence, written on the migration thread by Keith Adams, posting as
HN user "kmavm" on May 18, 2021, who states the role himself ("I was
Chief Architect at Slack from 2016 to 2020"), is the cleanest statement
of the trap in the corpus: "The
assumption that tenants are perfectly isolated is actually the original sin
of early Slack infrastructure that we adopted Vitess to migrate away from."

The wrong key was seductive because it was not wrong by the product's
lights. A workspace is the tenant as the product and the org chart both see
it, so workspace-per-shard looked like a natural tenant key. The failure was
granularity: the unit of isolation was coarser than the unit of load, and
nothing in the early design made that mismatch visible until Enterprise
features began joining lots of "little Slacks" into a corporate-wide entity
and traffic concentrated unevenly inside big workspaces. The correction,
re-sharding by channel on Vitess, took years and is the register-recorded
P19 face at Slack (a three-year double-write and read-diffing program;
C66/F10; the failure chapters return to it in Chapters 18
and 20 with the fuller mechanics). The point for this chapter's four
properties: ANALYSIS's fails-when for P4, "the tenant exists but chosen
coarse", is not an edge case. It is the default mistake of a team that lets the product
taxonomy pick the shard key.

## 5. Reading the moves with the patterns

All five patterns named in this chapter register faces in the P-index; what
varies is where the fails-when evidence comes from. These are patterns whose
faces are first-party successes, with one fails-when carried by a failure
post-mortem instead.

- **P4 (natural tenant key)**: faces C01 (domain), C04 (shop_id), C05 (user
id in ID), C26 (app-managed ID-to-shard), plus the R6 addition C55
(workspace ID). ANALYSIS's fails-when for P4 (the tenant chosen coarse)
is taught not by a P4 case but by F10 below.
- **P5 (seductive wrong key)**: face F10, the workspace-per-shard assumption.
ANALYSIS's fails-when for P5 is itself the failure: "F10 + Slack's own
3-year correction (P19 discipline applied)". A pattern whose teaching
evidence is a decade of drag on an engineering organization.
- **P6 (vertical first)**: faces C01 (schema domains + Vitess vertical
shards) and C54 (the dozen vertically partitioned databases built 2020-22
before the nine-month horizontal project). No fails-when registered; the
discipline reading is that it is a sequencing rule, and its breach is
invisible until the horizontal project has to absorb it.
- **P7 (movable tenants)**: face C04, whole-shop moves "entirely online and
with virtually zero consumer-facing downtime". No fails-when registered.
- **P19 (migration playbook)**: the chapter's verification property. This is
a forward pin, not a full treatment: C55's dark reads and its
implemented-by-different-people verification, plus C54's
shadow-application-readiness framework, appear here as evidence that
verification is a first-class budget line; the playbook itself is Chapter
22's material.

One caution from the corpus's evidence ladder applies to the whole chapter:
these are the successes. Survivorship cuts the same way it did in Chapter
11's matrix. The fleets that shard by the wrong key and quietly stall do not
publish posts; F10 only became public because Slack both failed and
recovered loudly, and then sent its chief architect to talk about it.

## 6. Spec state and what the chapter hands forward

Spec state: the chapter map's line for this chapter, "Ch. 13 Scale-out
relational done right - C01, C04, C05, C26-precursor (P4-P7) (+ C54 Figma,
C55 Notion, R6)", governs per the repo's conflict rules. BOOK-PLAN's Ch. 13
inputs cell reads "C01 (Slack/GitHub), C04, C05, C47" with patterns "P4, P5,
P6, P7, P19" and is stale against the map in two respects: it names C47
(Vitess's build history is Chapter 1's material; the Vitess technology
appears here only as C01's and C66's chosen tool), and it predates the R6
additions of C54 and C55. The pattern cells agree except for P19, which the
chapter carries as the Property 4 preview and a forward pin rather than a
registered family pattern. The same flag convention as Chapters 6 and 11
applies: recorded here, to be folded into the next planning pass. One more
divergence for the same pass: BOOK-PLAN's Ch. 18 row schedules F10 (with
patterns "FC2, P5") while the chapter map's Ch. 18 line lists only F01 and
F03, F10 appearing there only in Ch. 20; the chapter's F10 pin follows the
plan.

What the chapter hands forward: to Chapter 14, the observation that some of
these fleets are precisely the ones that later re-evaluated the market
(Pinterest's 2022 evaluation of 10+ candidates is C26; its TiDB adoption is
Chapter 14's and T04's material, and its Vitess and Spanner assessment
quotes, register-recorded, are the honest rejection record). To Chapter 15,
the reminder that sharding is not the only scale move: the engine-swap
twins kept a fixed topology and changed the storage engine under it. To
Chapter 22, the playbook: this chapter's Property 4 cases are its
verification face. And to Chapter 21, the driver synthesis: none of the six
register rows records a raw "the database is slow" driver; the recorded
drivers are load and ops burden, and the choice to stay relational shows
cost and skill doing the deciding.

Verdict (assessment): the counter-program is the more instructive half of
the relational story. The exits of Chapter 12 prove leaving is possible;
this chapter proves staying is a designed, repeatable discipline, and the
difference between the fleets that stay well and the ones that stall is
almost entirely in the key choice and the verification tooling, not in the
database brand.

## Exercises

**●○○ (recall + identify).** Six cases, one signature each: name the shard
key for C04, C05, C55, and C01's first phase, and say which of the four
encoded the key into the ID itself rather than storing it in a column.
(Answers: shop_id; the 64-bit ID with shard bits; the workspace ID; schema
domains; C05.)

**●●○ (worked example).** Notion chose 480 logical shards across 32 physical
databases, 15 schemas per database. Using the register's P13/P7 R6 row,
explain what the divisibility bought them at the 32-to-96 re-shard, and what
a 480-shard plan would have looked like if the fleet had grown to 40 or 48
hosts instead. (The worked answer: divisibility kept every re-shard a
redistribution at the logical-to-physical map level, a PgBouncer-layer
redirection plus slow verified traffic migration, not a second key
migration; 40 and 48 both divide 480 evenly, which is why the plan
survives capacity growth in either direction.)

**●●● (design).** You inherit a workspace-product with a workspace-per-shard
design and its failures: hot workspaces, and a new Enterprise feature that
must join workspaces. Using F10's mechanism and C01's linter discipline,
design the escape in three sequenced steps, and say which of the register's
P19 steps you would refuse to skip even under deadline. (Expectation: a
verification-first plan; the sequencing rule beats the tooling rule, and
skipping the diff tooling is the one skipped step the corpus links to every
failure class.)

**◆ (pattern diagnostic, no hints).** A team sharded by a natural key with
the app's own ID service; two years later, every capacity fix is a
migration, because single shards cannot absorb their own growth and moving
a tenant means a measured outage window. Which pattern's discipline was
skipped, and which registered face shows the property done right? (The
answer is P7: C04.)

## Sources

1. [GitHub: partitioning](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/) (C01)
2. [GitHub: MySQL 8.0](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/) (C01)
3. [Shopify Eng](https://shopify.engineering/mysql-database-shard-balancing-terabyte-scale) (C04)
4. [Instagram Eng](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c) (C05, dead fetch at the platform cap; corroborated via search snippets)
5. [Pinterest sharding](https://medium.com/pinterest-engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f) (C26, dead fetch at the platform cap; corroborated via search snippets)
6. [Figma blog](https://www.figma.com/blog/how-figmas-databases-team-lived-to-tell-the-scale/) (C54)
7. [Figma: PGKeeper](https://www.figma.com/blog/pgkeeper-building-the-bouncer-we-needed-for-postgres/) (C54)
8. [Notion: herding elephants](https://www.notion.com/blog/sharding-postgres-at-notion) (C55)
9. [Notion: the great re-shard](https://www.notion.com/blog/the-great-re-shard) (C55)
10. [HN comment](https://news.ycombinator.com/item?id=27199229) (F10/P5)
