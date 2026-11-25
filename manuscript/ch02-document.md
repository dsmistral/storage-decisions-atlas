# Chapter 2 - Document: Three Reasons, One Aggregate

**Governing idea.** "We have unstructured data" is the wrong reason to choose a
document database. In the documented record, the reason that survives scrutiny
is access-shape, not structure: the data is organized around an *aggregate*
(a listing, a checkout configuration, a user's profile or game state) that is
read, cached, and written as one unit, and is rarely joined across units. Teams
landed on the family for three different reasons; all three reasons
reduce to the same fit test. The family's build case (Espresso) then shows both
the power and the price of the model, and the quiet returns (T10, T19) show
what happens when the access shape changes (P2, P15).

**Cases in this chapter:** C06 (eBay), C07 (Stripe), C08 (EA/Sailthru), C09
(LinkedIn Espresso); T10 and T19 as the exit-flow preview. Register IDs resolve
in Appendix A; every source URL is copied from `corpus/REGISTER.md`.

---

## 1. Context: the marketing reason and the real reason

The document family is usually introduced by data format: "schemaless," "JSON,
no joins," "for unstructured data." That framing explains almost nothing in the
record. Every case below stores data that is perfectly structured; what varies
is that the structure differs *between* aggregates, and that the unit a user or
service touches is one aggregate, not a join of many.

The real question the four cases answer: when is "the aggregate is the unit of
work" the dominant property of a workload? When it is, one writable JSON-shaped
record beats a host of normalized tables plus cache invalidation. When it is
not, when queries reach *across* aggregates, the same choice becomes the
friction source that Chapter 16's returns document.

## 2. Three reasons, each against its time

### Reason 1: a catalog whose schema varies per item (eBay, C06)

eBay, with 170M+ buyers, 1B+ listings across 190 markets, runs
mission-critical, multi-datacenter applications on MongoDB, which per the
vendor-published account "now also serves System of Record use cases at eBay"
([C06: MongoDB blog](https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb), VP).

The workload logic is easy to reconstruct (assessment): a marketplace listing
is the canonical ragged aggregate. A car listing, a sneaker listing and a
service listing share a spine (seller, price, category, photos) and diverge
everywhere else; forcing them into one relational schema means either a
thousand sparse columns or an attribute-table join per page view. Storing the
listing as one document makes the common case, "render one listing whole," a
single-key read.

Note the evidence level: this is the vendor's account (VP). eBay published no
first-party engineering post in the register for this case, so operational
claims (multi-datacenter behavior, system-of-record suitability) rest on the
vendor's write-up. The chapter treats the fit reasoning as assessment; the
scale figures as vendor-reported.

### Reason 2: durable operational documents under microservices (Stripe, C07)

Stripe's document databases underpin "99.999% uptime" with "zero-downtime data
migrations" ([C07: Stripe](https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations), FP).

Here the reason is not schema flexibility but operational durability of
per-object state (payment intents, subscription state, configuration): the
aggregate is the transactional unit, reads and writes are by aggregate key,
and the engineering priority is to evolve the stored representation without
stopping the writes. The zero-downtime migration discipline Stripe describes
is the P19 playbook applied inside a document store, and it previews Chapter
22's argument that the migration machinery, not the engine, is where the
difficulty lives.

### Reason 3: rapidly evolving user and game state (EA and Sailthru, C08)

EA's Spearhead studio selected MongoDB to store user data and game state for
FIFA Online; Sailthru grew to 125M content documents and 5B messages a month,
40 TB across 120 nodes ([C08: MongoDB at scale](https://www.mongodb.com/products/capabilities/mongodb-scale), VP).

Game state and messaging are the document model at its most literal: the
user (or user-plus-session) is one blob whose internal fields change with
every game release, and whose read pattern is "load my state, mutate, save."
The Sailthru half of the case adds the volume dimension (page-count documents
at message rates), and both are vendor-published (VP), like C06.

### The build: LinkedIn's Espresso (C09)

LinkedIn served online apps from a split stack: "Relational Database
Management Systems (RDBMSs) such as Oracle and key-value stores such as
Voldemort, both serving different use cases" ([C09: LinkedIn Engineering](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store), FP).
Espresso was built to merge the halves: a distributed *document-relational*
store with transactions, secondary indexes, and hash partitioning by primary
key, Oracle's feature set on Voldemort's horizontal scale.

What happened: Espresso became the source of truth for hundreds of terabytes
and some of LinkedIn's most valuable datasets, serving millions of records per
second at peak ([C09: LinkedIn Engineering](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store), FP).

The design lesson in Espresso is the chapter's thesis stated by construction:
the data model follows the access pattern (a user is a document group), not
the other way around. LinkedIn wanted Oracle-class features *because* its
access patterns, read the profile, transact on the profile, index into it,
never stopped being relational-shaped; it wanted Voldemort-class scale because
the write and distribution demands had left one node behind. The document
model was the compromise shape between the two.

The maintenance lesson is P15, and it grades the build a specific way:
Espresso solved an internal need, stayed internal (no open-source release or
productization on record as of the 2012 post that introduced it), and reads as
delivering less than the equivalent 2026 product would, because LinkedIn bears
the full maintenance forever (assessment; P15's build-test dwell continues in
the Ch. 8 evaluation discussion, alongside Manhattan and Schemaless).

## 3. What happened (sourced outcomes, with evidence levels)

- **eBay (C06, VP).** Mission-critical multi-datacenter operation on MongoDB;
  the vendor states it "now also serves System of Record use cases" ([C06: MongoDB blog](https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb)).
- **Stripe (C07, FP).** Document databases carried 99.999% uptime with
  zero-downtime migrations as a first-class capability ([C07: Stripe](https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations)).
- **EA/Sailthru (C08, VP).** FIFA Online user and game state on MongoDB;
  Sailthru at 125M documents, 5B messages/month, 40 TB over 120 nodes ([C08: MongoDB at scale](https://www.mongodb.com/products/capabilities/mongodb-scale)).
- **LinkedIn (C09, FP).** Espresso as source of truth for hundreds of TB,
  millions of records per second at peak ([C09: LinkedIn Engineering](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store)).

## 4. The comparison: the fit test, and where it breaks

| Test | eBay (catalog) | Stripe (ops docs) | EA/Sailthru (state) | Espresso (hybrid) |
|---|---|---|---|---|
| The aggregate | one listing | one payment/config object | one user's game/profile state | one profile's document group |
| Read shape | whole-aggregate render | by key, per service | load, mutate, save | relational queries on the group |
| Cross-aggregate joins | rare (search is indexed) | rare | rare | kept alive: secondary indexes |
| Schema volatility | extreme, per category | moderate, evolution over years | high, per game release | low, feature shock absorbed by build |
| Transactions across units | not required | within one aggregate | not required | required: relational inheritance |
| Evidence level | VP | FP | VP | FP |

**Cross-case comparison paragraph.** The four cases agree on one property and
divide on everything else: the *aggregate* is the natural unit, accessed whole
and rarely joined across. Within that agreement, the reasons differ (catalog
flexibility, operational durability, fast-changing state, unified semantics),
which is the point: "document database" is not one decision but a family of
decisions that share an access shape. The two vendor-published cases (C06,
C08) and the two first-party cases (C07, C09) also bracket the evidence
spectrum in this family: document-family stories are the most
vendor-mediated in the corpus, a fact worth carrying into any purchasing
decision informed by this book (Appendix B).

Where the fit test breaks is the other side of the chapter. The moment the
workload needs (i) queries across aggregates, (ii) transactions spanning
several documents, or (iii) ad-hoc analysis with joins, the document shape
stops helping and starts taxing: the app re-implements joins, or the data
gets exported somewhere relational. The recorded exits from the family follow
exactly those three cracks (T10, T19), previewed next.

## 5. The exit-flow preview: T10 and T19

The transition ledger records two exit classes from the family:

- **T10 (document to key-value, Mixmax).** A small-team exit from self-run
  MongoDB replica sets to managed DynamoDB: "the great migration" paid down
  operational burden, not a data-model mistake ([T10: Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb), FP). The aggregate shape
survived the move (assessment: the post documents the move, not the data
  model, so the shape-survival reading is the reviewer's inference); the ops
  burden did not survive (self-running a database cluster is itself an ops
  burden that the team bought its way out of: the ops-burden driver, Ch. 21).
- **T19 (document to relational, the "quiet return"): slice (fintech), Infisical, and (R6) Voucherify.** slice moved for ACID over money flows and one unified SQL+JSONB platform ([T19: slice](https://engineering.slice.bank.in/seamlessly-migrating-from-mongodb-to-postgresql-leveraging-data-platform-for-scalable-success-404e2577a47b?gi=dd706d6a831e), FP); Infisical similarly converged on one platform ([T19: Infisical](https://infisical.com/blog/postgresql-migration-technical), FP); Voucherify moved without downtime via side-by-side sync (triggers + AWS DMS in document mode) and cut ~30% of cost, per the author's own cross-post ([T19: Voucherify](https://www.voucherify.io/blog/how-we-moved-from-mongodb-to-postgres-without-downtime), [author cross-post](https://dev.to/msedzielewski/how-we-moved-from-mongodb-to-postgres-without-downtime-and-cut-our-costs-by-30-7fd), FP; the ~30% figure is the author's claim, vendor-adjacent: treat as self-reported).

Two honest boundary conditions on T19, both registered: the returns are
first-party only from small and mid-size teams; the big-tech reversal from a
dedicated document store back to relational engines remains undocumented
(deferral D3), and the direction remains open: a team that fits
one node rarely needs the document model's distribution story at all
(assessment). The full matrix reading is Chapter 16's business; Chapter 2
plants the seed: the family's exits are access-shape exits, not fashion exits.

## 6. The pattern

- **P2 - Workload-match the migration.** Split by access pattern, not by
  flagship product. Faces here: the three reasons *are* three access patterns
  arriving at one shape. Fails when semantic translation is assumed free: the
  T19 movers all had to rebuild integrity machinery (Voucherify's triggers,
  slice's money-flow invariants), the same residual cost Register notes for
  Amazon's savepoint timeouts (C46) and Deliveroo's Streams rebuild (C63).
- **P15 - Build only what converges.** The builds that paid off (Vitess,
  MyRocks, M3/Atlas in Chapter 1's neighborhood) became platforms or open
  source; the internal-only ones (Espresso, Manhattan, Schemaless) delivered
  less than the equivalent 2026 product. Faces here: C09, with C49 (Netflix
  buying persistence-as-a-service in 2023) as the modern control. Fails when
  there is no open-source or platform plan: the team bears full maintenance
  forever.

## Exercises

**●○○ (identify the aggregate).** A support-ticket product stores: tickets,
each with messages, tags, assignee history; a customer directory; an SLA
rules table. Tickets are rendered whole in the UI; SLA rules change weekly;
reports join tickets to customers. Which unit is the true aggregate for a
document store, and which part of the workload already points outside the
family?

*Hint:* apply the fit-test table row by row.

*Worked answer (assessment):* the ticket (messages + tags + history is the
unit rendered whole) is the aggregate; the SLA table is small, rule-like, and
joins in reports, and the reporting joins to the customer directory are
exactly the cross-aggregate access mark. So: document store for tickets,
relational for customers+rules+reports (or a doc store plus a separate
reporting path). One store for all three would be the P2 fails-when.

**●●○ (apply the framework).** You are the platform lead at a 60-engineer
company with a self-run MongoDB cluster (3 replica sets, 2 TB): a DevOps
rotation that exists mostly to babysit it, workload that is 85% aggregate
reads by ID, quarterly schema migrations executed with a custom online-migration
tool, and a finance requirement landing next year: per-customer invoice tables
with cross-customer Excel-style exports. Your options: (a) stay, delegate the
ops to a managed document service; (b) move the whole system to relational;
(c) split: document for the aggregate workload, relational for invoices and
exports. Decide, per the chapter's fit test and patterns, and name the
patterns.

*Worked answer (assessment, judgment):* (c), deliberately. The 85%-by-ID
workload is the family's home turf (the aggregate test passes), so a rewrite
to relational buys nothing for the dominant access shape; the invoice/export
requirement is the cross-aggregate, join-shaped workload where the family
taxes you, and moving *that* to relational avoids re-implementing exports in
the application. (a) is viable and cheaper than (b) or (c), and a structure
where the managed document service is paired with a warehouse for exports is
the honest version of the split. (b) is the T19 shape and would carry the
highest semantic-translation cost (P2 fails-when) for little fit gain. The
pattern credits: P2 (split by access pattern), P15 n/a (nothing built), P19
(if the split goes forward, the playbook chapter).

**●●● (steelman the build, then weigh it).** LinkedIn's Espresso (C09) was
built in the early 2010s, when no product sold "Oracle semantics at
Voldemort scale." Write a 200-400 word assessment that (i) states the
strongest case that the build was correct *for its time*, (ii) applies the
P15 test to the same build viewed from 2026, and (iii) takes a position on
whether a 2015-startup with LinkedIn's access patterns should have built,
bought, or done what DoorDash did in 2022 (C25). Supported parts must cite
[C09](https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store) and any register entry you use; position items are Verdict (assessment).

*Suggested answer sketch (assessment):* (i) in 2010-2013, the honest market
check comes up empty: no distributed database then sold transactions plus
secondary indexes plus horizontal scale; the corpus's own timeline (F1 2012,
OceanBase 2013-19) says the products arrived years later; (ii) P15: the build
never converged into a platform, so LinkedIn pays maintenance forever, the
test's signature failure; (iii) a 2015 startup should buy (the market existed
by then; its requirements were write scale plus relational queries, which is
the NewSQL family's spec sheet: Chapter 8); DoorDash's 2022 move is the
correct template for the "buy when converged" rule (C25).

**◆ (diagnostic: no hints).** Three teams describe their moment. For each,
decide whether the document family fits, and name the one pattern whose
fails-when they are closest to: (i) a CMS whose "pages" nest widgets
arbitrarily deep and render whole; (ii) a payments team storing
transactions-as-documents, now asked "how many distinct customers spent over
Rs. X across all their transactions last month?"; (iii) a startup whose
founder wants MongoDB "because we don't have a schema yet". One sentence
each; no credit for engine names.

*(Diagnostic answers, assessment: (i) fits: whole-aggregate render, hyper-
volatile internal schema, no cross-aggregate joins; closest fails-when is
P2's, if reporting joins are forced into the same store later. (ii) does not
fit the new question: that is a cross-aggregate aggregation, the T19 shape;
the payments store itself may stay document (C07's shape) if the query is
served by a relational sidecar or warehouse. (iii) fits nothing yet: the
reason given is the structural-with-no-aggregate anti-pattern; the correct
P-pattern is P2 applied in reverse: choose the shape from the access pattern
that exists, not from the fear of one that doesn't.)*

## Sources

All from the register; URLs copied exactly.

- C06 eBay (MongoDB blog, VP): https://www.mongodb.com/blog/post/ebay-building-mission-critical-multi-data-center-applications-with-mongodb
- C07 Stripe document databases (FP): https://stripe.dev/blog/how-stripes-document-databases-supported-99.999-uptime-with-zero-downtime-data-migrations
- C08 EA FIFA Online / Sailthru (MongoDB at scale, VP): https://www.mongodb.com/products/capabilities/mongodb-scale
- C09 LinkedIn Espresso (FP): https://engineering.linkedin.com/espresso/introducing-espresso-linkedins-hot-new-distributed-document-store
- T10 Mixmax Mongo→DynamoDB (FP): https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb
- T19 slice (FP): https://engineering.slice.bank.in/seamlessly-migrating-from-mongodb-to-postgresql-leveraging-data-platform-for-scalable-success-404e2577a47b?gi=dd706d6a831e
- T19 Infisical (FP): https://infisical.com/blog/postgresql-migration-technical
- T19 Voucherify (FP): https://www.voucherify.io/blog/how-we-moved-from-mongodb-to-postgres-without-downtime
- T19 Voucherify author cross-post (~30% figure): https://dev.to/msedzielewski/how-we-moved-from-mongodb-to-postgres-without-downtime-and-cut-our-costs-by-30-7fd