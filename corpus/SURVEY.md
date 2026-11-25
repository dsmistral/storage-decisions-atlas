
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

### 22a. Zerodha — single-node PostgreSQL at ~trillion-row scale (2023–present) *(R6 fold-in, session 4)*

**Context.** India's largest stock broker runs its own infrastructure and is emphatic about its choice: self-hosted PostgreSQL on bare EC2 instances, "not moving out of PostgreSQL" ([Zerodha tech: Working with PostgreSQL](https://zerodha.tech/blog/working-with-postgresql/)).

**Choice.** One node per database, mastery overbars: connection discipline, hardware tuned to the workload, and an ephemeral result-cache layer (DungBeetle, open-sourced) that creates ~7 million Postgres tables per day on a dedicated hot-cache instance to serve reports queried from a ~trillion-row database ([Zerodha tech: Working with PostgreSQL](https://zerodha.tech/blog/working-with-postgresql/), [DungBeetle](https://github.com/zerodha/dungbeetle), [FOSS United talk abstract](https://platform.fossunited.org/c/indiafoss/indiafoss24/cfp/oqtefvp0pj)).

**What happened.** Scale-up worked far past the point most teams shard. When one 20 TB database finally needed sharding, it was sharded "in two hours with zero schema changes" — the maturity dividend of deep mastery plus the engine's own tooling (FP).

**Verdict (assessment).** Stack Overflow's kin at Indian-fintech scale, made possible by ops discipline instead of headcount. The DungBeetle pattern is the case where scale problems are outgrown by craft; watch whether it converges into a product (R5 watch W4-adjacent).

### 22b. OpenAI — 800M ChatGPT users on unsharded PostgreSQL (Jan 2026) *(R6 fold-in, session 4)*

**Context.** ChatGPT serves 800M users from one Azure PostgreSQL Flexible Server primary plus nearly 50 read replicas across multiple regions, millions of QPS ([OpenAI: Scaling PostgreSQL](https://openai.com/index/scaling-postgresql/), FP; third-party coverage at PR level: [ByteByteGo](https://blog.bytebytego.com/p/how-openai-scaled-to-800-million), [VentureBeat](https://venturebeat.com/data/how-openai-is-scaling-the-postgresql-database-to-800-million-users)).

**Choice.** Reads scale by replication, not sharding; primary load kept low because a single writer cannot scale writes. One SEV-0 PostgreSQL incident in 12 months: the ImageGen launch, a >10× write surge and 100M new users in one week. Remaining write-heavy workloads are being migrated to sharded systems, and third-party coverage reports a "no new tables in PostgreSQL" rule (watch-list W5).

**Verdict (assessment).** The strongest modern "you may not need to shard yet" datapoint, and the only one that names the wall (writes). Read the "no new tables" line as frontier management reported second-hand, not documented policy.

*R6 fold-in note (session 4, Chapter 1 authoring): cases C54–C66 were registered in R6 without dossiers; dossiers are folded in at authoring time. The two above feed Chapter 1 as the modern counter-weights beside dossier 20; the remaining R6 dossiers fold in as their chapters are authored.*

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

