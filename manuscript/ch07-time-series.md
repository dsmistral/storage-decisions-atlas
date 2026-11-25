# Chapter 7 - Time-Series: The Family Both Giants Built

**Governing idea.** A time-series database owes its users three things at
once: ingest that survives cardinality (the number of distinct series),
storage economics that survive retention (how long the raw points live), and
query paths that survive aggregations (downsampling and rollups applied
before human eyes or alert rules arrive). For most of the 2010s, no product
solved the three together at giant scale, and the two giants of operational
metrics, Uber and Netflix, independently reached the same conclusion: build
the platform, then open-source it. Below that scale, the family's economics
cut the other way: NexusTrade, a small trading analytics product, bought a
"purpose-built" time-series migration, fought it for two weeks, and went
home. This chapter reads both directions of the bargain.

**Cases in this chapter:** C23 (Uber M3, first-party), C24 (Netflix Atlas,
first-party), T17 (the metrics-stack transition both executed), F09
(NexusTrade on TimescaleDB, self-reported). Register IDs resolve in Appendix
A; every source URL is copied from `corpus/REGISTER.md`.

---

## 1. Context: three promises, one bill

Time-series workloads look simple and are not. The write side is a
firehose: every container, every request counter, every device emits points
continuously, and the distinct-series count (cardinality) grows with every
label a dashboard wants to slice by. The read side is dominated by range
scans and aggregation: show me the ninety-ninth percentile latency for this
service, per zone, over the last six hours, in five-minute buckets. Between
them sits the bill: storage for raw points, and the compute to roll them up
into something a human or an alert rule can afford to touch.

The general-purpose products of the era handled each promise separately.
What broke at giant scale, per the operators' own accounts below, is the
combination: cardinality that keeps climbing, retention that must be
enforced cheaply, and downsampling that must happen somewhere smarter than
"a cron job over the raw store". What breaks at small scale, per the
counter-case below, is the buyer's side of the same equation: the words
"purpose-built for time-series" name a marketing category, not a guarantee,
until they are measured on your queries.

## 2. The cases, each against its time

### Uber builds M3 (C23, T17)

