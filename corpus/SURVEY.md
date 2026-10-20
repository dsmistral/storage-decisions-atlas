
# Choosing and switching data stores: 22 case studies, the transition matrix, and the failures

## Question

How have well-known companies chosen, implemented, and transitioned between data storage schemes? What motivated each choice, how did each transition actually go, and — with hindsight — could they have done better, why, and how? The goal is a source-backed dossier series that a reader can use to inform their own storage decisions.

## Executive summary

1. **Nobody migrates for technology's sake.** Behind nearly every one of the 20 cases sits one of five real drivers: availability requirements, operational burden, cost at scale, transactional guarantees, or team skill. Technology choice is downstream of these.
2. **The 2010s "NoSQL exodus" is being quietly reversed.** Netflix, Twitter, Pinterest and Discord each discovered that operating eventually consistent stores at scale was harder than expected; Twitter built Manhattan partly to *add back* strong consistency, Pinterest moved to NewSQL (TiDB), and Discord's fix was only one-third "a new database" — the rest was keeping load off the database.
3. **Sharded relational with a natural tenant key is the most reproducible success pattern.** YouTube (Vitess), Slack, GitHub, Shopify (pods), Instagram and Pinterest all kept SQL semantics and scaled by splitting along a real-world boundary (user, workspace, shop, domain). The classic failure inside this pattern is choosing the wrong key — Slack called its original workspace-per-shard design its "original sin."
4. **"Build your own" was right only at extreme scale or extreme specialization** — Amazon (Dynamo), Google (Spanner/F1), Facebook (MyRocks), Dropbox (Magic Pocket) — and each of those teams had research-grade engineering depth. Everyone else who built their own general-purpose store (LinkedIn's Espresso) built what converged engines now sell as products.
5. **The transitions that went well shared four properties:** dual writes with reconciliation, incremental cutovers per tenant or shard, rollback preserved for the entire window, and bespoke tooling that made the backfill fast (Discord cut three months to nine days).
6. **"Do nothing" is sometimes the best answer.** Stack Overflow serves one of the world's biggest sites from a couple of SQL Server boxes; 37signals has run Basecamp on one big database for over a decade. Both outperform fashion on cost per request.
7. **My hindsight verdicts:** most teams chose correctly *for the tools of their time*. The clearest "could have done better" is Uber's Postgres-to-MySQL move — critics and later Postgres releases suggest operational inexperience, not the engine, was the binding constraint. The clearest exemplary migrations are Discord (Cassandra→ScyllaDB), Slack (→Vitess), and Facebook (InnoDB→MyRocks).
8. **Every database family now has two or three flagship production cases, compared** — Cassandra at Apple and Netflix (plus Instagram's engine swap), ClickHouse at Cloudflare and Bloomberg, Neo4j behind the Panama Papers, eBay ShopBot and Zurich Insurance, Uber's M3 and Netflix's Atlas for time-series, Netflix's EVCache and Facebook's Memcache for caching, DoorDash's CockroachDB (with Pinterest and Google), and turbopuffer and Spotify Voyager for the vector generation. The family-by-family comparison shows the *conditions* under which each family works, not just that it can.
9. **The transition matrix is dense in both directions** — nearly every family-to-family move now has a documented case (matrix below). The one move almost absent from first-party sources: the quiet return *from* dedicated document stores to relational engines.
10. **Failures teach more than migrations.** The post-mortem set — Foursquare's unwatched shard, GitLab's silently failing backups, GitHub's 43-second partition, Sidekick's and Ma.gnolia's missing backups, Code Spaces' shared blast radius, S3's typo'd runbook, Cloudflare's migration regression, and a two-week TimescaleDB round trip — shows that storage failures are overwhelmingly *operations* failures: backups, failover, monitoring, and verification, not engine selection.

## Methodology

Searches were run in three targeted rounds: (1) pioneering exits from relational/monolithic stacks, (2) NoSQL adoption and regret, (3) scale-out relational, build-your-own, deliberate-simplicity, and edge cases. Primary sources were preferred: engineering blogs, SOSP/VLDB papers, official docs, and migration announcements; secondary press (InfoWorld, InfoQ, CIO Dive) was used for context and figures the primaries didn't state. Claims below cite the source that carries them. Limitations: engineering blogs are self-reported and partly marketing; internal figures (cost, downtime) are unaudited; several architectures have evolved since publication, so each dossier is dated. Where I judge a decision with hindsight, it is labelled **Verdict (assessment)** — that is inference from evidence, not sourced fact.

## How to read these cases

- **Survivorship bias**: nobody blogs "we migrated and it was fine and boring" — the loud cases are the extremes. Stack Overflow and 37signals matter precisely because they are the silent majority done well.
- **Availability rhetoric vs. reality**: "we needed five nines" is often really "our ops team was drowning" (Twitter's own words) or "the license renewal was due" (a real driver in the Amazon and Alibaba Oracle exits, visible between the lines).
- **Timing dominates correctness**: Google could build Spanner in 2012; a startup in 2012 could not. Judge each choice against what was procurable *then*, not now.

## Case index

| Case | Move | Primary driver | Verdict (assessment) |
|---|---|---|---|
| Amazon (cart) | custom → Dynamo (2007) | availability under partition | Exemplary; became the template |
| Amazon (consumer) | Oracle → AWS services (2019) | cost, agility, license | Executed well; hiccups understated |
| Netflix | Oracle → SimpleDB → Cassandra (2008–11) | availability, cloud | Right for its time |
| Twitter | MySQL → Cassandra → Manhattan (2012–16) | ops burden, latency, consistency | Necessary then; buyable today |
| YouTube | single MySQL → Vitess (2010) | write scale | Exemplary; gift to industry |
| Slack | active-active MySQL → Vitess (2017–20) | sharding-key fix, resharding | Exemplary correction of own error |
| GitHub | single MySQL → domain split → Vitess shards | blast radius, growth | Exemplary sequencing |
| Shopify | single MySQL → sharded pods | merchant tenancy | Exemplary "boring tech" |
| Instagram | Postgres → sharded Postgres (2011) | growth with 3 engineers | Right call, maximal leverage |
| Pinterest | NoSQL → sharded MySQL → TiDB (2011–22) | NoSQL broke; then HBase ops | Textbook learning loop |
| Google (AdWords) | sharded MySQL → F1/Spanner (2012) | transactions + global scale | Gold standard |
| Alibaba/Ant | Oracle → OceanBase (2013–19) | consistency, peak, cost | Ambitious, validated |
| Uber | Postgres 9.2 → MySQL + Schemaless (2015) | ops skill, write patterns | Could have done better |
| Discord | MongoDB → Cassandra → ScyllaDB (2017–23) | GC pain, hot partitions, cost | Exemplary on all three generations |
| Dropbox (files) | AWS S3 → Magic Pocket (2016) | cost at exabyte scale | Right at their scale only |
| Dropbox (metadata) | MySQL → Edgestore → Panda | sharding ceiling | Exemplary future-proofing |
| Facebook (UDB) | InnoDB → MyRocks (2016–19) | space on flash | Exemplary; open-sourced |
| LinkedIn | Oracle + Voldemort → Espresso | scale + features + cost | Right then; buyable now |
| Stack Overflow | stayed on SQL Server | simplicity | The counter-example that works |
| 37signals | stayed on one MySQL/Postgres | team size, speed | The counter-example that works |
| Cloudflare | SQLite in Durable Objects (2023) | strong consistency at edge | The 2026 generation of the same trade |

---

# Findings: the dossiers

## Pattern 1 — Availability over consistency: the pioneering exits

### 1. Amazon — building Dynamo (2007)

**Context.** Amazon's retail services (cart, sessions, wish lists, product catalog) were served by relational databases that "would lead to inefficiencies and limit scale and availability" ([Dynamo, SOSP 2007](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)).

**Choice.** A purpose-built, always-writable key-value store: consistent hashing, quorum replication, vector clocks, application-side conflict resolution.

**Motivation.** The cart must accept writes *during* a partition or outage; a relational policy for resolving conflicts would be "too simplistic for any application," so reconciliation is pushed to the application, which knows the semantics (the cart merges by union of items).

**What happened.** Dynamo became the intellectual ancestor of Cassandra, DynamoDB, and virtually every eventually consistent store.

**Verdict (assessment).** Exemplary. The deep insight — that even the *conflict-resolution policy* belongs to the application — has aged perfectly. Could they have done better? No; in 2007 nothing procurable offered partition-tolerant writes. The only modern critique is that many teams copied Dynamo's trade-offs without Amazon's application-side discipline, which is the copiers' fault, not Amazon's.

### 2. Amazon — the great Oracle exit (2019)

**Context.** Amazon's consumer business ran its core systems on Oracle for decades, while AWS sold competing database services — a publicly acrimonious vendor relationship.

**Choice.** Complete migration: 75 PB across ~7,500 Oracle databases onto DynamoDB, Aurora, RDS and Redshift, with "little or no downtime," workload-matched: low-latency services to DynamoDB/ElastiCache, transactional relational to Aurora/RDS, analytics to Redshift. Named results include Buyer Fraud (40 TB moved with one hour of downtime, same-or-better performance at half the cost on Aurora) and Financial Ledger (120 TB, latency down 40%, cost down 70%, on DynamoDB) ([AWS News Blog](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/)).

**Motivation.** Officially performance, cost and agility; the commercial rivalry with Oracle was an obvious additional pressure.

**What happened.** The migration finished after several years, not without friction: by October 2018 Amazon was ~92% done, and applications suffered "intermittent timeouts" from savepoint-management mismatches during the Aurora transition — a reminder that even a cloud giant with the best tooling hits semantic translation costs ([CIO Dive](https://www.ciodive.com/news/amazon-kisses-last-oracle-database-goodbye-completes-years-long-migration/565073/)).

**Verdict (assessment).** Could they do better? The *workload matching* is the exemplary part — they did not convert Oracle to one flagship product but split by access pattern. The understated lesson is semantic: savepoints, procedural code and PL/SQL semantics do not port automatically. An honest report would lead with that, not with the celebration video.

### 3. Netflix — Oracle → SimpleDB → Cassandra (2008–2011)

**Context.** In 2008, Netflix's single Oracle instance for the website "melted down" — a multi-master replication issue left the site down and Netflix "couldn't ship DVDs for ~3 days" ([Cassandra from the trenches](https://www.slideshare.net/jasedbrown/cassandra-from-the-trenches-migrating-netflix-10586521)). The company resolved to abandon its datacenter entirely.

**Choice.** Move everything to AWS; replace the monolithic Oracle with Cassandra — with a transitional stop at SimpleDB.

**Motivation.** "Solutions like Oracle don't run very well on virtualized hardware … the architecture of Cassandra and the availability and consistency tuning and scalability made it a clear choice" (Christos Kalantzis). Adrian Cockcroft's framing: with one database "everything is in one place, which is only convenient until it fails"; many Cassandra instances "break at different times, so you're losing small pieces of the system, rather than the whole system at once" ([InfoWorld](https://www.infoworld.com/article/2171162/big-movies-big-data-netflix-embraces-nosql-in-the-cloud.html)).

**What happened.** SimpleDB proved unreliable at their volumes — itself a case study in "managed primitive ≠ production database" — and was in turn replaced. Cassandra became Netflix's backbone, and Netflix became Cassandra's most famous reference customer.

**Verdict (assessment).** Right for its time: in 2008–11 there was no procurable multi-region strongly consistent alternative on cloud. With 2026 hindsight, the same outage might push a team toward a distributed SQL engine, keeping consistency for subscriptions and billing. Netflix also paid the real Dynamo-style price: years of building tooling (Astyanax, Dyno-queues, Raigad) to make Cassandra operable. The honest summary: correct choice, underestimated running cost.

## Pattern 2 — Scale-out relational discipline

### 4. YouTube — building Vitess (2010)

**Context.** YouTube's write traffic "became too high for the master database to handle," and application-level sharding plus replicas stopped being enough ([Vitess history](https://vitess.io/docs/21.0/overview/history/)).

**Choice.** Rather than adopt a NoSQL store, YouTube doubled down on MySQL and built Vitess: a routing/proxy layer (VTGate), per-MySQL sidecars (VTTablet), and later, online resharding. Open-sourced, it became a CNCF project.

**Motivation.** Keep MySQL's transactional semantics and the team's operational knowledge; make sharding a platform concern instead of an application one.

**What happened.** Vitess now runs Slack, GitHub, Square, Shopify-adjacent fleets, JD.com (35M QPS peak), Etsy and others; its creators' first resharding "took an entire night of master downtime" — the pain that motivated online resharding ([Vitess history](https://vitess.io/docs/21.0/overview/history/), [PlanetScale](https://planetscale.com/blog/horizontal-sharding-for-mysql-made-easy)).

**Verdict (assessment).** Exemplary — the rare case where solving your own problem created an industry utility. Could they do better? The one regret embedded in the history is how long resharding stayed manual and offline; Vitess itself fixed that, so the answer is "they did."

### 5. Slack — active-active MySQL → Vitess (2017–2020)

**Context.** Slack ran all data on active-active MySQL clusters. Its original design placed each workspace — with all of its data — on a single shard, assuming tenant isolation. A Slack engineer later described that assumption as "the original sin of early Slack infrastructure that we adopted Vitess to migrate away from" ([Hacker News discussion](https://news.ycombinator.com/item?id=27199229)).

**Choice.** Vitess: re-shard messages by channel ID with online splits, keeping MySQL semantics.

**Motivation.** Fix a bad sharding key, gain online resharding, standardize operations. Migration began in 2017 with a low-risk workload (RSS feeds), and Vitess now serves 99% of Slack's query load; Slack became a top Vitess contributor ([Slack Engineering](https://slack.engineering/scaling-datastores-at-slack-with-vitess/)).

**What happened.** A ~3-year incremental migration using double-writes and a parallel read-diffing system comparing old and new paths — executed through the 2020 work-from-home traffic spike.

**Verdict (assessment).** Exemplary *correction*. Could they have done better? Yes — at design time, in 2014, by choosing a finer sharding key (channel, not workspace). The deeper lesson: sharding keys are chosen long before the pain arrives, and "the tenant is the shard" is the single most seductive wrong choice in SaaS.

### 6. GitHub — splitting the monolith's database (2021–2023)

**Context.** GitHub began as a Rails app with one MySQL database; 15 years later it runs 1,200+ hosts, 300+ TB, 5.5M queries/second across 50+ clusters ([GitHub Blog](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/)).

**Choice.** Cheapest thing first: vertical partitioning — moving groups of tables (domains) apart without splitting rows — using Vitess's vertical-sharding features and VTGate as the application endpoint; only *then* horizontal sharding for the largest domains, with linters preventing queries from joining across domains ([GitHub Blog](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/)).

**Motivation.** Blast radius and growth, not raw QPS: split the failure domains before splitting rows.

**Verdict (assessment).** Exemplary sequencing, and the most imitable of all cases: vertical (domain) splits are low-risk, reversible, and buy years before real sharding. Could they do better? Nothing evident; the MySQL 8.0 fleet-wide upgrade under SLO was later done the same incremental way.

### 7. Shopify — sharded "pods" and zero-downtime shop moves

**Context.** Shopify's data model has a natural tenant: the shop. Since ~2014 it has sharded MySQL by `shop_id`.

**Choice.** A podded runtime: each pod is an isolated slice of Shopify — its own MySQL shard, Redis, Memcached — with Nginx routing by shop. Shard balancing moves whole shops between MySQL shards, "entirely online and with virtually zero consumer-facing downtime," chosen by data-driven analysis of utilization to defuse hotspots ([Shopify Engineering](https://shopify.engineering/mysql-database-shard-balancing-terabyte-scale)).

**Motivation.** Keep Rails + MySQL; scale horizontally by adding pods; isolate failures; survive Black Friday.

**Verdict (assessment).** Exemplary boring-technology scaling. The interesting subtlety is that Shopify's shard-balancing handles the problem most sharded systems consider intractable — *post-hoc* key rebalancing — by making the tenant movable instead of rehashing the keyspace. Could they do better? The pod model does duplicate stateful services per pod; that's a cost they accept for isolation.

### 8. Instagram — sharded Postgres with three engineers (2011)

**Context.** At 25 photos and 90 likes per second, Instagram evaluated NoSQL options and decided to shard PostgreSQL instead ([Instagram Engineering](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c)).

**Choice.** Thousands of *logical* shards implemented as Postgres schemas, mapped to few physical servers; ID generation (timestamp + shard bits) pushed *into* Postgres via PL/pgSQL to avoid running a separate ID service; later growth absorbed by memcache + Redis rather than re-platforming.

**Motivation.** Team size. With three engineers, every component must be one they already know how to operate.

**Verdict (assessment).** Exemplary leverage. The ID scheme design is the most-copied part (64-bit IDs encoding shard + timestamp). Could they do better? The schemas-as-logical-shards trick made moving shards a matter of relocating logical units — the right primitive. No visible error.

### 9. Pinterest — NoSQL broke, then MySQL, then TiDB (2011–2022)

**Context.** "Around September 2011, every piece of our infrastructure was over capacity. We had several NoSQL technologies, all of which eventually broke catastrophically. We also had a boatload of MySQL slaves … lots of irritating bugs" ([Pinterest Engineering](https://medium.com/pinterest-engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f)).

**Choice.** Home-grown sharded MySQL (ShardDB) with app-managed ID→shard mapping.

**Motivation.** Operability by a small infra team; predictable behavior over feature richness.

**What happened.** A decade later, Pinterest re-ran the decision properly: in 2022 it evaluated 10+ candidates (in-house KV, ShardDB, Vitess, VoltDB, Phoenix, Spanner, CosmosDB, TiDB, YugabyteDB…) and deprecated HBase while adopting TiDB for workloads needing horizontal scale without manual sharding. Notably, their assessment of Vitess: "cross-shard queries may not be optimized … relatively high maintenance costs"; of Spanner: "not open source … migration cost from AWS to GCP could be prohibitively high" ([Pinterest Engineering](https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10)).

**Verdict (assessment).** The best learning loop in the set: adopt → break → build simple → evaluate honestly → buy the converged engine. Could they do better? Only in wishing the 2022 evaluation had been possible in 2011 — it wasn't.

## Pattern 3 — Transactions at global scale: distributed SQL

### 10. Google — F1 on Spanner, for AdWords (2012)

**Context.** AdWords ran on manually sharded MySQL on specially acquired hardware. Ads "had just completed a very challenging resharding project … and was resolute in its desire to never do it again" ([Google Cloud blog](https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award)).

**Choice.** F1, a distributed SQL layer over Spanner: synchronous cross-datacenter replication, global transactions via TrueTime, SQL with joins.

**Motivation.** Ads is a billing system: eventual consistency was never an option, MySQL master-slave failover "was difficult, and risked data loss and downtime," and manual resharding was unsustainable ([the morning paper](https://blog.acolyer.org/2015/01/08/spanner-googles-globally-distributed-database/)). F1 explicitly replaces the "sharded MySQL hacky implementation" with principled infrastructure ([Murat Demirbas's notes](http://muratbuffalo.blogspot.com/2023/07/f1-distributed-sql-database-that-scales.html)).

**What happened.** F1 ran all AdWords data from 2012; the accepted cost was higher per-operation latency, hidden by asynchronous application patterns.

**Verdict (assessment).** Gold standard. Could they do better? No — they built the thing everyone else now buys (Spanner, CockroachDB, YugabyteDB, TiDB all inherit from this lineage). The transferable insight: they accepted *latency* as the price of correctness rather than giving up correctness — the inverse of most 2010s NoSQL trade-offs.

### 11. Alibaba / Ant — Oracle → OceanBase (2013–2019)

**Context.** Alipay's core ledger ran on Oracle and expensive hardware; peak events (Double 11) exceeded what vertical scaling could absorb.

**Choice.** Build OceanBase from scratch: distributed, low-cost commodity servers, strongly consistent. "In 2013, Alipay decided to stop using Oracle. Since the alternative choice MySQL can't ensure strong consistency between active server and standby server, OceanBase got its first opportunity" ([Database of Databases](https://dbdb.io/db/oceanbase/revisions/24)).

**Motivation.** Three stated principles: distributed, low cost, high reliability; plus transaction rates no procurable system could meet (the 2017 Double 11 peak reached 256,000 payments per second on OceanBase, with all Alipay core data on it) ([Alibaba Cloud case studies](https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies)).

**What happened.** OceanBase carried Ant Financial to 100% of traffic, won the TPC-C world record from Oracle in 2019, and is now sold externally.

**Verdict (assessment).** Ambitious, and validated by the one benchmark nobody can fake for long: nine consecutive Double 11s. The honest caveats: the build took a dedicated database team (most companies don't have one), and early versions were meaningfully slower than Oracle per query — the team's own account describes taking on the "10% gap" deliberately. Today this exact need is served by commercial distributed SQL; building remains justified only at Alibaba's scale.

## Pattern 4 — NoSQL adopted, then regretted

### 12. Twitter — MySQL → Cassandra → Manhattan, and back to strong consistency (2012–2016)

**Context.** Twitter's timeline storage grew through MySQL and Cassandra generations, each leaving operational scars.

**Choice.** Build Manhattan, a real-time multi-tenant distributed database.

**Motivation.** Striking honesty: "We were spending far too much time firefighting production systems … standing up new storage capacity … involved too much manual work and process. Our experience … made it clear that the situation was simply not sustainable" ([Twitter Engineering](https://blog.x.com/engineering/en_us/a/2014/manhattan-our-real-time-multi-tenant-distributed-database-for-twitter-scale)); a lead engineer framed the need as "millions of queries per second, with extremely low latency in a real-time environment" ([InfoQ](https://www.infoq.com/news/2014/05/twitters-manhattan/)).

**What happened.** Manhattan became the primary store for tweets, DMs and ads — and then, tellingly, in 2016 Twitter *added strong consistency* to it, because eventual consistency could not guarantee basics like key uniqueness: "If we can't guarantee uniqueness of, for example, email addresses … one email address may get attached to two accounts" ([Twitter Engineering](https://blog.x.com/engineering/en_us/a/2016/strong-consistency-in-manhattan)). By 2020, Twitter's own engineers noted the distributed-SQL landscape had evolved so much that "some of these solutions can potentially complement or maybe even replace our internal offerings in the future" ([Yugabyte recap](https://www.yugabyte.com/blog/recap-the-distributed-database-behind-twitter/)).

**Verdict (assessment).** The most instructive arc: adopt Cassandra (the 2010s default), discover that predictable latency and ops simplicity dominate, build Manhattan, then discover that your customers need strong consistency and retrofit it — effectively rebuilding a distributed SQL database bottom-up. Could they do better? Not with 2012's options; today, unambiguously yes — this is the exact requirement set Spanner-class engines sell.

### 13. Discord — MongoDB → Cassandra → ScyllaDB (2015–2023)

**Context.** Discord outgrew MongoDB for messages in 2015 and moved to Cassandra in 2017 for scalable linear writes ([InfoQ](https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/)).

**Choice.** After years of Cassandra pain — "GC pauses affecting latency, all the way to super long consecutive GC pauses … an operator would have to manually reboot and babysit the node" — Discord moved every database to ScyllaDB (Cassandra-compatible, C++, shard-per-core), leaving messages last ([Discord blog](https://discord.com/blog/how-discord-stores-trillions-of-messages)).

**Motivation.** Not the data model — Scylla speaks Cassandra's protocol — but the *operational shape*: predictable latency, no GC, fewer nodes. Plus, crucially, a Rust "data services" layer that collapses duplicate reads before they hit storage.

**What happened.** Trillions of messages moved from a 177-node Cassandra cluster to 72 ScyllaDB nodes with no downtime; a bespoke Rust migrator with SQLite checkpointing shortened the data move from three months to nine days ([InfoQ](https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/)).

**Verdict (assessment).** Exemplary on all three generations: each was the reasonable choice of its moment, and the final move is a masterclass — same data model, better engine, load shedding at the service layer, fast bespoke tooling, done while firefighting. The transferable insight (their own framing): only one of the three fixes was "a new database."

### 14. Uber — Postgres 9.2 → MySQL + Schemaless (2015)

**Context.** Uber's core trip database ran on Postgres 9.2, and the company published a detailed technical rationale for switching to MySQL: Postgres's page-level MVCC copies whole rows on update (write amplification for hot rows), replication was less flexible, schema changes locked tables, and the buffer pool and connection model fit their workloads poorly ([Uber Engineering](https://www.uber.com/us/en/blog/postgres-to-mysql-migration/)).

**Motivation.** Per the blog: engine mechanics. Per prominent outside reviewers: "The biggest thing biting PostgreSQL here is definitely lack of operational knowledge, as this was the decisive reason why Uber really migrated (back) to MySQL" ([Ayende](https://ayende.com/blog/175137/re-why-uber-engineering-switched-from-postgres-to-mysql)) — Uber's team knew MySQL deeply from previous work, and MySQL was also the substrate for their in-house Schemaless layer.

**What happened.** Uber built Schemaless on MySQL (a sharded document store above InnoDB) and reports satisfaction; the blog post became one of the most debated migration posts ever written.

**Verdict (assessment).** The clearest "could have done better" in the set — with caveats. The published argument leaned on Postgres 9.2-era limitations that later releases largely erased (logical replication in PG10, far better partitioning in PG10+, improved vacuum and update handling), and the row-copy critique applies mostly to update-heavy narrow rows, which Postgres has since mitigated (HOT updates aside, their hot-row pattern was unusual). A team with deep Postgres operational expertise could plausibly have stayed, upgraded, and schema-tuned. That said: (a) team skill *is* a legitimate architectural input, (b) Schemaless was the real product decision and MySQL suited it, and (c) "switch to what your best operators know" is defensible engineering even when the published mechanics are overstated. The fair hindsight verdict: the outcome was fine; the *justification* conflated engine limits with ops inexperience, and the industry took the wrong lesson from it ("Postgres doesn't scale") for years.

### 15. Pinterest's NoSQL detour — see dossier 9

Covered above; it belongs in this pattern too: "several NoSQL technologies, all of which eventually broke catastrophically."

## Pattern 5 — Build your own when cost or specialization dominates

### 16. Dropbox — Magic Pocket: off AWS S3, into own datacenters (2016)

**Context.** Dropbox was an early S3 adopter and one of AWS's largest customers; file content was in S3, metadata in-house.

**Choice.** After 2.5 years of building, migrate file storage to fully in-house infrastructure — serving 90%+ of user data from own datacenters ([Dropbox Tech](https://dropbox.tech/infrastructure/magic-pocket-infrastructure), [InfoQ](https://www.infoq.com/news/2016/03/Dropbox-AWS/)).

**Motivation.** Cost at extreme scale and control of the full stack — not any technical limitation of S3: "We used AWS S3 because storage at scale was an extremely hard problem to solve. It was only a few years later, when we really believed we could tackle this problem better for our needs, that we even tried" ([DCD interview](https://www.datacenterdynamics.com/en/analysis/how-dropbox-pulled-off-its-hybrid-cloud-transition/)).

**What happened.** The migration kept reversibility as a first-class feature — "for the initial migration out of S3 to Magic Pocket, we built the ability to move data back and forth" — and Dropbox still uses S3 where it lacks scale or price advantage (UK, mainland Europe, Japan) ([DCD](https://www.datacenterdynamics.com/en/analysis/how-dropbox-pulled-off-its-hybrid-cloud-transition/)). Inside the bold system, the boring choice: the Block Index is "a giant sharded MySQL cluster … MySQL turned out to be more than capable," which they call "a really great decision in terms of simplifying development and minimizing unknowns" ([Dropbox Tech](https://dropbox.tech/infrastructure/inside-the-magic-pocket)).

**Verdict (assessment).** Right at their scale, wrong below it — the case is instructive precisely because the team says the quiet part: their own storage organization's job is to "advocate us moving the data back" if AWS ever becomes cheaper. The generalizable lesson is the internal architecture, not the migration: bold system at the edges, boring components inside.

### 17. Dropbox — Edgestore → Alki → Panda: when sharding has a ceiling (2017–2024)

**Context.** Dropbox's metadata ran on two huge sharded-MySQL systems: the Filesystem and Edgestore, which fronts "thousands of MySQL nodes" ([Dropbox Tech](https://dropbox.tech/infrastructure/cross-shard-transactions-at-10-million-requests-per-second)).

**What happened.** Edgestore hit a structural ceiling: after splitting 256 clusters with 8 shards into 512 clusters with 4 shards, it could perform "only 2 more splits, after which we would reach 1 shard per cluster with no more splits available" ([Dropbox Tech](https://dropbox.tech/infrastructure/alki--or-how-we-learned-to-stop-worrying-and-love-cold-metadata)). They then built Panda, a transactional KV layer whose features (rebalancing, two-phase commit, engine independence) are deliberately implemented "independent of the underlying storage engine … we can even migrate metadata back and forth between entirely different storage engine implementations if we choose" ([Dropbox Tech](https://dropbox.tech/infrastructure/panda-metadata-stack-petabyte-scale-transactional-key-value-store)).

**Motivation.** Cost-efficiency at growth, and escape from the fixed-shard-count trap.

**Verdict (assessment).** Exemplary future-proofing. The lesson: a *fixed* number of shards is a deferred architectural debt with a known due date, and the escape hatch is abstraction — decouple the API from the engine so the engine becomes replaceable.

### 18. Facebook — InnoDB → MyRocks for the social graph (2016–2019)

**Context.** Facebook's UDB (tens of petabytes of social-graph activity) was space-bound on flash: "Even though we used InnoDB compression and there was excess CPU and random I/O capacity, it was unlikely that we could enhance InnoDB to use less space" ([Meta Engineering](https://engineering.fb.com/2017/09/25/core-infra/migrating-a-database-from-innodb-to-myrocks/)).

**Choice.** MyRocks: RocksDB (LSM-tree) as a MySQL storage engine. "Our objective was … reduce the amount of storage and number of servers used by half."

**What happened.** Storage halved as targeted; the VLDB paper reports instance size down 62.3%, servers down to less than half, fewer I/O operations and less CPU for the same traffic; migration ran in careful stages with double-write consistency checks and InnoDB replicas kept for rollback; the success later justified moving Messenger off HBase onto MyRocks ([VLDB 2020](https://www.vldb.org/pvldb/vol13/p3217-matsunobu.pdf)).

**Verdict (assessment).** Exemplary. Note the shape of the decision: not "new database," but *new storage engine under the same database* — schema, replication, tooling and application stack unchanged, "mostly transparent to client applications." That is the smallest possible blast radius for a storage migration. Could they do better? No; and they compounded the win by open-sourcing (MyRocks now ships in Percona Server).

### 19. LinkedIn — Oracle + Voldemort → Espresso (2010s)

**Context.** LinkedIn served online apps from a split stack: "Relational Database Management Systems (RDBMSs) such as Oracle and key-value stores such as Voldemort — both serving different use cases" ([LinkedIn Engineering](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store)).

**Choice.** Espresso: a home-grown distributed *document-relational* store with transactions, secondary indexes, and hash partitioning by primary key — combining Oracle's feature set with Voldemort's horizontal scale.

**Motivation.** Feature-rich + scalable + cheaper than Oracle at their growth rate; a unified system instead of a bifurcated one.

**What happened.** Espresso became the source of truth for hundreds of terabytes and some of LinkedIn's most valuable datasets, serving millions of records per second at peak.

**Verdict (assessment).** Right for its time; today this exact need — Oracle's semantics without Oracle's scaling limits — is precisely what PostgreSQL-family and distributed SQL products sell, and a 2026 LinkedIn would almost certainly buy rather than build. The case remains valuable as the clearest statement of *why* document-relational hybrids exist: the data model follows the access pattern (a user is a document group), not the other way around.

## Pattern 6 — Deliberate simplicity: the counter-examples

### 20. Stack Overflow — a couple of SQL Server boxes (2013–present)

**Context.** One of the highest-traffic sites on earth ran — famously — on ~10 servers, with two SQL Server masters plus replicas: 334M SQL queries and 148M HTTP requests in a day, average write time "0 milliseconds," SQL servers with 384 GB RAM and SSDs, kept at 5–10% CPU ([Nick Craver, 2013](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/); [2016 architecture](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/)).

**Motivation.** "Scale but not at scale": extreme efficiency, scale-up hardware, vanilla SQL (one stored procedure, Dapper micro-ORM), heavy caching — and no sharding, ever.

**Verdict (assessment).** The counter-example that works, with honest boundary conditions: read-mostly workload, exceptional in-house ops depth, and a business that benefits from hardware efficiency. Could they do better? No — by their own analysis, moving to cloud/distributed storage would cost more and add risk for no user-visible gain. The transferable lesson is *measured restraint*: every unit of scale they avoided buying, they spent on engineers instead.

### 21. 37signals — the majestic monolith, one big database (2004–present)

**Context.** Basecamp and HEY run as "the majestic monolith … It's one app with one big database, and there's a lot of efficiency in terms of workflow that you get from just having one database … Stay in the majestic monolith as long as you can. We've been running this architecture for 10 years" ([37signals dev](https://dev.37signals.com/the-rails-delegated-type-pattern/)).

**Motivation.** Team size and iteration speed: joins across one database make features cheap; a small team can operate one database well.

**Verdict (assessment).** Same verdict as Stack Overflow with the same boundary conditions. Together the two cases falsify "scale means distribution" for the large class of products whose workload fits one box — with a caveat both would endorse: the moment your data model outgrows one node, choose the tenant key before you think you need it.

## Pattern 7 — The 2026 generation: consistency at the edge

### 22. Cloudflare — SQLite in Durable Objects (2023)

**Context.** Traditional cloud architecture "favors large distributed databases that live separately from application servers, while SQLite is designed to run as an embedded library" ([Cloudflare blog](https://blog.cloudflare.com/sqlite-in-durable-objects/)).

**Choice.** A full SQLite database embedded in each Durable Object, with local disk as a cache over durable object storage (R2), giving "essentially zero" storage latency that is "nevertheless durable and consistent."

**Motivation.** Strong consistency *without distributed consensus*: each object has exactly one writer, so a per-object SQLite instance is serialized by construction — "since this storage lives together with the object, it is strongly consistent yet fast to access" ([Cloudflare docs](https://developers.cloudflare.com/durable-objects/)).

**Verdict (assessment).** Structurally, this is Shopify's pods, Amazon's per-service Dynamo, and Stack Overflow's scale-up combined and miniaturized: instead of distributing one database, *shard the application* and give each shard an embedded database. It is the strongest current answer to the tenancy problem — the shard boundary is the object (chat room, document, tenant), and consistency comes from the single writer, not from Paxos. Watch its limits: per-object storage ceilings make it a per-tenant store, not a global analytics engine.

---

# Cross-case analysis: what actually drives these decisions

Reading the twenty cases together, the *stated* technical reasons collapse into five real drivers:

1. **Availability and blast radius** (Amazon, Netflix, Shopify, GitHub): the fear is not QPS but one failure taking everything. The structural answer is almost always tenant or domain isolation, not a different database.
2. **Operational burden** (Twitter, Discord, Pinterest, Dropbox): every "we adopted the hot new store" story eventually became a story about GC pauses, repairs, hot partitions, babysitting nodes, and on-call load. Twitter's "we were spending far too much time firefighting" and Discord's GC chronicle are the same story with different engines.
3. **Cost at scale** (Facebook, Dropbox, Amazon, Alibaba): MyRocks halved servers; Magic Pocket saved S3 margins; Oracle licenses pushed two giants. Cost, not fashion, drives the largest migrations.
4. **Correctness requirements** (Google F1, Alibaba OceanBase, Twitter's Manhattan retrofit, Slack): billing, ledgers, identity and uniqueness eventually force strong consistency. Every team that gave it up for availability bought it back later, expensively.
5. **Team skill** (Instagram, Uber, Stack Overflow, 37signals): the engine your best operators know is worth more than any benchmark. Uber is the cautionary twin: skill justified the move, but its blog blamed the engine, misleading readers for years.

Three recurring regularities:

- **The tenant key is the whole game.** Shopify (shop), Slack (channel, after workspace failed), Instagram (user ID in the ID itself), Cloudflare (object). Choose it early and choose it fine-grained.
- **Every successful migration had the same shape**: dual writes → backfill → shadow reads with diffing → incremental cutover (per tenant/shard) → rollback kept warm throughout. Slack's read-diffing, MyRocks' staged rollout with InnoDB rollback replicas, Discord's checkpointed migrator, Shopify's per-shop moves, and Dropbox's bidirectional S3↔Magic Pocket replication are the same playbook in five dialects.
- **Build only what converges.** The builds that paid off (Vitess, MyRocks, Magic Pocket) became platforms or open source — the externality was part of the return. The builds that merely solved an internal need (Espresso, Manhattan, Schemaless) delivered less than the equivalent 2026 product would, because their teams bear the full maintenance forever.

# A decision framework, distilled from the cases

1. **Start boring.** One Postgres/MySQL on good hardware, measured. Stack Overflow and 37signals define the ceiling of "boring" — it is higher than almost everyone believes.
2. **Split vertically before horizontally.** GitHub's domain tables and Shopify's pods are lower-risk and reversible; row-level sharding is neither.
3. **Find the natural tenant and make it the shard key** — and make tenants movable (Shopify) rather than statically hashed. Never choose the *organizational* boundary (Slack's workspace) when a finer one exists.
4. **Match the consistency requirement to the money.** Anything that bills, debits, or uniquely identifies (Google, Alibaba, Twitter's retrofit) needs strong consistency; caches, feeds and carts (Amazon) genuinely don't.
5. **Weigh ops cost as first-class.** The Discord lesson: a "better database" bought less than keeping load off the database; the Twitter lesson: firefighting time is a migration reason even when benchmarks look fine.
6. **Re-check the market before building.** In 2012 Twitter had to build Manhattan; in 2026 Pinterest could evaluate ten products and adopt one (TiDB). Building a general-purpose store now requires justification that a product (or a storage engine swap, à la MyRocks) cannot meet.
7. **Migrate with reversibility**: dual-write, backfill, diff, incremental cutover, warm rollback. If your plan cannot return to the old system on day 60, it is not a plan (Discord, MyRocks, Dropbox all obeyed this).

# One engine per family: the strongest documented production use cases

*The companion 2026 report lists ten database families. For each, here is the strongest real production evidence found — "strongest" meaning largest or best-attested, not "only." Cases already covered by a dossier above point back to it.*

| Family | Strongest documented case | The evidence |
|---|---|---|
| Relational (OLTP) | GitHub; Stack Overflow | Dossiers 6 and 20. GitHub: 1,200+ MySQL hosts, 300+ TB, 5.5M queries/second across 50+ clusters ([GitHub Blog](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/)); Stack Overflow: ~148M HTTP requests and 334M SQL queries a day on a couple of SQL Server masters ([Nick Craver](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/)). |
| Document | eBay on MongoDB | eBay — 170M+ active buyers, 1B+ live listings, 190 markets — runs mission-critical multi-datacenter applications on MongoDB, which per the vendor-published account "now also serves System of Record use cases at eBay" ([MongoDB blog](https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb)). The operations-side twin is Stripe's document database underpinning 99.999% uptime with zero-downtime migrations ([Stripe](https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations)). |
| Key-value | Amazon on DynamoDB | The Financial Ledger: 120 TB off Oracle, latency down 40%, cost down 70% ([AWS News Blog](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/)); the small-team version is Mixmax's MongoDB→DynamoDB migration ([Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb)). |
| Wide-column | Apple on Cassandra | The Apache Cassandra project credits Apple with 160,000+ instances storing 100+ PB across 1,000+ clusters — the largest documented Cassandra fleet — and Netflix with 10,000+ instances, 6 PB and over a trillion requests per day ([DevPro Journal, Cassandra 4.0 release](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/)). Reported caveat: press accounts since 2015 say Apple has been moving parts of iCloud/CloudKit off Cassandra onto FoundationDB ([The Dulin Report](https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/), [Engineers' Codex](https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions)) — press-reported, not first-party. |
| Columnar | Cloudflare on ClickHouse | HTTP analytics at 6M requests/second on ClickHouse, replacing a 2014-era CitusDB (Postgres) pipeline ([Cloudflare blog](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/)); a critical log producer moved "from Elasticsearch to a much smaller ClickHouse cluster" ([Cloudflare blog](https://blog.cloudflare.com/log-analytics-using-clickhouse/)); the case is showcased at quadrillion-row scale ([ClickHouse blog](https://clickhouse.com/blog/cloudflare)). |
| Graph | ICIJ on Neo4j | 11.5 million leaked Panama Papers files explored by hundreds of journalists through Neo4j + Linkurious: "relationships are all-important in telling you where the criminality lies, who works with whom" (Mar Cabra, ICIJ) ([Computer Weekly](https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software), [Neo4j interview](https://neo4j.com/blog/cypher-and-gql/icij-neo4j-unravel-panama-papers/)). The commercial twin: Zurich Insurance's claims-fraud investigation, 500,000 claims a year ([Neo4j case study, PDF](https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf)). |
| Time-series | Uber on M3 | Built in-house because at Uber's cardinality "common solutions like Prometheus and Graphite become difficult to manage and sometimes cease to work" ([Uber Engineering](https://www.uber.com/us/en/blog/billion-data-point-challenge/)); M3 downsamples at collection time, serves multi-region queries, and was open-sourced — reported at 6.6 billion time series ([Uber Engineering](https://www.uber.com/us/en/blog/m3/), [Packt](https://www.packtpub.com/en-in/learning/tech-news/uber-open-sources-its-large-scale-metrics-platform-m3-for-prometheus)). The cautionary twin is the failed TimescaleDB migration (post-mortem 31). |
| NewSQL | DoorDash on CockroachDB | DoorDash's fulfillment backend: a Postgres single-primary write bottleneck whose weekly Friday peak "would crash and burn," against DoorDash's recommended ≤500 GB Postgres tables; the move to CockroachDB removed the write bottleneck and the team reports handling 10× load ([DoorDash engineering](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/), [Cockroach Labs](https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/)). Pinterest's TiDB adoption (dossier 9) and Google F1 (dossier 10) complete the family. |
| In-memory | Netflix on EVCache | EVCache: a distributed, replicated memcached-semantics cache purpose-built for Netflix's AWS estate — zone-aware topology, 2–9 replicas per cluster, so "most of the requests are served by the cache" ([Netflix EVCache wiki](https://github.com/Netflix/EVCache/wiki)). |
| Vector | turbopuffer | Object-storage-first search serving Anthropic, Atlassian, Cursor and Notion; 4 trillion documents in production and 25,000+ searches per second as of April 2026 ([AI Engineer profile](https://ai.engineer/orgs/turbopuffer)). The founding economics are the interesting part: Readwise paid ~$5k/month for its relational database, but vector search over the same 100M+ documents would have cost $20k+/month on incumbent architectures — a feature shelved until the cost structure changed ([turbopuffer blog](https://turbopuffer.com/blog/turbopuffer)); Cursor's migration cut its search costs ~95% ([Latent Space](https://www.latent.space/p/turbopuffer)). |

**How to read this table (assessment).** Several "strongest cases" rest partly on vendor-published material (eBay/MongoDB, Zurich/Neo4j, DoorDash/Cockroach Labs, the ClickHouse-published Cloudflare profile); where the company documents itself (Uber, Netflix, Cloudflare's own blog, turbopuffer's own engineering posts), the account is first-party. And none of it implies exclusivity: Apple runs FoundationDB alongside Cassandra, DoorDash still runs dozens of Postgres-adjacent services, Cloudflare also uses Postgres and SQLite at other layers. "Strongest case" means strongest evidence for the family's fit — not a monoculture.

## The cases, compared family by family

*One flagship proves a family can work; two or three prove the conditions under which it works. Each family below gets its documented cases and a comparison.*

### Relational (OLTP): three ways to stay relational

GitHub, Stack Overflow and 37signals solved three different "scale" problems without leaving the family: GitHub split by domain and shard on MySQL/Vitess at 5.5M queries/second ([GitHub Blog](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/)); Stack Overflow scale-up'd to ~334M SQL queries a day on two masters at 5–10% CPU ([Nick Craver](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/)); 37signals ran one big database for over a decade because joins are cheap and teams are small ([37signals dev](https://dev.37signals.com/the-rails-delegated-type-pattern/)).

**Comparison (assessment).** Same family, three topologies, one shared decision: keep single-writer SQL semantics and change *where* rows live, not *what* the data model is. The comparison also falsifies "scale forces distribution": Stack Overflow and 37signals out-serve the actual traffic of most distributed deployments on a single writer.

### Document: three reasons for the same family

eBay runs mission-critical multi-datacenter applications on MongoDB, which now "also serves System of Record use cases" ([MongoDB blog](https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb)); Stripe runs document databases under 99.999% uptime with zero-downtime migrations ([Stripe](https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations)); EA's Spearhead studio selected MongoDB to store user data and game state for FIFA Online, and Sailthru grew to 125M content documents and 5B messages a month, 40 TB across 120 nodes ([MongoDB at scale, vendor](https://www.mongodb.com/products/capabilities/mongodb-scale)).

**Comparison (assessment).** Three different jobs: flexible product-catalog schemas (eBay), durable operational documents under microservices (Stripe), rapidly evolving game state (EA). All share one property: the *aggregate* — listing, player, message set — is the natural unit, accessed whole and rarely joined across. That is the document family's real fit test, not "unstructured data."

### Key-value: the same pattern at three scales

Amazon's Financial Ledger put 120 TB off Oracle onto DynamoDB, latency down 40%, cost down 70% ([AWS News Blog](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/)); Duolingo stores 31 billion items in DynamoDB to deliver lessons ([AWS talk by Karin Tsai, Duolingo](https://www.youtube.com/watch?v=fhnAvn2YxZA)); Mixmax moved from self-run MongoDB to DynamoDB at small scale ([Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb)).

**Comparison (assessment).** The design is invariant across scales: single-key access, no joins, availability over read-modify-write. What changes is the cost lever. Duolingo's later tuning is the tell: extending a cache TTL cut inter-service traffic by more than 60%, and stale data in DynamoDB tables was pure spend ([Duolingo blog](https://blog.duolingo.com/reducing-cloud-spending/)) — in managed key-value, the bill is call volume, so *call* discipline matters more than schema discipline.

### Wide-column: three fleets and one recurring cure

Apple (160,000+ instances, 100+ PB, 1,000+ clusters) and Netflix (10,000+ instances, 6 PB, over a trillion requests a day) are the planetary deployments ([DevPro Journal](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/)); Instagram runs Cassandra beside Postgres for activity feeds, and when JVM limits bit, its engineers replaced Cassandra's *storage engine* with a RocksDB-based one — "Rocksandra" — behind the same API, hoping to push Cassandra toward a pluggable-engine architecture ([The New Stack](https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/)).

**Comparison (assessment).** The family's recurring pain is operational — GC pauses, repairs, hot partitions — and the recurring cure is not a new database but an engine transplant under the same protocol: Instagram's Rocksandra and Discord's Cassandra→ScyllaDB are the same move at different layers, and Facebook's InnoDB→MyRocks is its relational cousin. Choose wide-column for write-heavy fan-out; budget for the JVM and repair story from day one.

### Columnar: two opposite access patterns, one family

Cloudflare is the fan-in shape: 6M requests/second of telemetry pre-aggregated at ingest, TTL'd, with log workloads moved from Elasticsearch to a much smaller cluster ([Cloudflare blog](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/)). Bloomberg is the opposite: "What I have to optimize for is one single person querying data about a hundred million securities all at once. And what if then a thousand people do the same thing?" — on-prem, immutable data, MergeTree clones, with ZooKeeper stress during clones pushing them toward disaster-recovery drills and ClickHouse Keeper ([ClickHouse blog, Bloomberg meetup report](https://clickhouse.com/blog/nyc-meetup-report-large-scale-financial-market-analytics-with-clickhouse)). CERN's LHCb experiment applied the same engine to metadata on 10 billion events with 1,000+ attributes each ([Wikipedia — ClickHouse](https://en.wikipedia.org/wiki/ClickHouse)).

**Comparison (assessment).** Columnar's fit is not "analytics" in the abstract but *scanning few columns across many rows*. Fan-in telemetry (summarize everything cheaply) and fan-out deep queries (scan history on demand) both reduce to that — which is why the same engine serves a CDN, a trading terminal and a particle detector.

### Graph: three different jobs

ICIJ's Panama Papers work is exploration — the graph *is* the product: 11.5 million files, hundreds of journalists expanding networks by double-clicking through Neo4j + Linkurious ([Computer Weekly](https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software)). Zurich Insurance is acceleration: fraud investigators see context in a claims graph that tabular views could not assemble ([Neo4j case study, PDF](https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf)). eBay's ShopBot is inference context: a knowledge graph behind a conversational shopping assistant, with natural-language understanding and machine learning run *on* the graph ([Neo4j interview with eBay's Ajinkya Kale, vendor](https://neo4j.com/blog/knowledge-graph/machine-learning-on-graphs-ajinkya-kale-senior-applied-researcher-ebay/)).

**Comparison (assessment).** The common denominator is the multi-hop query — "everyone connected to X within two steps, with weights" — which relational engines answer with recursive joins that collapse at depth. The three cases also form a maturity ladder: exploratory link analysis is the easy entry, production ML on graphs the hard end.

### Time-series: the family both giants built

Uber built M3 because at its cardinality "common solutions like Prometheus and Graphite become difficult to manage and sometimes cease to work" ([Uber Engineering](https://www.uber.com/us/en/blog/billion-data-point-challenge/)); before M3, its metrics went to a Graphite stack with Whisper files in a sharded Carbon cluster ([Packt](https://www.packtpub.com/en-in/learning/tech-news/uber-open-sources-its-large-scale-metrics-platform-m3-for-prometheus)). Netflix built Atlas to replace "Epic" — perl CGI scripts, RRDTool and MySQL — which struggled at ~2M distinct time series and needed an order of magnitude more; Atlas sustains over 1.2 billion time series in failover drills ([Netflix TechBlog](http://techblog.netflix.com/2014/12/introducing-atlas-netflixs-primary.html), [Atlas docs](https://netflix.github.io/atlas-docs/overview/)). Years later, Netflix rebuilt alerting into streaming evaluation after configured alerts grew 5× and critical health alerts reached engineers 45 minutes late ([Netflix TechBlog](https://netflixtechblog.com/improved-alerting-with-atlas-streaming-eval-e691c60dc61e)).

**Comparison (assessment).** Two giants, one conclusion: at their cardinality, the time-series products of the day did not solve cardinality, downsampling and retention economics *together* — so both built and open-sourced. Both then hit the same second-order mountain: the query/alert architecture matters as much as storage (M3's DAG query engine; Atlas's streaming eval). And the NexusTrade failure (post-mortem 31) is the mirror image: at small scale, the purpose-built engine lost to a generalist with time-series features. The family rewards scale-aware choice — build at giant cardinality, stay generalist below it.

### NewSQL: three triggers, one destination

DoorDash moved off Aurora Postgres because a single-primary write bottleneck "would crash and burn" at weekly peaks ([Cockroach Labs](https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/)); Pinterest moved off HBase/NoSQL wanting horizontal scale without manual sharding ([Pinterest Engineering](https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10)); Google built F1/Spanner to "never reshard again" with global transactions (dossier 10).

**Comparison (assessment).** What unites them is what *didn't* change: SQL semantics survived every move — DoorDash kept the Postgres wire protocol, Pinterest kept a MySQL dialect, F1 kept SQL with joins. NewSQL's promise is precisely that the transition is topological, not semantic: the risk profile is a resharding project, not a data-model rewrite. That is why this family absorbs exits from both relational and wide-column.

### In-memory: three deployments, one lifecycle

Netflix's EVCache is a product-grade layer: zone-aware, replicated memcached semantics, 2–9 copies per cluster, "most of the requests are served by the cache" ([Netflix EVCache wiki](https://github.com/Netflix/EVCache/wiki)). Facebook's Memcache is the science: the NSDI'13 paper on arguably the world's largest memcached deployment, engineering consistency, replication and load balancing around a soft-state store that "may lose or evict data without notice" ([USENIX NSDI'13](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala)). Twitter's Twemcache is the third datapoint: hundreds of cache servers keeping over 20 TB in memory for 30+ services, ~2 trillion queries a day (over 23M QPS) — a hardened fork of memcached, fronted by the Twemproxy connection-multiplexing proxy ([Twitter Engineering](https://blog.x.com/engineering/en_us/a/2012/caching-with-twemcache)).

**Comparison (assessment).** The three deployments trace the same lifecycle: plain memcached → a hardened fork or wrapper (Twemcache, EVCache) → a proxy layer (Twemproxy). Nobody runs vanilla memcached at these scales; the family's real-world unit is "memcached plus your ops wrapper." The deeper split is who owns correctness: Netflix treats the cache as infrastructure with its own redundancy; Facebook's paper made the *application* own invalidation around a best-effort store. The fit test is unchanged: the authoritative store must survive the cache being empty, and the correctness model must survive it being stale.

### Vector: the index-placement question

turbopuffer is the durable, managed pole: object storage with SSD cache made Readwise's shelved feature shippable — vector search at $20k+/month on incumbent architectures became viable at a fraction ([turbopuffer](https://turbopuffer.com/blog/turbopuffer)) — and Cursor's migration cut search costs ~95% ([Latent Space](https://www.latent.space/p/turbopuffer)). Spotify's Voyager is the stateless pole: an in-memory approximate-nearest-neighbor *library* (HNSW), ~10× faster than Annoy at similar recall, "used extensively in production at Spotify... queried hundreds of millions of times per day," deployed statelessly on Kubernetes to avoid maintaining a stateful database cluster ([Spotify Engineering](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library), [GitHub](https://github.com/spotify/voyager), [InfoQ](https://www.infoq.com/news/2023/11/spotify-ann-voyager/)).

**Comparison (assessment).** The two 2020s answers diverge on exactly one axis: where the index lives. If vectors are a *feature* of a bigger serving system, a library keeps operations near zero; if retrieval is the *product*, with filtering, hybrid search and durability, a vector engine earns its keep. Both companies independently rejected the heavyweight middle ground — which is the convergence thesis of the companion report showing up in production choices. And the unwind flow is now documented in both directions: Confident AI replaced Pinecone with pgvector and measured better accuracy and QPS on the same compute ([Confident AI](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector)), while Pinecone's own blog documents users arriving after hitting pgvector's limits ([Pinecone](https://www.pinecone.io/blog/pinecone-vs-pgvector/)) — the family's honest verdict is workload-dependence, not direction.

# The transition matrix: documented moves between families

*Which moves between families have strong public cases, and which don't. "How it went" is sourced; where it is my judgement, it says (assessment).*

| From → To | Case | What triggered it | How it went |
|---|---|---|---|
| Relational → key-value | Amazon Financial Ledger: Oracle → DynamoDB (dossier 2) | 120 TB ledger; cost, latency | 70% cost cut, 40% latency cut; savepoint friction in the wider Oracle exit |
| Relational → wide-column | Netflix: Oracle → Cassandra (dossier 3); Twitter: MySQL → Cassandra for timelines (dossier 12) | the 2008 shipping-stopping outage; timeline write scale | both right for their time; both later left Cassandra-class ops behind |
| Relational → distributed SQL | DoorDash: Aurora Postgres → CockroachDB | single-primary write bottleneck at weekly peaks; ≤500 GB Postgres table guidance | three cutover attempts; final cutover "lossless"; alert volume fell despite dozens more clusters ([DoorDash](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/), [The New Stack](https://thenewstack.io/how-doordash-migrated-from-aurora-postgres-to-cockroachdb/)) |
| Sharded relational → distributed SQL | Google: sharded MySQL → F1/Spanner (dossier 10); Pinterest: NoSQL/HBase → TiDB (dossier 9) | "never reshard again"; HBase ops burden | gold standard; and the most honest public re-evaluation |
| Postgres-based OLAP → columnar | Cloudflare: CitusDB → ClickHouse | analytics at 6M req/s outgrew the Postgres pipeline | exemplary — and still produced the billing regression years later (post-mortem 30) |
| Search engine → columnar | Cloudflare: Elasticsearch → ClickHouse (logs) | log volume and cost; immutable data fits columnar | "much smaller" cluster ([Cloudflare blog](https://blog.cloudflare.com/log-analytics-using-clickhouse/)) |
| Wide-column → wide-column | Discord: Cassandra → ScyllaDB (dossier 13) | GC pauses, hot partitions, node count | exemplary: 177 → 72 nodes, 3 months → 9 days |
| Wide-column → LSM-under-MySQL | Facebook Messenger: HBase → MyRocks | operating the HBase stack | exemplary ([VLDB 2020](https://www.vldb.org/pvldb/vol13/p3217-matsunobu.pdf)) |
| Wide-column → transactional KV | Apple: Cassandra → FoundationDB (reported) | iCloud/CloudKit scale and transactional needs | press-reported since 2015; Apple still ships Cassandra patches — treat as direction, not completed fact ([The Dulin Report](https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/)) |
| Document → key-value | Mixmax: MongoDB → DynamoDB | ops burden of self-run replica sets | clean and well-documented at small scale ([Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb)) |
| RDBMS → RDBMS | Uber: Postgres → MySQL (dossier 14) | write-amplification claims + team skill | outcome fine; the published justification aged badly |
| Managed object storage → own | Dropbox: S3 → Magic Pocket (dossier 16) | cost at exabyte scale | right at their scale; deliberately reversible |
| Time-series → back again | NexusTrade: MongoDB → TimescaleDB → MongoDB | "purpose-built" performance assumption | failed within two weeks; reversed, ending cheaper than before (post-mortem 31) |
| Search/vector → object-storage-first vector | Cursor → turbopuffer; Readwise shelved-then-shipped | vector storage would cost 4× the entire relational bill | ~95% cost cut; feature viability restored ([turbopuffer](https://turbopuffer.com/blog/turbopuffer), [Latent Space](https://www.latent.space/p/turbopuffer)) |
| Document → wide-column | Discord: MongoDB → Cassandra for messages (dossier 13) | messages needed scalable linear writes | worked for six years until Cassandra's GC pain forced the ScyllaDB move |
| Engine swap *inside* a wide-column engine | Instagram: "Rocksandra" — RocksDB storage engine inside Cassandra ([The New Stack](https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/)) | JVM GC limits at feed scale | same-protocol engine transplant — MyRocks' pattern inside a NoSQL engine |
| Metrics stack → own time-series platform | Uber: Graphite/Whisper → M3; Netflix: Epic → Atlas | cardinality explosion at scale | both built and open-sourced; both then had to rebuild the query/alert layer (M3's DAG engine; Atlas streaming eval) |
| ANN library → ANN library | Spotify: Annoy → Voyager (HNSW) ([Spotify Engineering](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library)) | Annoy's performance no longer adequate; hnswlib bugs; JVM needs | ~10× faster at similar recall; stateless deployments ([InfoQ](https://www.infoq.com/news/2023/11/spotify-ann-voyager/)) |
| Document → relational (the "quiet return") | slice (fintech): MongoDB → PostgreSQL; Infisical: MongoDB → PostgreSQL | ACID for money flows; one unified SQL+JSONB platform | first-party returns exist — but only from small/mid-size teams; the big-tech reversal remains undocumented ([slice engineering](https://engineering.slice.bank.in/seamlessly-migrating-from-mongodb-to-postgresql-leveraging-data-platform-for-scalable-success-404e2577a47b?gi=dd706d6a831e), [Infisical](https://infisical.com/blog/postgresql-migration-technical)) |
| Dedicated vector → engine-embedded (pgvector) | Confident AI: Pinecone → pgvector | fragmented data sources; accuracy + QPS on the same compute | outperformed Pinecone on their workload; the reverse flow is also vendor-documented — Pinecone publishes users arriving after hitting pgvector limits, so this move is genuinely workload-dependent ([Confident AI](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector), [Pinecone](https://www.pinecone.io/blog/pinecone-vs-pgvector/)) |

**Comparing within the busiest moves (assessment).** Three clusters of cases are worth reading side by side:

- **Relational → distributed SQL, three eras, three answers.** Google *built* (2012: no product existed), Alibaba *built* (2013–19: peak transaction rates nothing procurable could meet), DoorDash and Pinterest *bought* (2022–23: converged products exist). The same requirement set — transactions plus horizontal scale — produced two builds and two purchases purely because the market matured between them. This is the decision framework's "re-check the market before building" rule, demonstrated chronologically.
- **The engine-swap twins.** Discord (Cassandra→ScyllaDB), Instagram (Rocksandra inside Cassandra), Facebook (InnoDB→MyRocks under MySQL): all three kept the protocol, schema and operations stack, and replaced only storage internals; all three cite the same motivation class — predictable latency, GC pauses, or space/write amplification; all three kept rollback paths. It is the lowest-blast-radius move in the matrix, and its repeated independent invention is the strongest signal that "swap the engine, keep the database" deserves a place in every team's option set.
- **The build-your-time-series twins.** Uber's M3 and Netflix's Atlas both replaced Graphite/Epic-class systems, both open-sourced, and both then discovered the *second* mountain: the query and alerting layer, which is where M3 built a DAG query engine and Netflix moved alerting to streaming evaluation after the 45-minute-late-alerts incident. Compare with the one small-scale case in the corpus (NexusTrade, post-mortem 31): the same family punishes the buyer who over-scales and the builder who under-scales.

**The gaps (assessment).** Both gaps flagged earlier are now *partially* filled — with honest scale caveats. Document → relational returns are now first-party documented, but only by small and mid-size teams (slice, Infisical): the big-tech reversal remains undocumented, which is itself the finding — the survivors who blog are the ones whose return was cheap enough to survive blogging. The vector unwind is documented in both directions: Confident AI moved Pinecone → pgvector and reports better accuracy and QPS on the same compute ([Confident AI](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector)); press reporting cites 40–60% cost reductions for Pinecone → pgvector at 10–100M vectors ([Java Code Geeks](https://www.javacodegeeks.com/2026/05/vector-databases-when-pgvector-beats-pinecone-and-when-it-doesnt.html)); and Pinecone's own engineering blog documents the counter-flow of users arriving *from* pgvector ([Pinecone](https://www.pinecone.io/blog/pinecone-vs-pgvector/)). The honest verdict: this move is genuinely workload-dependent, not directional. What remains missing from the public record: a first-party, large-scale (100M+ vector) unwind of a dedicated vector store back into an embedded engine — watch-list.

# Mistakes and post-mortems: what the failures teach

*The brief said it plainly: we learn from mistakes. Every case below is real and sourced; the categories (wrong choice, wrong transition, wrong implementation, wrong operations) are mine, and each entry ends with the transferable lesson.*

### 23. Foursquare — the shard nobody was watching (2010) — wrong implementation

**What happened.** Foursquare's MongoDB check-ins database went down for 11 hours. One shard had grown far larger than the other, the working set no longer fit in RAM, and — the detail that made the case canonical — adding a third shard did not free memory, because MongoDB's 4 KB pages held ~300-byte check-in documents, so every page fetched from disk carried mostly dead weight ([InfoQ](https://www.infoq.com/news/2010/10/4square_mongodb_outage/), [archived Foursquare blog](https://web.archive.org/web/20131114075609/http://blog.foursquare.com/2010/10/05/so-that-was-a-bummer/), [High Scalability](http://highscalability.com/blog/2010/10/15/troubles-with-sharding-what-can-we-learn-from-the-foursquare.html)). The team's post-mortem thread walks the sequence in detail ([mongodb-user Google Group](https://groups.google.com/g/mongodb-user/c/UoqU8ofp134)).

**Why it matters here.** Foursquare had moved to MongoDB *from* PostgreSQL — their engineering lead's own later account ([Hacker News](https://news.ycombinator.com/item?id=3203006)) — so this is less "wrong engine chosen" than "distribution adopted without the monitoring and capacity discipline distribution requires."

**Lesson.** Shard-level metrics are not optional; the page-size-to-document-size ratio is a storage-design input; and "add a node" is not a capacity plan when the data is unevenly distributed. The engine survived the story; the operational discipline became the industry's.

### 24. GitLab — the backup that never ran (2017) — wrong operations

**What happened.** During a fix for replication lag, an engineer deleted the primary database directory (~300 GB) on the wrong side of the replication pair. The five backup mechanisms then failed one by one: pg_dump backups had been silently failing for weeks (9.2-era binaries against a 9.6 database), and no disk snapshots existed. Six hours of user data (issues, comments) were permanently lost; full service took roughly 18 hours to restore ([GitLab post-mortem](https://about.gitlab.com/blog/gitlab-dot-com-database-incident/), [Availability Digest analysis, PDF](https://www.availabilitydigest.com/public_articles/1204/gitlab.pdf)).

**Lesson.** A backup you have never restored is a hope, not a backup — and one unverified assumption in the chain (here: version-matched backup tooling) invalidates the whole chain. GitLab's response also became the template for post-incident transparency: full timeline, published restoration attempts, blameless process fixes.

### 25. GitHub — 43 seconds of partition, 24 hours of degradation (2018) — wrong failover model

**What happened.** A 43-second network partition between the US East and West coasts triggered MySQL failovers; Orchestrator briefly promoted a new primary on the wrong coast, and the resulting topology routed application traffic across the country. GitHub chose to fail forward — preserving ~30 minutes of West Coast writes — accepting 24h11m of degraded availability ([GitHub post-incident analysis](https://github.blog/news-insights/company-news/oct21-post-incident-analysis/)).

**Lesson.** Failover automation must be tested against partitions, not just dead nodes: split-brain during a *brief* partition is the case the tooling misses. And GitHub's choice is the cleanest public demonstration that consistency vs. availability is a real-time trade with a price on both sides — they paid a day of degraded service to avoid half an hour of data loss, which for a code host is the right call (assessment).

### 26. Danger / T-Mobile Sidekick — no backup at all (2009) — wrong trust

**What happened.** A SAN upgrade corrupted the service's data; roughly 800,000 users lost contacts, calendars and photos. Microsoft's Danger subsidiary had no verified working backup; some data was restored from month-old offsite tapes. The event was called the biggest disaster in the then-young history of cloud computing ([Wikipedia](https://en.wikipedia.org/wiki/2009_Sidekick_data_loss), [Data Center Knowledge](https://www.datacenterknowledge.com/archives/2009/10/10/t-mobile-microsoft-lost-all-the-sidekick-backups)).

**Lesson.** Trusting a platform's or vendor's durability story without a verification path of your own is not a backup strategy. Sidekick is the canonical case that taught consumers that "the cloud" is somebody else's operations.

### 27. Ma.gnolia — the backup was on the same broken stack (2009) — wrong architecture

**What happened.** The social bookmarking service lost all user data when its ~0.5 TB MySQL database corrupted and the backup copies failed with it; the primary and its backups went down together, and user data was never recovered ([WIRED](https://www.wired.com/2009/01/magnolia-suffer/), [Data Center Knowledge](https://www.datacenterknowledge.com/archives/2009/02/19/magnolia-data-is-gone-for-good), [Wikipedia](https://en.wikipedia.org/wiki/Gnolia)).

**Lesson.** Copies on the same engine, same host, or same failure domain are not redundancy. Restore-path diversity — different engine, different site, different failure domain — is the actual deliverable.

### 28. Code Spaces — the backup was inside the blast radius (2014) — wrong security architecture

**What happened.** An attacker gained the AWS console credentials of the code-hosting company and, when staff tried to reclaim control, deleted the EBS snapshots, S3 data and AMIs. The company folded within days. The "backup plan" was real but shared the same single point of compromise as production ([Ars Technica](https://arstechnica.com/information-technology/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/)).

**Lesson.** Backups must sit outside the compromise domain of production: separate accounts, separate credentials, deletion that requires more than one human or one key. Infrastructure-as-code cuts both ways — automatable creation means automatable destruction.

### 29. AWS S3 — the typo'd runbook (2017) — wrong process controls

**What happened.** A command executed per an established playbook removed far more subsystem servers than intended; the index and placement subsystems fell below safe capacity and S3 in us-east-1 was fully unavailable for roughly four hours, cascading through every service built on it. S&P 500 companies were estimated to have lost ~$150M — and AWS's own status page, hosted on S3, could not update ([AWS post-mortem](https://aws.amazon.com/message/41926/), [NPR](https://www.npr.org/sections/thetwo-way/2017/03/03/518322734/amazon-and-the-150-million-typo)).

**Lesson.** Trusted runbooks are code and need the same guardrails: rate limits, safety interlocks, re-validation after every change. Status pages must live outside the system they monitor — an incident-communication single point of failure compounds every other failure.

### 30. Cloudflare — the migration regression that hid itself (2025) — wrong verification layer

**What happened.** After a ClickHouse migration, the daily aggregation jobs that produce Cloudflare's bills slowed dramatically — while I/O, memory, rows scanned and parts read all looked normal. The bottleneck turned out to be lock contention inside ClickHouse's query planner, invisible in the usual hardware metrics; Cloudflare diagnosed it and contributed fixes upstream ([Cloudflare blog](https://blog.cloudflare.com/clickhouse-query-plan-contention/)).

**Lesson.** Post-migration verification must profile the *new engine's* internals, not just the workload's hardware counters: the bottleneck moves to the layer nobody was watching. The exemplary part is the response — root-cause to the lock, patch upstream, then run the table migration with Merge tables so old and new coexist during the transition.

### 31. NexusTrade — the two-week round trip (2025) — wrong transition

**What happened.** A solo-built trading platform migrated its historical backtest data from MongoDB to TimescaleDB on the assumption that a "purpose-built" time-series database must be faster. Continuous aggregates and columnar compression — the headline features — fought the workload instead, and the author migrated back within two weeks, this time to MongoDB's native time-series collections (which they had not known existed), ending cheaper than before ([NexusTrade](https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003)).

**Lesson.** "Purpose-built" is marketing until measured on your own queries — and the engine's headline features can be exactly what the workload cannot afford. The recovery was possible only because the old path was kept intact: the same reversibility rule the big migrations obey. The report is unusually honest about a failure most would never publish, which is exactly why it belongs here.

### The pattern across all failures (assessment)

- **Half the catastrophic cases are backup failures, not database failures** (GitLab, Sidekick, Ma.gnolia, Code Spaces). Backup architecture deserves the same rigor as schema design: independent failure domains, restore-tested, deletion-protected.
- **Distribution creates failure modes that scale-up systems never had.** Unmonitored shard growth (Foursquare) and failover split-brain (GitHub) are *categories*, not one-offs; adopting distribution means adopting its failure model.
- **Migrations regress in the unprofiled layer** (Cloudflare): budget for post-cutover profiling with the new engine's own tooling, not the old dashboards.
- **Wrong transitions are survivable only with reversibility preserved** (NexusTrade, plus Dropbox's bidirectional replication, MyRocks' InnoDB rollback replicas, Slack's read-diffing). Every case in this report that kept a way home either used it or was glad it existed.
- **The mistakes already inside the dossiers are the same species:** Slack's workspace-per-shard key (design), Uber's engine-blaming justification (communication), Pinterest's technologies that "broke catastrophically" (choice), Netflix's 2008 single box (architecture), Amazon's savepoint timeouts (implementation). Storage engineering's failure modes are a small set wearing different logos.

# Open questions

- **Unaudited figures.** Cost reductions (Amazon's "70%", Financial Ledger claims) are vendor-reported; no case publishes a like-for-like TCO.
- **Post-migration drift.** Several architectures have evolved since their canonical blog post (Netflix's stack today, Twitter post-2022, Uber's current Schemaless status); current-state claims here stop at the cited dates.
- **Undisclosed motives.** License pressure (Amazon, Alibaba) and vendor rivalry color the public record; the technical record cannot be fully separated from the commercial one.
- **Survivor gap.** Failed migrations are rarely blogged; the success-shaped sample overstates the ease of e.g. Oracle exits (Amazon's savepoint timeouts are a rare public crack).
- **The NewSQL wave is young.** Pinterest's TiDB adoption is the largest public "regret the build, buy the product" datapoint; its long-term outcome is not yet written.

# Recommendations / next steps

1. **If choosing today**: default to a converged engine (Postgres/pgvector-class, MySQL/Vitess, or a distributed SQL product if multi-region transactions are real requirements) and reserve NoSQL for genuinely availability-first, query-simple workloads.
2. **If migrating**: copy the playbook above; budget as much for the diff/verification tooling as for the data move itself — the cases agree this is where migrations die.
3. **If pressured to build**: require a written ops-cost forecast at 3× current scale (the Twitter/Discord test), an open-source or platform plan (the Vitess/MyRocks test), and an engine-agnostic abstraction layer (the Panda test).
4. **Deeper reading in order**: the Dynamo paper (still the best statement of availability-first design), the F1/Spanner papers (consistency-first), Discord's trillions-of-messages post (migration craft), and Nick Craver's Stack Overflow series (restraint).

---

# Source notes

| Source | Credibility | Last updated |
|---|---|---|
| [DeCandia et al., "Dynamo: Amazon's Highly Available Key-value Store" (SOSP 2007 PDF)](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) | 5/5 | Oct 2007 |
| [AWS News Blog — Migration Complete: Amazon's final Oracle database](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/) | 5/5 | Oct 2019 |
| [CIO Dive — Amazon completes Oracle migration](https://www.ciodive.com/news/amazon-kisses-last-oracle-database-goodbye-completes-years-long-migration/565073/) | 3/5 | Oct 2019 |
| [InfoWorld — Netflix embraces NoSQL in the cloud](https://www.infoworld.com/article/2171162/big-movies-big-data-netflix-embraces-nosql-in-the-cloud.html) | 3/5 | 2011 |
| [SlideShare — Migrating Netflix from Datacenter Oracle to Global Cassandra](https://www.slideshare.net/slideshow/migrating-netflix-from-oracle-to-global-cassandra/8600029) | 3/5 | - |
| [SlideShare — Cassandra from the trenches: migrating Netflix](https://www.slideshare.net/jasedbrown/cassandra-from-the-trenches-migrating-netflix-10586521) | 3/5 | - |
| [Twitter Engineering — Manhattan, our real-time multi-tenant distributed database](https://blog.x.com/engineering/en_us/a/2014/manhattan-our-real-time-multi-tenant-distributed-database-for-twitter-scale) | 4/5 | 2014 |
| [Twitter Engineering — Strong consistency in Manhattan](https://blog.x.com/engineering/en_us/a/2016/strong-consistency-in-manhattan) | 4/5 | 2016 |
| [InfoQ — Twitter's Manhattan](https://www.infoq.com/news/2014/05/twitters-manhattan/) | 4/5 | 2014 |
| [Yugabyte — The distributed database behind Twitter (2020 summit recap)](https://www.yugabyte.com/blog/recap-the-distributed-database-behind-twitter/) | 3/5 | 2020 |
| [Vitess docs — History](https://vitess.io/docs/21.0/overview/history/) | 5/5 | - |
| [Slack Engineering — Scaling datastores at Slack with Vitess](https://slack.engineering/scaling-datastores-at-slack-with-vitess/) | 5/5 | 2020 |
| [Hacker News — discussion of Slack's Vitess migration ("original sin" comment)](https://news.ycombinator.com/item?id=27199229) | 2/5 | 2021 |
| [GitHub Blog — Partitioning GitHub's relational databases to handle scale](https://github.blog/engineering/infrastructure/partitioning-githubs-relational-databases-scale/) | 5/5 | 2021 |
| [GitHub Blog — Upgrading GitHub.com to MySQL 8.0](https://github.blog/engineering/infrastructure/upgrading-github-com-to-mysql-8-0/) | 5/5 | 2023 |
| [Shopify Engineering — Shard balancing at terabyte scale](https://shopify.engineering/mysql-database-shard-balancing-terabyte-scale) | 5/5 | - |
| [Instagram Engineering — Sharding & IDs at Instagram](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c) | 5/5 | Dec 2011 |
| [Pinterest Engineering — Sharding Pinterest: how we scaled our MySQL fleet](https://medium.com/pinterest-engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f) | 5/5 | - |
| [Pinterest Engineering — TiDB adoption at Pinterest](https://medium.com/pinterest-engineering/tidb-adoption-at-pinterest-1130ab787a10) | 5/5 | - |
| [Google Cloud Blog — Reflecting on the Spanner paper](https://cloud.google.com/blog/products/databases/reflecting-on-spanner-papers-sigops-hall-of-fame-award) | 5/5 | 2022 |
| [the morning paper — Spanner: Google's globally-distributed database](https://blog.acolyer.org/2015/01/08/spanner-googles-globally-distributed-database/) | 4/5 | 2015 |
| [Murat Demirbas — F1: a distributed SQL database that scales](http://muratbuffalo.blogspot.com/2023/07/f1-distributed-sql-database-that-scales.html) | 3/5 | Jul 2023 |
| [Database of Databases — OceanBase](https://dbdb.io/db/oceanbase/revisions/24) | 3/5 | - |
| [Alibaba Cloud — ApsaraDB for OceanBase case studies](https://www.alibabacloud.com/help/en/apsaradb-for-oceanbase/latest/case-studies) | 4/5 | - |
| [Uber Engineering — Why Uber Engineering switched from Postgres to MySQL](https://www.uber.com/us/en/blog/postgres-to-mysql-migration/) | 5/5 | 2016 |
| [Ayende — re: Why Uber Engineering switched from Postgres to MySQL](https://ayende.com/blog/175137/re-why-uber-engineering-switched-from-postgres-to-mysql) | 3/5 | 2016 |
| [Discord — How Discord stores trillions of messages](https://discord.com/blog/how-discord-stores-trillions-of-messages) | 5/5 | 2023 |
| [InfoQ — Discord migrates trillions of messages from Cassandra to ScyllaDB](https://www.infoq.com/news/2023/06/discord-cassandra-scylladb/) | 4/5 | Jun 2023 |
| [Dropbox Tech — Scaling to exabytes and beyond (Magic Pocket)](https://dropbox.tech/infrastructure/magic-pocket-infrastructure) | 5/5 | Mar 2016 |
| [Dropbox Tech — Inside the Magic Pocket](https://dropbox.tech/infrastructure/inside-the-magic-pocket) | 5/5 | - |
| [InfoQ — Dropbox develops Magic Pocket, moves away from AWS](https://www.infoq.com/news/2016/03/Dropbox-AWS/) | 4/5 | Mar 2016 |
| [Data Center Dynamics — How Dropbox pulled off its hybrid cloud transition](https://www.datacenterdynamics.com/en/analysis/how-dropbox-pulled-off-its-hybrid-cloud-transition/) | 3/5 | - |
| [Dropbox Tech — (Re)Introducing Edgestore](https://dropbox.tech/infrastructure/reintroducing-edgestore) | 5/5 | - |
| [Dropbox Tech — Future-proofing our metadata stack with Panda](https://dropbox.tech/infrastructure/panda-metadata-stack-petabyte-scale-transactional-key-value-store) | 5/5 | - |
| [Dropbox Tech — Alki, or how we learned to stop worrying and love cold metadata](https://dropbox.tech/infrastructure/alki--or-how-we-learned-to-stop-worrying-and-love-cold-metadata) | 5/5 | - |
| [Dropbox Tech — Cross-shard transactions at 10M requests per second](https://dropbox.tech/infrastructure/cross-shard-transactions-at-10-million-requests-per-second) | 5/5 | - |
| [Meta Engineering — MyRocks: a space- and write-optimized MySQL database](https://engineering.fb.com/2016/08/31/core-infra/myrocks-a-space-and-write-optimized-mysql-database/) | 5/5 | Aug 2016 |
| [Meta Engineering — Migrating a database from InnoDB to MyRocks](https://engineering.fb.com/2017/09/25/core-infra/migrating-a-database-from-innodb-to-myrocks/) | 5/5 | Sep 2017 |
| [VLDB 2020 — MyRocks: LSM-tree database storage engine serving Facebook's social graph](https://www.vldb.org/pvldb/vol13/p3217-matsunobu.pdf) | 5/5 | 2020 |
| [LinkedIn Engineering — Introducing Espresso](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store) | 5/5 | - |
| [Nick Craver — What it takes to run Stack Overflow (2013)](https://nickcraver.com/blog/2013/11/22/what-it-takes-to-run-stack-overflow/) | 5/5 | Nov 2013 |
| [Nick Craver — Stack Overflow: The Architecture, 2016 edition](https://nickcraver.com/blog/2016/02/17/stack-overflow-the-architecture-2016-edition/) | 5/5 | Feb 2016 |
| [37signals dev — Recordables / delegated types (majestic monolith discussion)](https://dev.37signals.com/the-rails-delegated-type-pattern/) | 4/5 | - |
| [Cloudflare Blog — Zero-latency SQLite storage in every Durable Object](https://blog.cloudflare.com/sqlite-in-durable-objects/) | 5/5 | - |
| [Cloudflare Docs — Durable Objects overview](https://developers.cloudflare.com/durable-objects/) | 5/5 | - |
| [Stripe — How Stripe's document databases supported 99.999% uptime with zero-downtime data migrations](https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations) | 5/5 | - |
| [Mixmax — The Great Migration: from MongoDB to DynamoDB](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb) | 4/5 | - |
| [PlanetScale — Horizontal sharding for MySQL made easy (Vitess adopter context)](https://planetscale.com/blog/horizontal-sharding-for-mysql-made-easy) | 3/5 | 2020 |
| [InfoQ — Foursquare's 11-hour MongoDB outage](https://www.infoq.com/news/2010/10/4square_mongodb_outage/) | 4/5 | Oct 2010 |
| [Foursquare blog (archived) — "So that was a bummer"](https://web.archive.org/web/20131114075609/http://blog.foursquare.com/2010/10/05/so-that-was-a-bummer/) | 4/5 | Oct 2010 |
| [High Scalability — Troubles with sharding: the Foursquare lesson](http://highscalability.com/blog/2010/10/15/troubles-with-sharding-what-can-we-learn-from-the-foursquare.html) | 3/5 | Oct 2010 |
| [mongodb-user Google Group — Foursquare outage post-mortem thread](https://groups.google.com/g/mongodb-user/c/UoqU8ofp134) | 3/5 | Oct 2010 |
| [Hacker News — Foursquare engineering lead on the PostgreSQL→MongoDB move](https://news.ycombinator.com/item?id=3203006) | 2/5 | 2012 |
| [GitLab — post-mortem: the gitlab.com database incident](https://about.gitlab.com/blog/gitlab-dot-com-database-incident/) | 5/5 | Feb 2017 |
| [Availability Digest — GitLab incident analysis (PDF)](https://www.availabilitydigest.com/public_articles/1204/gitlab.pdf) | 3/5 | 2017 |
| [GitHub Blog — October 21, 2018 post-incident analysis](https://github.blog/news-insights/company-news/oct21-post-incident-analysis/) | 5/5 | 2018 |
| [Wikipedia — 2009 Sidekick data loss](https://en.wikipedia.org/wiki/2009_Sidekick_data_loss) | 3/5 | - |
| [Data Center Knowledge — T-Mobile/Microsoft lost all the Sidekick backups](https://www.datacenterknowledge.com/archives/2009/10/10/t-mobile-microsoft-lost-all-the-sidekick-backups) | 3/5 | Oct 2009 |
| [WIRED — Ma.gnolia suffers catastrophic data loss](https://www.wired.com/2009/01/magnolia-suffer/) | 3/5 | Jan 2009 |
| [Data Center Knowledge — Ma.gnolia data is gone for good](https://www.datacenterknowledge.com/archives/2009/02/19/magnolia-data-is-gone-for-good) | 3/5 | Feb 2009 |
| [Wikipedia — Ma.gnolia](https://en.wikipedia.org/wiki/Gnolia) | 2/5 | - |
| [Ars Technica — AWS console breach leads to demise of Code Spaces](https://arstechnica.com/information-technology/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/) | 4/5 | Jun 2014 |
| [AWS — Summary of the Amazon S3 service disruption (message 41926)](https://aws.amazon.com/message/41926/) | 5/5 | Feb 2017 |
| [NPR — Amazon and the $150 million typo](https://www.npr.org/sections/thetwo-way/2017/03/03/518322734/amazon-and-the-150-million-typo) | 3/5 | Mar 2017 |
| [DevPro Journal — Apache Cassandra 4.0 release (largest-deployment figures)](https://www.devprojournal.com/technology-trends/open-source/whats-included-in-the-cassandra-4-0-release-2/) | 3/5 | Jul 2021 |
| [IT Brief — Cassandra 4.0, used by Apple, Netflix, Amazon and thousands more](https://itbrief.co.nz/story/apache-cassandra-version-4-0-released-used-by-apple-netflix-amazon-and-thousands-more) | 2/5 | Jul 2021 |
| [Engineers' Codex — How Apple built iCloud to store billions of databases](https://read.engineerscodex.com/p/how-apple-built-icloud-to-store-billions) | 2/5 | - |
| [The Dulin Report — Apple is (or was) the biggest user of Apache Cassandra](https://thedulinreport.com/2015/04/23/apple-is-or-was-the-biggest-user-of-apache-cassandra/) | 2/5 | Apr 2015 |
| [DoorDash — How we scaled New Verticals Fulfillment Backend with CockroachDB](https://careersatdoordash.com/blog/how-we-scaled-new-verticals-fulfillment-backend-with-cockroachdb/) | 5/5 | 2024 |
| [The New Stack — How DoorDash migrated from Aurora Postgres to CockroachDB](https://thenewstack.io/how-doordash-migrated-from-aurora-postgres-to-cockroachdb/) | 3/5 | Dec 2023 |
| [Cockroach Labs — Why DoorDash migrated from Aurora Postgres to CockroachDB (vendor)](https://www.cockroachlabs.com/blog/aurora-postgres-to-cockroachdb/) | 3/5 | 2023 |
| [Cloudflare Blog — HTTP Analytics for 6M requests per second using ClickHouse](https://blog.cloudflare.com/http-analytics-for-6m-requests-per-second-using-clickhouse/) | 5/5 | 2019 |
| [Cloudflare Blog — Log analytics using ClickHouse](https://blog.cloudflare.com/log-analytics-using-clickhouse/) | 5/5 | 2020 |
| [Cloudflare Blog — Our billing pipeline was suddenly slow (ClickHouse query-plan contention)](https://blog.cloudflare.com/clickhouse-query-plan-contention/) | 5/5 | 2025 |
| [ClickHouse Blog — Cloudflare case: analytics at quadrillion-row scale (vendor)](https://clickhouse.com/blog/cloudflare) | 3/5 | 2025 |
| [Netflix — EVCache wiki (distributed memcached-based caching)](https://github.com/Netflix/EVCache/wiki) | 4/5 | - |
| [Computer Weekly — Panama Papers revealed by graph database software](https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software) | 4/5 | 2016 |
| [Neo4j Blog — How the ICIJ used Neo4j to unravel the Panama Papers (vendor)](https://neo4j.com/blog/cypher-and-gql/icij-neo4j-unravel-panama-papers/) | 3/5 | 2016 |
| [Neo4j case study — Zurich Insurance fraud investigation, PDF (vendor)](https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf) | 3/5 | - |
| [Uber Engineering — M3: large-scale metrics platform for Prometheus](https://www.uber.com/us/en/blog/m3/) | 5/5 | 2018 |
| [Uber Engineering — The Billion Data Point Challenge](https://www.uber.com/us/en/blog/billion-data-point-challenge/) | 5/5 | 2019 |
| [Packt — Uber open-sources its metrics platform M3](https://www.packtpub.com/en-in/learning/tech-news/uber-open-sources-its-large-scale-metrics-platform-m3-for-prometheus) | 2/5 | 2018 |
| [turbopuffer — Inaugural blog post (Readwise cost case, architecture)](https://turbopuffer.com/blog/turbopuffer) | 4/5 | 2023 |
| [AI Engineer — turbopuffer profile (customers, scale figures)](https://ai.engineer/orgs/turbopuffer) | 3/5 | 2026 |
| [Latent Space — turbopuffer: retrieval after RAG (Cursor 95% cost cut)](https://www.latent.space/p/turbopuffer) | 3/5 | 2025 |
| [MongoDB Blog — eBay: mission-critical multi-data-center applications (vendor)](https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb) | 3/5 | - |
| [NexusTrade — I went through hell and back migrating to TimescaleDB](https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003) | 2/5 | Oct 2025 |
| [AWS / YouTube — Duolingo stores 31 billion items on Amazon DynamoDB (Karin Tsai, Duolingo)](https://www.youtube.com/watch?v=fhnAvn2YxZA) | 3/5 | - |
| [Duolingo Blog — Reducing cloud spending (DynamoDB cost tuning)](https://blog.duolingo.com/reducing-cloud-spending/) | 4/5 | 2024 |
| [MongoDB — MongoDB at scale (EA, Sailthru; vendor)](https://www.mongodb.com/products/capabilities/mongodb-scale) | 2/5 | - |
| [ClickHouse Blog — Large-scale financial market analytics with ClickHouse (Bloomberg meetup report)](https://clickhouse.com/blog/nyc-meetup-report-large-scale-financial-market-analytics-with-clickhouse) | 3/5 | Dec 2022 |
| [Wikipedia — ClickHouse (CERN LHCb deployment)](https://en.wikipedia.org/wiki/ClickHouse) | 3/5 | - |
| [The New Stack — Instagram supercharges Cassandra with a pluggable RocksDB storage engine](https://thenewstack.io/instagram-supercharges-cassandra-pluggable-rocksdb-storage-engine/) | 3/5 | 2018 |
| [Netflix TechBlog — Introducing Atlas: Netflix's primary telemetry platform](http://techblog.netflix.com/2014/12/introducing-atlas-netflixs-primary.html) | 5/5 | Dec 2014 |
| [Netflix TechBlog — Improved alerting with Atlas Streaming Eval](https://netflixtechblog.com/improved-alerting-with-atlas-streaming-eval-e691c60dc61e) | 5/5 | - |
| [Atlas docs — Overview (Epic history)](https://netflix.github.io/atlas-docs/overview/) | 4/5 | - |
| [USENIX NSDI '13 — Scaling Memcache at Facebook](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala) | 5/5 | 2013 |
| [Spotify Engineering — Introducing Voyager: Spotify's new nearest-neighbor search library](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library) | 5/5 | Oct 2023 |
| [Spotify — Voyager (GitHub: production usage figures)](https://github.com/spotify/voyager) | 5/5 | - |
| [InfoQ — Spotify open-sources Voyager nearest-neighbor search library](https://www.infoq.com/news/2023/11/spotify-ann-voyager/) | 4/5 | Nov 2023 |
| [Neo4j Blog — Machine learning on graphs: interview with eBay's Ajinkya Kale (vendor)](https://neo4j.com/blog/knowledge-graph/machine-learning-on-graphs-ajinkya-kale-senior-applied-researcher-ebay/) | 2/5 | - |
| [Twitter Engineering — Caching with Twemcache](https://blog.x.com/engineering/en_us/a/2012/caching-with-twemcache) | 5/5 | 2012 |
| [slice engineering — Freaking hard migration from MongoDB to PostgreSQL](https://engineering.slice.bank.in/seamlessly-migrating-from-mongodb-to-postgresql-leveraging-data-platform-for-scalable-success-404e2577a47b?gi=dd706d6a831e) | 4/5 | - |
| [Infisical — The Great Migration from MongoDB to PostgreSQL](https://infisical.com/blog/postgresql-migration-technical) | 4/5 | - |
| [Confident AI — Why we replaced Pinecone with PGVector](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector) | 3/5 | - |
| [Pinecone — Pinecone vs. Postgres pgvector (vendor)](https://www.pinecone.io/blog/pinecone-vs-pgvector/) | 2/5 | - |
| [Java Code Geeks — When pgvector beats Pinecone (and when it doesn't)](https://www.javacodegeeks.com/2026/05/vector-databases-when-pgvector-beats-pinecone-and-when-it-doesnt.html) | 2/5 | May 2026 |

**Conflicts and caveats.** Uber's own post and its critics disagree on whether Postgres's engine or Uber's operational inexperience drove the switch; both are cited and the verdict marks the divergence. Amazon's 2019 claims are the vendor's own; CIO Dive's reporting adds the savepoint-timeout frictions. Pinterest's TiDB post evaluates competitors (Vitess, Spanner) in Pinterest's own terms — a one-sided but first-party source. Cloudflare's Durable Objects docs are the vendor's product documentation. Dates marked "-" were not stated on the page. The vendor-published case studies (eBay/MongoDB, Zurich/Neo4j, DoorDash/Cockroach Labs, Cloudflare/ClickHouse, ICIJ's account on Neo4j's blog, turbopuffer's own posts) are marketing-adjacent by nature; the customer-named facts are vendor-reported. Apple's Cassandra→FoundationDB direction is press-reported, not first-party. The Foursquare and NexusTrade accounts are participant self-reports — the most valuable kind for failure analysis, but single-voiced.