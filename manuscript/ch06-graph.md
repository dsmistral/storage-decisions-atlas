# Chapter 6 - Graph: Three Jobs

**Governing idea.** The graph family earns its keep on one kind of question:
the multi-hop query, "everyone connected to X within two steps, with
weights" (the survey's characterization), which relational engines answer
with recursive joins that collapse at depth. This chapter's three cases all bought the same engine (Neo4j) and
all asked that question, but they form a maturity ladder of jobs: the graph as
an exploration surface for humans, the graph as an acceleration layer for
domain experts, and the graph as inference context for machine learning.
Reading them side by side answers a question the vendor material never asks:
what stays constant as a graph installation grows up, and what gets harder?

**Cases in this chapter:** C20 (ICIJ Panama Papers: Neo4j + Linkurious,
FP/VP: the speaker is first-party, the channel is the vendor's), C21 (Zurich
Insurance claims fraud, vendor-published), C22 (eBay ShopBot knowledge graph,
vendor-published).
Register IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`.

---

## 1. Context: buying by job, not by name

A graph database stores nodes and edges as first-class citizens, so the
query engine walks connections directly instead of re-deriving them with
joins. For one-hop questions (orders by customer) that is a wash; relational
engines have served those for forty years. The family's distinctive bet pays
off exactly when the question's shape is "follow the chain": agents
connected to shell companies connected to accounts, or a fraudster's claims
history connected to accomplices, or a product connected to attributes
connected to how people talk about it.

The three cases ahead share that shape and nothing else, which is the point.
They differ in who the user is (hundreds of journalists, fraud
investigators, a research team behind a shopping assistant), in how central
the graph is to the product (the whole product, an internal tool, an
inference component), and in how much of the workload is human exploration
versus machine inference. The chapter's grading follows the evidence: a rich
first-party account exists for the first case (though the corpus carries it
on the vendor's channel, so its grade is FP/VP); the other two are
vendor-published and the chapter holds them to that standard.

## 2. The cases, each against its time

### The exploration surface: ICIJ's Panama Papers (C20)

The International Consortium of Investigative Journalists is "a network of
around 200 journalists in more than 65 countries" ([C20: Neo4j interview with
Mar Cabra, ICIJ](https://neo4j.com/blog/cypher-and-gql/icij-neo4j-unravel-panama-papers/), FP).
When the Panama Papers leak arrived, its raw material was 11.5 million files:
PDFs, emails, scans, unstructured documents with no schema and one implicit
product, the relationships inside them. ICIJ's data and research unit built
an indexing layer (Apache Tika for extraction, Solr-backed search front
ends) and a Neo4j graph as the shared explorer, with Linkurious chosen as the
visualization layer: "software that allows you to visualize graphs very
easily" (same source). ICIJ's Data & Research Unit editor Mar Cabra's
verdict quoted by the
press: "Neo4j is a revolutionary discovery tool that has transformed our
investigative journalism process. This simply would not have been possible
before on this scale. It's magic." ([C20: Computer Weekly](https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software), press).

The case's engineering interest is not scale (11.5 million files is modest);
it is that the graph is the product. Data entered as a one-time investigative
import was structured so that a journalist, not a query, is the runtime:
double-clicking through neighbors of an entity is the access pattern, and the
engine, the visualization, and the workflow were chosen around that single
job.

### The acceleration layer: Zurich Insurance (C21)

Zurich Switzerland "began automating fraud detection early on", running
claims through rule-based software; its claims center answers "a million
calls... each year and process[es] 500,000 claims" ([C21: Neo4j case study,
PDF](https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf), VP).
The problem the vendor documents is context: a rule engine flags what smells
wrong, but a human investigator deciding in minutes needs the surrounding
web (prior claims, shared addresses, linked parties) that tabular views
"could not assemble" (register C21, VP). Neo4j "provides investigators with
the context they need to make quick decisions" (same source). Note the shape
of this case compared with ICIJ: the graph sits beside an existing
rule-based production stack and serves people, not training pipelines. The
evidence level is VP, and the chapter's grading does not pretend otherwise:
the case study is the vendor's own document, and its numbers (claims per
year, employees at the center) are unaudited self-reports. What survives the
discount is the access-pattern claim: the multi-hop question here is asked by
domain experts, dozens of times a day, at the moment the decision happens.

### The inference component: eBay ShopBot (C22)

eBay's ShopBot was the shopping assistant driven by a graph database, and
the interview's own framing is the cleanest statement of the job: "Ajinkya
and his team use Natural Language Understanding to bake machine learning
into the graph database that drives eBay's virtual shopping assistant, eBay
ShopBot"; for the team's lead it is "all about the synergy between graphs
and machine learning" ([C22: Neo4j interview with Ajinkya Kale,
eBay](https://neo4j.com/blog/knowledge-graph/machine-learning-on-graphs-ajinkya-kale-senior-applied-researcher-ebay/), VP).
In the maturity ladder this is the hard end: the graph is consumed by
programs, its schema encodes product entities and their relations, and its
correctness is judged indirectly (through the assistant's answers) rather
than by a human reading a visualization. The case is the newest of the three
and the least documented; the register holds one vendor interview, and this
chapter reads it as a direction record, not an operations study.

## 3. What happened (sourced outcomes, with evidence levels)

- **ICIJ (C20, FP/VP).** 11.5 million leak files mined through Neo4j +
  Linkurious by a 200-journalist network in over 65 countries; the toolchain
  (Tika, Solr, Neo4j, Linkurious) was documented in the ICIJ's own interview
  ([C20:
  Neo4j interview](https://neo4j.com/blog/cypher-and-gql/icij-neo4j-unravel-panama-papers/); [C20: Computer Weekly](https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software)).
- **Zurich (C21, VP).** 6,100 employees answering ~1 million calls and
  processing 500,000 claims a year; Neo4j added as the context layer for
  investigators working with rule-based fraud detection; investigators get
  connected context "to make quick decisions" ([C21: Neo4j case study,
  PDF](https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf)).
- **eBay (C22, VP).** A knowledge graph powering the ShopBot assistant, with
  NLU and machine learning run on top of it; one vendor interview on record
  ([C22: Neo4j interview](https://neo4j.com/blog/knowledge-graph/machine-learning-on-graphs-ajinkya-kale-senior-applied-researcher-ebay/)).

## 4. The comparison: the maturity ladder

| Test | ICIJ (exploration) | Zurich (acceleration) | eBay ShopBot (inference) |
|---|---|---|---|
| The graph is... | the product | a tool beside the rules engine | a component of an ML product |
| Who consumes it | hundreds of journalists, visually | fraud investigators, in the decision moment | a program (NLU + ML) |
| The multi-hop question | follow the money | assembled context per claim | resolve entities for the assistant |
| What's missing | nothing material (FP) | operations detail; unaudited figures | almost everything (one interview) |
| Evidence level | FP + press | VP | VP |

**Cross-case comparison paragraph.** What stays constant up the ladder is the
access pattern (walk connections, human-legible) and the buy-not-build
choice: all three adopted, none built. What changes is blast radius and
verifiability: the exploratory layer fails visibly (a missing link is a
journalist's afternoon), the acceleration layer fails operationally (an
investigator's minute), and the inferential layer fails silently (a wrong
entity resolution inside an assistant's answer). That gradient, not a
feature list, is the real maturity measure of a graph installation, and it
is what a team should price before adopting: how deep up the ladder their
job sits, and therefore how much verification burden the choice quietly
carries.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **ICIJ (C20):** exemplary, and the corpus is lucky to have a first-party
  voice behind it. The lesson generalizes past journalism: when the data's
  value is relationships between documents, a graph plus a purpose-built
  visualization is the correct machine, and Cabra's own verdict ("magic")
  was earned by the workflow it served, not by marketing.
- **Zurich (C21):** believable and well-shaped, but vendor-carried. The
  direction (graph context beside a rules engine) is the standard production
  shape for fraud work and the case's logic holds; the absence of any
  non-vendor figure is why its grade stays at VP.
- **eBay (C22):** the right case at the wrong detail level. "ML on graphs"
  is the family's growth story, and the corpus currently holds it up with
  one interview; this book accepts that and says so. The lesson is about
  evidence budgets: at the inferential end of the ladder, the public record
  is structurally thin, because the interesting failures are internal
  metrics, not launch posts.
- **The family chapter's own finding:** no P-index migration pattern fires
  here, and that absence is a finding, not a gap to paper over. The three
  cases are adoptions split by job, close to the family rulebook at Ch. 5's
  P2 (match the workload) but without any migration to run: the buying was
  the decision. The chapter map's pattern cell for this chapter records that
  honestly.

## 6. The pattern

- **The family rule (unstated in the P-index, which holds migration
  patterns):** a graph engine is the right buy when the workload's queries
  are multi-hop and human- or inference-legible; the three cases are
  instances by job (exploration, acceleration, inference), not by scale.
- **P15 contrast (build only what converges):** all three cases bought an
  engine rather than building one; the graph family in this corpus holds no
  build case at all, which is itself a strong signal that the converged
  products satisfy the family's real requirements better than in-house reuses
  of relational machinery.
- **BOOK-PLAN note (assessment):** the plan's pattern cell for this chapter
  lists P2 ("workload-match the migration"); the chapter's reading, recorded
  here for the next planning pass, is that no migration occurs in any of the
  three registered cases, so P2's migration machinery has nothing to grip.
  The chapter names the access-pattern rule and the P15 contrast instead.

## Exercises

**●○○ (identify the pattern).** A compliance team at a bank needs to answer
"which entities are connected to this politically exposed person within two
steps?" several times a day, on data reloaded weekly from a warehouse. Name
the family rule this matches and the two purchases it implies.

*Hint:* the users read a screen; they never write a query.

*Worked answer (assessment):* the graph family rule (multi-hop,
human-legible); the two purchases are the engine and the visualization
layer, the ICIJ shape (Neo4j + Linkurious is the corpus's face). No
migration pattern fires; this is an adoption split by job.

**●●○ (apply the framework).** The same bank now wants fraud alerts raised
automatically by scoring paths in the graph in real time. Apply the chapter's
ladder: what changes about the verification burden as the graph's consumer
moves from investigator to program?

*Worked answer (assessment, judgment):* everything in the comparison
gradient: at the exploration level a graph mistake is visible to the human
reading the screen; at the inference level a wrong or missing edge becomes a
silent mis-score, so the verification burden moves from the analyst's eyes
into data-quality pipelines, entity-resolution checks, and monitoring on the
score distribution itself. The chapter's guidance: before climbing the
ladder, budget for that infrastructure, because it cannot be bought as a
feature flag.

**●●● (steelman, then weigh).** Write the strongest case that the bank
should not adopt a graph database at all and should instead answer the
two-step questions with recursive CTEs on its existing warehouse. Then weigh
it against the three registered cases. End with the transferable rule.

*Steelman and verdict (assessment):* (i) the steelman: at two hops with
weekly-reloaded data, a modern columnar warehouse answers recursive CTEs
interactively for an analyst; the operational cost of a second engine
(serving, visualization stack, skills) is real, and the chapter's own
corpus holds vanishing evidence of graph workloads that relational engines
cannot match at depth two; (ii) the weigh: the corpus's three cases do not
include such an alternative because none of the adopters needed one, ICIJ's
leak shape and Zurich's decision-moment context both exceed it. The
transferable rule: prototype the exact multi-hop question at your exact
operational depth (hop count, freshness, audience) in the incumbent engine
first; buy the graph when the incumbent's answer degrades visibly, not
because the data has entities in it.

**◆ (diagnostic: no hints).** One sentence each: which graph job (and
therefore which verification burden)? (i) A social network serves "friends
of friends who like X" in a product screen. (ii) A lab tracks citation
networks for a report written once a quarter. (iii) A support product
resolves "the customer mentioned the machine the agent shipped last week".

*(Diagnostic answers, assessment: (i) inference, product-load verification:
the graph serves latency-bound traffic and needs engine-grade ops from day
one; (ii) exploration, lowest burden: casual refresh, human-readable output,
almost any engine survives; (iii) acceleration shading into inference: the
graph feeds context to humans and to automated resolution, so both lens
classes of verification apply.)*

## Sources

All from the register; URLs copied exactly.

- C20 Panama Papers revealed by graph database visualisation software (Computer Weekly, press): https://www.computerweekly.com/news/450280758/Panama-Papers-revealed-by-graph-database-visualisation-software
- C20 ICIJ: how we used Neo4j to unravel the Panama Papers (Neo4j interview with Mar Cabra, FP): https://neo4j.com/blog/cypher-and-gql/icij-neo4j-unravel-panama-papers/
- C21 Faster Fraud Investigation with Neo4j, Zurich Switzerland (Neo4j case study PDF, VP): https://go.neo4j.com/rs/710-RRC-335/images/Neo4j-Case-Study-Zurich-EN-US.pdf
- C22 Machine learning on graphs: 5-minute interview with Ajinkya Kale, eBay (Neo4j, VP): https://neo4j.com/blog/knowledge-graph/machine-learning-on-graphs-ajinkya-kale-senior-applied-researcher-ebay/