Uber's metrics platform post is blunt about the ceiling of the generalists:
"common solutions like Prometheus and Graphite become difficult to manage
and sometimes cease to work" as the dimensionality and usage of metrics
increase ([C23: Uber, the billion data point challenge](https://www.uber.com/us/en/blog/billion-data-point-challenge/), FP).
The stakes it reports are whole-company: its engine handles around 2,500
queries per second, about 8.5 billion data points per second, and roughly 35
Gbps (same source). The register's press carry states the platform
"now houses over 6.6 billion time series" (register C23, press,
[Packt](https://www.packtpub.com/en-in/learning/tech-news/uber-open-sources-its-large-scale-metrics-platform-m3-for-prometheus)).

The design choice the register highlights is economics, not heroics:
downsampling happens at collection time, via the coordinator's retention and
rollup rules ([C23: Uber, M3: Uber's Open Source, Large-scale Metrics
Platform for Prometheus](https://www.uber.com/us/en/blog/m3/), FP). The
platform was built in-house and open-sourced, in full (the survey's
characterization of register T17: both giants built, both open-sourced,
both then rebuilt the query and alert layer that sat on top, which is where
M3 grew a DAG query engine; the giants' full comparison is §4 below, and
the migration-playbook treatment is Chapter 22's material).

### Netflix builds Atlas (C24, T17)

Netflix's predecessor stack was "a home-grown solution called Epic": "Epic
was a combination of perl CGI scripts, RRDTool logging, and MySQL. We were
tracking around 2M distinct time series and the monitoring system was
regularly failing to keep up with the volume of data." ([C24: Atlas docs,
overview](https://netflix.github.io/atlas-docs/overview/), FP). The volume
trends it lists (rolling out per-instance granularity, longer retention,
cloud-scale experimentation) presaged an order-of-magnitude jump in metric
volume, and Atlas was Netflix's answer: in-memory storage for dimensional
time series, "allowing it to gather and report very large numbers of
metrics, very quickly" (same source). The register records Atlas sustaining
over 1.2 billion time series in failover drills (register C24, FP; the 2014
introduction report is the source and its live page now served a stub, the
figure is verbatim in the indexed excerpt: "we can easily see Atlas sustain
greater than 1.2 billion time series").

The second mountain arrived on schedule, as T17 records: alerting. Years
later the team was paged because the Metrics Alerting System was "falling
behind": "critical application health alerts reached engineers 45 minutes
late", after the number of configured alerts had recently "increased
dramatically, by 5 times"; the fix was a
streaming-evaluation architecture ([C24: Netflix TechBlog, improved alerting
with Atlas streaming eval](https://netflixtechblog.com/improved-alerting-with-atlas-streaming-eval-e691c60dc61e), FP;
live page now serves a stub, quotes verified through indexed excerpts).
Store the metric, and alerting on it is a second engineering problem, at a
second scale.

### The counter-case: NexusTrade runs home (F09)

NexusTrade, a small trading analytics product, migrated its MongoDB data to
TimescaleDB on the "purpose-built" assumption, and the author's account is
unusually honest about the outcome. The title is the receipt: "I went
through hell and back migrating to TimescaleDB. I didn't last two weeks."
([F09: NexusTrade](https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003), SR).
The mechanics it documents: the advanced features ("continuous aggregates
and compression") "worked... for a while", then stability eroded and the
promised sub-second query performance failed to materialize (same source);
the author also discovered, mid-migration, that
MongoDB "really *had* time-series collections" (same source). The final
economics: Mongo's native time-series collections were cheaper for this
workload, and the migration was rolled back inside two weeks.

The failure class is FC5, "purpose-built" is marketing until measured on
your queries, and the nuance worth keeping: the engine that won was the one
the team already operated, which knew its native time-series support, not
the one with the sharper category name.

## 3. What happened (sourced outcomes, with evidence levels)

- **Uber (C23, FP).** Generalist stack (Prometheus/Graphite class) replaced
  by an in-house platform at 8.5 billion data points per second handled;
  downsampling and retention enforced at collection; built and fully
  open-sourced ([C23: Uber billion data point](https://www.uber.com/us/en/blog/billion-data-point-challenge/); [C23: Uber M3](https://www.uber.com/us/en/blog/m3/)).
- **Netflix (C24, FP).** Epic (perl + RRDTool + MySQL), failing at ~2M
  series, replaced by Atlas; >1.2 billion series in failover drills; alerting
  rebuilt later as streaming evaluation after configured alerts grew 5x and
  critical alerts ran 45 minutes late ([C24: Atlas docs](https://netflix.github.io/atlas-docs/overview/); [C24: Netflix TechBlog, alerting](https://netflixtechblog.com/improved-alerting-with-atlas-streaming-eval-e691c60dc61e)).
- **NexusTrade (F09, SR).** Two-week round trip: Mongo → TimescaleDB → back
  to Mongo's native time-series collections, on measured cost and stability
  ([F09: NexusTrade](https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003)).

## 4. The comparison: the two directions of one equation

| Test | Uber M3 (build, giant) | Netflix Atlas (build, giant) | NexusTrade (buy, small) |
|---|---|---|---|
| Cardinality ceiling hit | Prometheus/Graphite "cease to work" | Epic failing at ~2M series vs an order-of-magnitude jump coming | none: sub-giant throughout |
| What was built (or bought) | own platform, open-sourced | own platform, open-sourced | "purpose-built" TSDB |
| Where the next cost landed | the query/alert layer (DAG engine) | the alerting layer (streaming eval) | stability + cost, immediately |
| Reversibility | n/a (greenfield platform) | n/a (greenfield platform) | round trip survived: old path intact |
| Evidence level | FP | FP | SR |

**Cross-case comparison paragraph.** The family's economics are one equation
with two solutions. At giant cardinality, the per-series cost of the
generalist's answer ("keep everything anonymous and scan it") stopped being
payable, and both giants absorbed the build cost because metrics were
load-bearing for their operations; open-sourcing converted maintenance from
a private tax into a shared one, which is P15's success shape. Below the
giants, the same equation punishes the buyer who imports the giants' answer
without the giants' scale: a "purpose-built" engine's advanced machinery
(continuous aggregates, compression) is a feature list until it is measured
on your queries, and the round trip's survival, two weeks and home again, is
the counter-case's most valuable output.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Uber (C23):** exemplary, and the open-sourcing is what makes the case
  reusable; the build that stayed public became Prometheus-adjacent
  infrastructure. Verdict: correct then, effectively the converged product
  today.
- **Netflix (C24):** exemplary twice: once for the foresight (the
  order-of-magnitude trend analysis that preceded the build), once for
  publishing the second mountain honestly (a 45-minute-late critical alert
  is a board-level fact, and the corpus is richer for it).
- **NexusTrade (F09):** the small operator's canonical near-miss. Nothing
  in the account is flattering, which is precisely why it works as evidence:
  measured cost beat category name in two weeks, and the old path was kept
  intact the whole time. Verdict: the failure class every family chapter
  should end with.
- **The family's rule:** buy below giant cardinality, build only once the
  generalist's math is demonstrably dead, and keep the old path warm either
  way.

## 6. The pattern

- **P15 - Build only what converges.** This chapter carries the build-side
  success faces: C23/C24 (M3, Atlas), built at genuine giant scale and
  open-sourced; the register's contrast faces (Espresso, Manhattan,
  Schemaless) are reads for Chapter 12 and the Part II transitions. Fails-when: no
  open-source or platform plan, so the team bears full maintenance forever.
- **FC5 - "Purpose-built" is marketing until measured on your queries.**
  Face: F09 (continuous aggregates and compression fought the workload; back
  in two weeks). Fails-when: the buyer optimizes for the category name, and
  measures after the cutover instead of before.
- **T17 pattern (both giants):** replace the metrics stack wholesale, then
  rebuild the query and alert layer at its own scale. The register records
  it as one transition; the cases show why it must be planned as two.

## Exercises

**●○○ (identify the pattern).** An ops team runs ~50,000 distinct series on
a managed cloud product. A vendor demo shows a "purpose-built" TSDB with
impressive continuous-aggregate features. What does the chapter's rule say
about the move, and which failure class is the risk?

*Hint:* the team's last migration log entry is not the vendor's demo.

*Worked answer (assessment):* below giant cardinality, buy (or stay bought)
and measure first: the generalist handles the load, so the build/buy
question is not P15-territory but the F09 question, and the risk class is
FC5 ("purpose-built" is marketing until measured on your queries). The
chapter's addition: run the team's own queries on the candidate before any
cutover decision.

**●●○ (apply the framework).** A platform team at ~2 billion active series
(equivalent to Atlas-era Netflix) proposes adopting the open-source M3 to
avoid "the build cost". Apply T17's two-mountain lesson: what must the
plan contain beyond the storage engine?

*Worked answer (assessment, judgment):* T17's recorded finding is that both
giants rebuilt the layer above the store (M3 a DAG query engine, Netflix
streaming-eval alerting) after the store itself was settled, so the plan
must budget the second migration: query mechanisms and the alert pipeline,
with a scaling argument for each, and it must reject the "adopt the engine,
inherit the platform" shortcut because both giants built that layer as the
second, separate mountain.

**●●● (steelman, then weigh).** Write the strongest case that Uber and
Netflix should have waited for the market (managed TSDBs) instead of
building in 2013-2016. Then weigh it against the register's record and end
with the transferable rule.

*Steelman and verdict (assessment):* (i) the steelman: the products the
giants needed exist in converged form today, and the capital both companies
spent on building was capital that produced zero market option value beyond
the second mountain, query and alert tooling, which they had to build
again; (ii) the weigh: the register's record defeats the steelman on
timeline alone (Atlas sustained 1.2B series in 2014 drills while the era's
products failed at 2M; Uber's 8.5B points/second was load-bearing for a
live operations business), and open-sourcing turned the spend into platform
value. The transferable rule: build when the requirement is load-bearing
and the market is a generation behind, and stop building exactly where the
requirement stops being unique; the giants' next platform (or your own)
will handle the commodity layers.

**◆ (diagnostic: no hints).** One sentence each: which side of the family's
equation? (i) 900 series, 90-day retention, ops partner wants "a TSDB".
(ii) 400M series growing 20% a quarter, alert rules are latency-critical.
(iii) 12 series, 10-year retention, one engineer.

*(Diagnostic answers, assessment: (i) buy, and apply FC5 to the demo;
900 series is not a TSDB problem at all, the generalist holds; (ii) giant
trajectory, T17's two mountains: plan the store and the alert layer as
separate projects with the old path kept warm; (iii) neither: retention
engineering on a vanilla store is the cheapest honest answer, and a TSDB is
the wrong sophistication.)*

## Sources

All from the register; URLs copied exactly.

- C23 Uber, the billion data point challenge (Uber Engineering, FP): https://www.uber.com/us/en/blog/billion-data-point-challenge/
- C23 Uber, M3: Uber's Open Source, Large-scale Metrics Platform for Prometheus (Uber Engineering, FP): https://www.uber.com/us/en/blog/m3/
- C23 Packt, Uber open sources its large scale metrics platform, M3 for Prometheus (press carry, PR): https://www.packtpub.com/en-in/learning/tech-news/uber-open-sources-its-large-scale-metrics-platform-m3-for-prometheus
- C24 Netflix Atlas docs, overview (Netflix, FP): https://netflix.github.io/atlas-docs/overview/
- C24 Netflix, Introducing Atlas: Netflix's Primary Telemetry Platform (Netflix TechBlog, FP): http://techblog.netflix.com/2014/12/introducing-atlas-netflixs-primary.html
- C24 Netflix, Improved Alerting with Atlas Streaming Eval (Netflix TechBlog, FP): https://netflixtechblog.com/improved-alerting-with-atlas-streaming-eval-e691c60dc61e
- F09 NexusTrade, I went through hell and back migrating to TimescaleDB (NexusTrade, SR): https://nexustrade.io/blog/i-went-through-hell-and-back-migrating-to-timescaledb-i-didnt-last-two-weeks-20251003
