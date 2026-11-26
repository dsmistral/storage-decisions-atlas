# Chapter 10 - Vector: Where the Index Lives

**Governing idea.** The vector family is the only one in this part whose
central question is not which engine but where the index lives. An
embedding makes every object a point in high-dimensional space, and the
family's job is nearest-neighbor search over that space at low latency.
Three placements exist. The index can live in a dedicated vector engine
(turbopuffer's object-storage-first service, Pinecone), in a library your
own service embeds (Spotify's Annoy and Voyager), or inside the relational
database you already run (pgvector in Postgres). The book plan's spec holds
the chapter's rule: if vectors are a feature, a library keeps operations
near zero; if retrieval is the product, an engine earns its keep; and the
unwind, when a team moves between placements, is documented in both
directions, which makes this the family where direction is least able to
settle into dogma.

**Cases in this chapter:** C32 (turbopuffer, first-party blog plus
third-party profiles), C33 (Spotify Voyager, first-party post and
repository), C34 (Confident AI's Pinecone-to-pgvector account,
first-party), C35 (Pinecone's counter-account, vendor post plus press).
Register IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`.

---

## 1. Context: a young family with a placement question

The vector family is the youngest in this book, and its case record is
short in years and long in stakes. Embeddings became infrastructure when
retrieval-augmented generation made "find similar things fast" a serving
requirement rather than an offline research step, and the 2022 to 2024
window produced a wave of dedicated vector databases priced for that
moment. The corpus's turbopuffer anchor names the pricing regime the wave
built: "In 2022, production-grade vector databases were relying on
in-memory storage at $2+ per GB, not counting the extra cost for durable
disk storage" ([C32: turbopuffer blog](https://turbopuffer.com/blog/turbopuffer),
FP; updated March 2026).

The fit question the family imposes is placement, not persistence. A team
whose product needs "find related documents" as one feature among many has
different obligations from a team whose product is the retrieval itself,
with filtering, hybrid keyword-and-vector search, and durability guarantees
on its index. Both 2020s poles in this chapter can be read as answers to
weights placed wrongly by the others: the dedicated engine answered a
market priced on in-memory storage, and the embedded middle (a vector
index inside an existing generalist store) trades scale and quality
ceilings for operational simplicity. The register's transition row for the
family's unwind makes the honest summary: workload-dependent, not
directional (T20).

## 2. The cases, each against its time

### turbopuffer: the cost-architecture engine (C32, T14)

The chapter's engine case begins as a shelved feature. In late 2022,
turbopuffer's co-founder and CEO Simon Hørup Eskildsen was helping Readwise
scale for the Readwise Reader launch, and the blog states the arithmetic
that killed the feature: "Readwise was paying ~$5k/month for their
relational database, but we found that vector search on the same 100m+
documents would've cost $20k/month+!" It then states the product
consequence with unusual bluntness: "This wasn't just expensive; it meant
we had to shelve a desired feature until costs came down. It gnawed at me
for months. A promising feature, postponed solely due to infrastructure
costs?" ([C32: turbopuffer blog](https://turbopuffer.com/blog/turbopuffer),
FP). The register records the same event as transition T14's Readwise face:
shelved feature, then shipped when the cost structure changed.

The architectural bet against the in-memory regime is the post's
engineering content: "object storage is now ubiquitous, NVMe SSDs have
become incredibly fast and affordable, and AI & vectors means we're asking
more from our search systems than before" (same source). The family's
engine pole is therefore a cost-and-durability design: separate compute
from object storage and cache hot data on NVMe when queried, which is what
the third-party profile records: "Its architecture separates compute from
storage, keeping data in object storage and bringing it into NVMe or memory
caches when queried" ([C32: AI Engineer profile](https://ai.engineer/orgs/turbopuffer),
PR). The scale claims sit on the vendor side and the profile's reporting:
"In April 2026, the company reported four trillion documents in production
and over 25,000 searches per second", and "Customers include Anthropic,
Atlassian, Cursor and Notion" (same source; the run-rate estimate on that
page is a third-party analyst's figure, kept at PR level).

The register's T14 Cursor face lives in the press interview, not the blog:
"launching turbopuffer as a scrappy side project, getting an email from
Cursor the next day, flying out after a 4am call, and helping cut Cursor's
costs by 95% while fixing their per-user economics" ([C32: Latent Space](https://www.latent.space/p/turbopuffer),
PR; the figure is turbopuffer-reported and carries P16's caution, below).

### Spotify's Voyager: the library keeps ops near zero (C33, T18)

Spotify's vector history starts early on the library side: "Spotify led
the pack by building and open sourcing Annoy, our hugely popular
nearest-neighbor search library, back in 2013", which "has served us
extremely well, powering features like Discover Weekly, Home, and
countless others" ([C33: Spotify Engineering](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library),
FP). A decade later the library was replaced by another library, which is
the family's cleanest transition record: the trigger, per the third-party
account, is that Annoy's algorithm "10 years on it no longer performs well
at their current scale" ([C33: InfoQ](https://www.infoq.com/news/2023/11/spotify-ann-voyager/),
PR), and the replacement is Voyager, an HNSW library built on the
open-source hnswlib package, claimed by the post at "More than 10 times the
speed of Annoy" (at the same recall) or "up to 50% more accuracy" (at the
same speed), with "4 times less memory usage than Annoy" via an 8-bit
floating-point format (same source). Transition T18 records the move with
the register's evidence level at FP.

The placement logic is the post's explicit design goal: "Many of Spotify's
systems use nearest-neighbor search in memory, enabling stateless
deployments (via Kubernetes) and almost entirely removing the maintenance
and cost burden of maintaining a stateful database cluster" ([C33: Spotify
Engineering](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library),
same source). The production proof sits in the repository README, and it is
the family's cleanest published scale statement: "Voyager is used
extensively in production at Spotify, and is queried hundreds of millions
of times per day to power numerous user-facing features", with the
positioning line "like Annoy, but with much higher recall" ([C33: Voyager on
GitHub](https://github.com/spotify/voyager), FP).

### Confident AI: Pinecone replaced with pgvector (C34, T20 outbound)

The embedded pole's outbound case is a first-party evaluation post from
Confident AI, a small evaluation-tooling vendor, titled around why it
replaced Pinecone with pgvector. Its data-architecture criticism is
structural: "the difficulty in unifying fragmented data sources, which is
necessary for conducting vector searches on your data" ([C34: Confident
AI](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector),
FP). Its measured element is not the company's own benchmark but one it
cites: "pgvector's IVFFlat index outperforms s1 pods on the same compute
and manages 143% more QPS", with the post marking the figures as "Data
taken from Supabase" (same source). The register's summary ("measured
better accuracy + QPS on same compute") compresses this; the chapter
keeps the compression visible and the citation chain exact: a first-party
post citing a third-party benchmark, not a first-party measurement.

The post is also honest about the reverse's edge cases, under the heading
"PGVector is Not Perfect Either": fragmented data sources cut both ways,
and it grants Pinecone a concrete per-pod comparison, "For about $160
monthly, you can use a p1.x2 pod without needing replicas and still achieve
close to 60 QPS with a high accuracy of 0.99" (same source). A register
note: the register's T20 row was built on this post and its sibling
counter-account below, and the register is careful to call the move
genuinely workload-dependent (register T20).

### Pinecone: the counter-flow, documented by the incumbent (C35, T20 inbound)

The corpus's honesty about direction depends on cases like this: the
incumbent engine vendor documents users moving into Pinecone from the
embedded pole. Pinecone's April 2024 engineering post, by three of its
engineers, states the flow plainly: "Over the past several months, people
have come to Pinecone after trying and failing with pgvector and other
bolt-on vector databases" ([C35: Pinecone blog](https://www.pinecone.io/blog/pinecone-vs-pgvector/),
VP), and frames its own investigation the same way: "When we investigated
why people choose pgvector initially and then switched to Pinecone" (same
source). The register's summary records the same flow, vendor-documented
users arriving after hitting pgvector limits, consistent with this wording.

The same post carries the family's most cautionary economics: a modeled
comparison whose stated assumptions are a light workload, "on average, 10
queries per minute" and monthly modification of 10% of the data, under
which "Pinecone Serverless is cheaper for the four tested datasets" (same
source; the post reports the margin as a 1.1x to 2.2x overall factor and a
1.5x to 2.9x ongoing-cost factor, both vendor-estimated). The other
direction is carried by the press piece at the register's PR level: "Teams
that have migrated workloads from Pinecone to pgvector at this scale have
reported 40–60% cost reductions, particularly when using managed PostgreSQL
services like Neon or Supabase that eliminate self-hosting overhead" ([C35:
Java Code Geeks](https://www.javacodegeeks.com/2026/05/vector-databases-when-pgvector-beats-pinecone-and-when-it-doesnt.html),
PR; the claim is scoped to the piece's "medium scale (10–100M vectors)"
band, and its provenance is the piece's compilation of migration reports,
not a first-party post).

## 3. What happened (sourced outcomes, with evidence levels)

- **turbopuffer (C32, FP blog + PR profiles).** Readwise's vector-search
  feature priced at $20k+/month on the incumbent architecture against a
  ~$5k/month relational bill was shelved, then became shippable on an
  object-storage-first architecture; the company reports four trillion
  documents in production and over 25,000 searches per second as of April
  2026, with Anthropic, Atlassian, Cursor and Notion named as customers
  ([C32: turbopuffer blog](https://turbopuffer.com/blog/turbopuffer);
  [C32: AI Engineer profile](https://ai.engineer/orgs/turbopuffer)).
  Cursor's ~95% search-cost cut is turbopuffer-reported via the Latent
  Space interview ([C32: Latent Space](https://www.latent.space/p/turbopuffer)),
  vendor-figure caution applies.
- **Spotify (C33, FP + PR).** Annoy (2013, open-sourced) was replaced by
  Voyager (2023, open-sourced, HNSW on hnswlib): claimed "More than 10
  times the speed of Annoy" at the same recall and "4 times less memory
  usage than Annoy"; deployed stateless on Kubernetes, "queried hundreds of
  millions of times per day" in production
  ([C33: Spotify Engineering](https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library);
  [C33: GitHub](https://github.com/spotify/voyager);
  [C33: InfoQ](https://www.infoq.com/news/2023/11/spotify-ann-voyager/)).
- **Confident AI (C34, FP with a cited third-party benchmark).** Moved from
  Pinecone to pgvector, citing source fragmentation; the measured element
  is Supabase benchmark data the post cites ("143% more QPS" on the same
  compute), not the company's own measurement
  ([C34: Confident AI](https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector)).
- **Pinecone (C35, VP + PR).** The incumbent's own post documents users
  arriving "after trying and failing with pgvector"; its cost model, under
  stated light-workload assumptions, reports Pinecone Serverless cheaper
  for four test datasets ([C35: Pinecone blog](https://www.pinecone.io/blog/pinecone-vs-pgvector/)).
  The press piece reports 40–60% cost cuts for Pinecone-to-pgvector moves
  in the 10–100M-vector band
  ([C35: Java Code Geeks](https://www.javacodegeeks.com/2026/05/vector-databases-when-pgvector-beats-pinecone-and-when-it-doesnt.html)).

## 4. The comparison: where the index lives

| Test | turbopuffer (C32) | Spotify Voyager (C33) | Confident AI (C34) | Pinecone (C35) |
|---|---|---|---|---|
| Index placement | dedicated engine, object storage + NVMe cache | in-process library, stateless on Kubernetes | Postgres-embedded (pgvector) | dedicated engine (managed) |
| Ops burden | owned by the vendor | near zero for the app team | absorbed by the existing Postgres ops | owned by the vendor |
| Fit claim | retrieval as product: filters, hybrid, durability | vectors as a feature of a bigger serving system | vectors as a feature, Postgres already present | retrieval as product with quality-first needs |
| Claimed economics | $20k+/month avoided at Readwise; ~95% cut at Cursor (vendor-reported) | 10x speed; 4x memory (first-party claims) | 143% more QPS on same compute (cited benchmark) | cheaper for 4 light datasets (vendor model); counter: 40–60% cuts at 10–100M vectors (press) |
| Evidence level | FP + PR | FP + PR | FP (cited benchmark) | VP + PR |

**Cross-case comparison paragraph.** The four cases line up on one axis,
where the index lives, and the family's economics follow from that axis
before any engine-quality question. turbopuffer put the index in a
dedicated engine built so retrieval is the product, with filters, hybrid
search and durability, and priced it by attacking the family's founding
in-memory cost floor (the $2+/GB regime). Spotify put the index in a
library, twice (Annoy, then Voyager), because vectors are a feature of a
serving system whose statelessness the platform team already owns, and the
maintenance burden of a stateful database cluster is exactly what the
placement refuses. pgvector puts the index inside Postgres, which is the
zero-new-operations placement: the Confident AI case shows its pull
(source fragmentation dies, the existing team absorbs it) and its ceiling
(the cited benchmark beats s1 pods, but the post's own honesty section
concedes where it is not perfect). Pinecone's counter-account shows the
incumbent pole converting embedded-pole failures into customers. That two
of the four records are vendor or vendor-adjacent, and the two
"measured" elements sit on a cited benchmark and a vendor cost model, is
not a footnote to this family's record; it is the record. The survey's
comparison verdict carries: the unwind is genuinely workload-dependent,
not directional (register T20; survey comparison, graded as assessment).

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **turbopuffer (C32):** the cost-architecture bet, and the family's
  cleanest narrative arc: a feature killed by an infrastructure bill, then
  revived by a change in where the index lives. The four-trillion-document
  scale and named customers, if they hold, make it the engine pole's
  production proof; but the two headline figures (the 95% Cursor cut and
  the revenue run rate) are vendor-reported and third-party estimated
  respectively, and P16's caution binds on both.
- **Spotify (C33):** the placement discipline case. The team's explicit
  goal was operating cost and statelessness, and it accepted a library's
  working-set ceiling (nearest-neighbor search in memory is the post's own
  framing; what the library does not bring, a stateful cluster's serving
  guarantees around the index, is exactly the ops the placement refuses) to
  get them. Replacing Annoy with Voyager after ten years, rather than
  adopting a database, is the placement question answered the same way
  twice, which is the strongest behavioral evidence in the corpus for the
  feature-pole reading.
- **Confident AI (C34):** the honest small-team record: a first-party
  account that cites its benchmark rather than claiming one, and concedes
  its choice's limits in its own headings. Its evidentiary weight for the
  family's unwind is correspondingly modest: one workload, borrowed
  numbers, and an author whose product (an evaluation framework) benefits
  from the story it tells.
- **Pinecone (C35):** the incumbent counter-case, useful precisely because
  it is vendor-generated: the migration narratives are the vendor's
  reading of inbound users, and the cost model's assumptions (10 queries
  per minute, 10% monthly churn) are stated and light. Read as "managed
  engines earn their keep at quality-first, high-QPS retrieval", it is
  consistent with the chapter's axis; read as economics, it is a modeled
  figure the reader must re-derive for their own workload.
- **The family's rule:** decide what vectors are to you. A feature buys
  the library if your serving system already owns state, and the embedded
  index if you already own Postgres and your vector count is inside the
  engine's demonstrated envelope; a product buys an engine whose
  economics you have modeled against your own query and churn pattern,
  not the vendor's.

## 6. The pattern

- **Family rule: where the index lives.** ANALYSIS records no pattern IDs
  for C32-C35 (the survey's comparison carries none), so the chapter names
  the rule directly, as Chapter 6 did for its family: the placement axis
  (library / embedded / engine) is zoomed by two load-bearing questions,
  whether vectors are a feature or the product, and who absorbs the
  operations. Fails-when: the placement is chosen on the vendor narrative
  rather than the workload's query, churn, and quality profile.
- **P16 - Cost at scale drives the largest migrations** (registered faces:
  C39, C41, C46/C28). This chapter carries P16's caution side: the family's
  founding story is a cost number (Readwise's $20k+/month against a ~$5k
  relational bill), and its headline migration figure (Cursor's 95%) is
  vendor-reported; Pinecone's counter-model is vendor-estimated under
  stated light-workload assumptions. Fails-when: cost is vendor-reported
  and unaudited, which describes most of this family's public numbers.
- **W1 watch (pgvector convergence vs dedicated vector stores), open.**
  The register's watch asks for a first-party, large-scale (100M+ vector)
  migration in either direction; the corpus still has none, only
  evaluation posts (C34) and vendor narratives (C35). This chapter keeps
  the watch open and records its present evidentiary state: both
  directions documented, both directions short of the register's bar.

## Exercises

**●○○ (identify the pattern).** A five-year-old web app already runs
Postgres. Its team adds "more like this" recommendations, embedding its 8
million product records, and writes the vector index into the same Postgres
instance. A platform reviewer objects that a dedicated vector database
would rank better at scale. Which of the chapter's placement obligations
does the team satisfy, and which register caution guards the reviewer's
counter-claim?

*Hint:* ask what vectors are to this product first; then ask what the
reviewer's figure would have to be before it enters the decision.

*Worked answer (assessment):* the team treats vectors as a feature and has
placed the index in the zero-new-operations middle (the C34 shape), which is
consistent with the chapter's rule; the reviewer's claim must clear P16's
caution before it counts, since this family's public performance figures
are vendor-reported or cited-benchmark material (the chapter's "measured"
elements sit on Supabase data and a vendor cost model). The decision
turns on the workload's actual query, churn, and quality profile, not on
either camp's marketing number.

**●●○ (apply the framework).** A nine-person team is building a semantic
search product where retrieval itself is the deliverable: 300 million
vectors, filtered hybrid queries on the critical path, four-nines
durability expectations, 8 queries per second steady state. The team has
no Postgres footprint and no platform group. Apply the chapter's
placement axis and P16's caution: where should the index live, and what
must the business case contain before any engine is bought?

*Worked answer (assessment, judgment):* retrieval is the product here, so
the library and embedded placements fail the test's own obligations
(filtering, hybrid, durability are the requirements, not features):
the index belongs in a dedicated engine. But the engine is not chosen on
narrative: P16 binds on every figure in this family's public record, so
the plan must contain a cost model run at the team's own QPS and churn
(the chapter's cautionary example is a vendor model built on 10 queries
per minute), a quality evaluation on the team's own queries, referencing
the filter-integration question the C32 article raises, and an exit
estimate: the corpus documents the unwind in both directions (T14, T18,
T20), which is the business case's hedge and no guarantee at 300M
vectors where the register's watch (W1) is open.

**●●● (steelman, then weigh).** Write the strongest case that the
dedicated vector database wave was a pricing bubble that pgvector and
libraries duly popped: the founding cost floor ($2+/GB in-memory) was an
engineering artifact, the deeply-funded engines' advantages were
temporary, and the chapter's own unwind record runs mostly out of the
engine pole. Then weigh it against the register's record, and end with
the transferable rule.

*Steelman and verdict (assessment):* (i) the steelman: the cost floor was
defeated by a single architectural insight (object storage plus NVMe
cache) that any operator could rent, the library pole had been free since
2013, and the embedded pole's draw (fragmentation, existing ops) needs
neither vendor; the T20 record's headline flows out of the engine pole
and the press-cited cost cuts point the same way. (ii) the weigh: the
register's record defeats the strong form of the steelman, because the
incumbent engine documents an inbound flow "after trying and failing with
pgvector", and the family's strongest published production proof (C32's
four trillion documents) is an engine pole built after the supposed
popping; the honest state is the register's own: workload-dependent, not
directional, with W1's first-party large-scale unwind still missing. The
transferable rule: a category's public numbers are mostly adversarial;
place the index by your workload, and treat every percentage in the wild
as P16 evidence.

**◆ (diagnostic: no hints).** One sentence each: where does the index
live? (i) An e-commerce app's "similar products" row, 6M vectors, Postgres
already serving the catalog. (ii) A music service's recommendation
serving, in-memory working set, platform team runs Kubernetes fleet-wide.
(iii) A legal-tech startup whose product is filtered semantic search over
customer contracts, multi-tenant, durability contractual. (iv) A three-
person internal tool adding embedding search over 200k wiki pages.

*(Diagnostic answers, assessment: (i) embedded index in the existing
Postgres, vectors are a feature and the placement adds no new operations
(the C34 shape, with the quality ceiling stated); (ii) the library, the
statelessness argument is C33's registered shape; (iii) a dedicated
engine, retrieval is the product and the obligations are contractual, so
wait for the P16-modeled economics rather than either camp's numbers;
(iv) embedded, and the chapter's smallest-workload trap is spending the
decision's time instead of shipping, with the register's both-directions
record as the escape hatch.)*

## Sources

All from the register; URLs copied exactly.

- C32 turbopuffer, turbopuffer: fast search on object storage (turbopuffer, FP): https://turbopuffer.com/blog/turbopuffer
- C32 AI Engineer, turbopuffer profile (AI Engineer, PR): https://ai.engineer/orgs/turbopuffer
- C32 Latent Space, turbopuffer: retrieval after RAG (Latent Space, PR): https://www.latent.space/p/turbopuffer
- C33 Spotify Engineering, Introducing Voyager: Spotify's new nearest-neighbor search library (Spotify, FP): https://engineering.atspotify.com/2023/10/introducing-voyager-spotifys-new-nearest-neighbor-search-library
- C33 Spotify, Voyager on GitHub (Spotify, FP): https://github.com/spotify/voyager
- C33 InfoQ, Spotify open-sources Voyager nearest-neighbor search library (InfoQ, PR): https://www.infoq.com/news/2023/11/spotify-ann-voyager/
- C34 Confident AI, Why we replaced Pinecone with PGVector (Confident AI, FP): https://www.confident-ai.com/blog/why-we-replaced-pinecone-with-pgvector
- C35 Pinecone, Pinecone vs. Postgres pgvector: For vector search, easy isn't so easy (Pinecone, VP): https://www.pinecone.io/blog/pinecone-vs-pgvector/
- C35 Java Code Geeks, Vector databases: When pgvector beats Pinecone (and when it doesn't) (Java Code Geeks, PR): https://www.javacodegeeks.com/2026/05/vector-databases-when-pgvector-beats-pinecone-and-when-it-doesnt.html
