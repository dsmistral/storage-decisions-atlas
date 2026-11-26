# Chapter 9 - In-Memory: Memcached Plus Your Ops Wrapper

**Governing idea.** Nobody runs vanilla memcached at scale. The family's
real-world unit is memcached plus your ops wrapper: a hardened fork, a
proxy, a replication layer, an invalidation scheme, some combination that
the operator owns. The deeper question the wrapper answers is who owns
correctness. A cache is a best-effort store: it may lose or evict data
without notice, so the authoritative store must survive the cache being
empty, and the correctness model must survive the cache being stale. This
chapter reads five deployments. Three are the memcached family itself at
giant scale (Netflix's EVCache, Facebook's Memcache, Twitter's Twemcache);
two are the purpose-built extension that appeared when money flows moved
into the same latency class, Aerospike run on-premises in India by PhonePe
and Flipkart, where the ops wrapper becomes the whole job and the team that
runs it is smaller than you would think.

**Cases in this chapter:** C29 (Netflix EVCache, first-party wiki), C30
(Facebook Memcache, first-party paper), C31 (Twitter Twemcache, first-party
post), C57 (PhonePe Aerospike, vendor story plus first-party fraud-model
post), C60 (Flipkart Aerospike, vendor account of first-party talks).
Register IDs resolve in Appendix A; every source URL is copied from
`corpus/REGISTER.md`.

---

## 1. Context: the fit test the family imposes

The in-memory family is the only one in this book whose products advertise
imperfection. A cache is not wrong when it loses data; that is the contract.
The engineering question is never "how do we keep the cache from failing"
but "what happens when it does". The corpus's fit test for the family, from
the survey: the authoritative store must survive the cache being empty, and
the correctness model must survive the cache being stale. Every deployment
in this chapter is an answer to one half of that test, and the deployments
disagree about which half is harder.

The book plan's spec for this chapter names the two structural facts: that
nobody runs vanilla memcached at scale, the real-world unit is the hardened
fork plus proxy; and that the split worth tracking is who owns correctness,
the application or the infrastructure. The R6 extension adds the third
fact: when the cache stops being a cache and becomes the latency floor for
payments, the family converges on purpose-built engines run by the
operator's own team, and the "who owns correctness" question inverts,
because the engine now owns it.

## 2. The cases, each against its time

### Netflix's EVCache: the wrapper as product (C29)

Netflix's EVCache wiki carries the family's value statement: caching
"reduces the number of servers needed to handle the requests as most of the
requests are served by the cache" ([C29: Netflix EVCache
wiki](https://github.com/Netflix/EVCache/wiki), FP). What EVCache adds to
memcached is exactly the wrapper stack the spec predicts: "a distributed,
replicated cache with a simple memcached-semantics" surface (same source),
data "sharded across the cluster based on Ketama consistent hashing", and a
multi-zone deployment with zone affinity, where a cluster's nodes can sit
"in the same availability zone or spread out across multiple availability
zones" (same source). On the copies count the wiki
is concrete: "Any number of copies of data are supported (some clusters
run with 2, others with 9)" (same source), which the register summarizes
as 2 to 9 replicas per cluster (register C29). On zones, the wiki's
multi-zone deployment with zone affinity sends a client's reads to its
local zone while "the writes are done on both the zones", so "data is
replicated across both the zones thus increasing its availability"; when
instance failure or eviction costs data, zone fallback can fetch it from
the other zone, at higher latency (same source). Note the split's shape
here: EVCache's replication owns redundancy, not correctness against
staleness; nothing in the wiki claims the cache handles staleness, and per
the governing test it need not, because in the memcached lineage the app
pays that tax (Section 3).

### Facebook's Memcache: the science of app-owned correctness (C30)

Facebook's NSDI 2013 paper is the family's scientific record, and its
abstract gives the scale with unusual directness: the system "handles
billions of requests per second and holds trillions of items" for "the
world's largest social network" ([C30: USENIX NSDI'13, Scaling
Memcache at Facebook](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala),
FP). The contract statement is on the same conference page but outside the
authors' abstract: it comes from the page's Public Summary, by Michael
Piatek, which reads "Memcached may lose or evict data without notice.
Replication and cache invalidation are not transactional, providing no
consistency guarantees." before asking outright: "Won't such an imprecise
system be incomprehensible to developers and impossible to operate?" (same
source). The register's summary records the design consequence: app-owned
invalidation, with the survey's comparison noting that "Facebook's paper
made the application own invalidation around a best-effort store" (SURVEY
comparison, graded assessment). Where Netflix wrapped memcached with
infrastructure redundancy, Facebook pushed correctness up into the
application, running a system the public summary itself concedes: "the
semantics of the system as a whole are difficult to state precisely" (same
source).

### Twitter's Twemcache: the fork and the proxy (C31)

Twitter's 2012 post is the hardened-fork datapoint: "we have hundreds of
dedicated cache servers keeping over 20TB of data from over 30 services
in-memory, including crucial data such as user information and Tweets";
"these servers handle almost 2 trillion queries on any given day (that's
more than 23 million queries per second)" ([C31: Twitter Engineering,
Caching with Twemcache](https://blog.x.com/engineering/en_us/a/2012/caching-with-twemcache),
FP). The fork existed because the team "needed a more robust and manageable
version of Memcached" (same source), and the post is fronted by one of the
family's honest corrections: an update the next morning narrowing the slab
calcification problem to "our v1.4.4 fork of Memcached" (same source). The
second half of the wrapper is Twemproxy, "a proxy for memcached protocol",
which "minimizes the connections to our backend caching servers" (same
source, since open-sourced). Fork for the engine, proxy for the fleet:
neither piece is vanilla memcached, and the register records the
multiplexing role as why the third building block exists at all.

### The purpose-built extension: PhonePe and Flipkart on Aerospike (C57, C60)

When the in-memory family's workload stopped being page acceleration and
became the payments ledger's latency floor, the memcached wrapper stack
stopped being the right shape, and the Indian scale case set is the record
of the alternative. PhonePe (C57) runs the family's most demanding
manifestation: "processing over 100 million transactions daily and serving
380 million registered users and 30 million merchants", with "hundreds of
Aerospike clusters and thousands of server nodes across three on-premises
data centers, managing over 160TB per site" ([C57: Aerospike customer
story](https://aerospike.com/resources/customer-stories/phonepe/), VP).
The company's own fraud-model post is the first-party anchor: "we process
over 2.5 billion transactions in a month and in April 2022, we set a
benchmark by processing 100 million transactions on a single day" ([C57:
PhonePe tech, fraud model](https://tech.phonepe.com/building-phonepes-real-time-transaction-model-to-prevent-frauds-a-journey/),
FP), with the model's measured outcomes: "Fraud rate was reduced by 40% on
consumer to consumer transactions" and the "Number of daily fraud
complaints reduced by 38% overall (50% in the specific segment of
high-value transactions that we are targeting)" (same source). Note what the engine is doing
here: real-time fraud checks against payment flows are a correctness job,
not a staleness-tolerant one, which is why the correctness split inverts.

Flipkart (C60) is the operations story of the same extension: "over 50
distinct use cases on Aerospike, including everything from the search bar
on the homepage to the systems behind recommendations, ads, pricing, and
inventory", together processing "an aggregate 90 million queries per second
across three data centers" ([C60: Aerospike, Flipkart's 90 million QPS
journey](https://aerospike.com/blog/flipkart-journey-90-million-qps/), VP;
an account of first-party Bangalore Summit talks, published November 2025).
The peak proof is press-grade: "Flipkart, India's homegrown e-commerce
marketplace, sustained performance of its in-house Aerospike database
platform at a rate of 95 million transactions per second (TPS) supporting
its annual flagship, The Big Billion Days", with multiple application teams
leveraging "Aerospike and its Kubernetes Operator" ([C60: Aerospike press
release](https://aerospike.com/press-release/aerospike-database-on-kubernetes-enabled-95-million-transactions-per-second/),
VP). And the posts' most transferable detail is organizational: the
throughput "is maintained by a core engineering team of fewer than ten
developers", who transformed Aerospike "from a collection of self-managed
clusters into a unified internal service layer" (same source). The third
press source in the register (BigDATAwire's coverage) dead-fetches under
this study's fetch cap and stands register-recorded, with the two
Aerospike-hosted pages live-verified here.

## 3. What happened (sourced outcomes, with evidence levels)

- **Netflix (C29, FP).** Memcached wrapped with zone-aware replication (2
  to 9 replicas per cluster per the register), Ketama sharding, and
  cross-zone propagation; "most of the requests are served by the cache";
  infrastructure owns redundancy, the app owns staleness tolerance
  ([C29: EVCache wiki](https://github.com/Netflix/EVCache/wiki)).
- **Facebook (C30, FP).** The world's largest social network's cache
  handled billions of requests per second and trillions of items over a
  best-effort store whose "may lose or evict data without notice" contract
  was compensated by app-owned invalidation
  ([C30: USENIX NSDI'13](https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala);
  the quoted wording is the page's Public Summary, by Michael Piatek;
  app-ownership per the register's record and the survey's comparison).
- **Twitter (C31, FP).** A hardened fork (Twemcache) for engine
  manageability plus Twemproxy for connection multiplexing; 20+ TB in
  memory, about 2 trillion queries a day, over 23 million QPS, 30+
  services; the post self-corrects its slab calcification claim to the
  v1.4.4 fork ([C31: Twitter Engineering](https://blog.x.com/engineering/en_us/a/2012/caching-with-twemcache)).
- **PhonePe (C57, VP + FP).** Aerospike on-premises as the payments
  latency floor: hundreds of clusters, thousands of nodes, three data
  centers, 160+ TB per site, millions of QPS; the fraud model's outcomes
  (40% fraud-rate reduction on P2P, 38% fewer daily fraud complaints,
  50% in the targeted segment) are first-party figures appended to a
  vendor-reported platform story
  ([C57: Aerospike customer story](https://aerospike.com/resources/customer-stories/phonepe/);
  [C57: PhonePe fraud model](https://tech.phonepe.com/building-phonepes-real-time-transaction-model-to-prevent-frauds-a-journey/)).
- **Flipkart (C60, VP).** 50+ use cases, 90M aggregate QPS across three
  data centers, 95M TPS sustained during The Big Billion Days on
  Kubernetes, run by a team of fewer than ten via a unified internal
  service layer ([C60: Aerospike blog](https://aerospike.com/blog/flipkart-journey-90-million-qps/);
  [C60: Aerospike press release](https://aerospike.com/press-release/aerospike-database-on-kubernetes-enabled-95-million-transactions-per-second/);
  BigDATAwire coverage register-recorded, dead fetch).

## 4. The comparison: who owns correctness, and who owns the wrapper

| Test | Netflix EVCache (C29) | Facebook Memcache (C30) | Twitter Twemcache (C31) | PhonePe (C57) | Flipkart (C60) |
|---|---|---|---|---|---|
| Engine | memcached + EVCache wrapper | memcached, best-effort | Twemcache hardened fork | Aerospike | Aerospike |
| Who owns correctness | the infrastructure for redundancy, the app for staleness | the application (invalidation) | the fork + proxy ops | the engine (real-time fraud checks) | the engine, ops by the small team |
| The wrapper | zone-aware replication, 2-9 replicas | leases, invalidation scheme | fork + Twemproxy | on-prem fleet, 3 DCs | Kubernetes Operator, service layer |
| Scale reported | most requests served by cache | billions of req/s, trillions of items | 23M+ QPS, 20+ TB | 100M+ txns daily, 2.5B/month | 90M QPS, 95M TPS peak |
| Evidence level | FP | FP | FP | VP/FP | VP |

**Cross-case comparison paragraph.** The register's five deployments trace
one family and two answers. In the memcached lineage, the engine is
deliberately imperfect and the engineering money goes into the wrapper:
Netflix bought redundancy with zones and replicas, Facebook bought
correctness with application-owned invalidation over a best-effort store,
Twitter bought manageability with a fork and a proxy.
The wrapper is the architecture. The Aerospike pair inverts the
arrangement for the money-flow workloads: correctness moves inside the
engine, the staleness contract disappears, and what remains for the
operator is fleet operations at on-premises scale, which is why the
interesting number in Flipkart's account is the size of the team that runs
it. The survey's comparison reads the memcached trio as one lifecycle,
plain memcached, then a hardened fork or wrapper, then a proxy layer; the
R6 extension reads as the point where the lifecycle ends and a
purpose-built engine takes the family's job.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Netflix (C29):** the wrapper as product, and the cleanest statement of
  the infrastructure-owns-correctness side. Worth reading next to Chapter
  7's Atlas decision: the same company drew the cache line and the
  metrics line in the same decade, both times wrapping a simple core.
- **Facebook (C30):** the family's scientific record, and the most honest
  contract in the corpus, with the paper's conference page conceding, in
  its Public Summary's own voice, that the system's semantics are hard to
  state. The app-owned invalidation decision is the reason the design
  scales; it is also an app-side tax (a discipline reading of P1; P1's
  registered face is C10) every imitator must be able to pay.
- **Twitter (C31):** the fork-and-proxy datapoint, with an unusually good
  publishing culture: the next-day self-correction narrows its own claim,
  which is precisely the evidence hygiene the corpus rewards.
- **PhonePe (C57):** the extension's production proof, with the fraud
  model's outcomes as the first-party anchor. The vendor-reported platform
  figures (160+ TB per site) carry P16's caution: unaudited, verify before
  generalizing.
- **Flipkart (C60):** the organizational proof that the extension can be
  operated by a team smaller than the traffic suggests, provided the
  platform work (the unified service layer) is done. The <10 team is the
  strongest datapoint in the corpus for P8's rule at strong scale: the right
  engine, run as a platform, replaces headcount.
- **The family's rule:** pick your correctness owner deliberately (app or
  infrastructure or engine), and then pick the smallest wrapper that makes
  that owner's job survivable; the moment correctness moves inside the
  engine, budget fleet operations, because that is all that is left.

## 6. The pattern

- **P1 - Availability-first, app-side reconciliation** (registered face:
  C10, Dynamo). This chapter carries the discipline reading: in the
  memcached lineage the app owns at least invalidation (C30's registered
  app-ownership), and the trade only works for teams with the app-side
  discipline to pay for it; the wrapper (C29, C31) is what the other teams
  buy instead. Fails-when: copied by teams without the app-side
  discipline, the trade needs semantics the app must own.
- **P8 - Team size is an architectural input** (registered faces: C05,
  C02, C03). This chapter's extension is the rule's strong-scale face:
  Flipkart's fewer-than-ten-developer team runs 90M QPS because the engine
  plus the unified service layer absorbs the operations; the rule cuts
  both ways, since the same concentration is a single point of
  organizational failure without the platform work. Fails-when: used to
  excuse avoidable learning, skill justified a move whose published
  mechanics were overstated.
- **P16 - Cost at scale drives the largest migrations** (registered faces:
  C39, C41, C46/C28). This chapter carries the caution side that ANALYSIS
  records for the R6 extension (C57/C60, "P16 caution + Ch. 9 breadth"):
  both vendors' platform figures are unaudited and vendor-reported, so
  verify before generalizing. Fails-when: cost is vendor-reported and
  unaudited.
- **T26 contrast (preview).** The same workload class has two answers in
  the corpus: PhonePe's self-run on-premises Aerospike versus Freecharge's
  move to managed DynamoDB (register T26), recorded fully in the returns
  chapter and the ops-burden driver's synthesis (Chapters 16 and 21).

## Exercises

**●○○ (identify the pattern).** A team adopts memcached for page
acceleration and delegates invalidation policy entirely to the cache
vendor's product defaults. Which registered pattern's fails-when are they
standing in, and which two of this chapter's deployments are the
alternative shapes?

*Hint:* the pattern is about what the app must own; the alternatives are the
two wrappers plus proxy and the engine-side answer.

*Worked answer (assessment):* under this chapter's discipline reading, the
pattern is P1, whose fails-when is
exactly this: the availability-first trade needs the app-side discipline,
and delegating invalidation to defaults is not ownership. The alternative
shapes are infrastructure ownership (Netflix's EVCache, C29: replication
and zones as the correctness carrier) and engine ownership (PhonePe's
Aerospike, C57: correctness inside the engine for correctness-bound
flows); Twitter's fork-plus-proxy (C31) is the manageability wrapper for a
staleness-tolerant workload.

**●●○ (apply the framework).** A payments platform plans a fraud-check
store with a 10-millisecond budget per check, evaluated against live
transaction flows. The team proposes memcached plus an app-side
invalidation scheme, citing Facebook's paper. Apply the chapter's split:
what is wrong with the fit, and what must the plan contain instead?

*Worked answer (assessment, judgment):* the fit is wrong because the
workload's checks are correctness jobs, not staleness-tolerant ones: C30's
design compensates for a best-effort store with app-owned invalidation,
but a fraud decision against live payment flows needs the engine to own
consistency (C57's shape), not a wrapper around a store that "may lose or
evict data without notice". The plan must name who owns correctness (the
engine), budget the fleet operations that ownership implies (C57's three
on-prem data centers, or a managed engine per T26's contrast), and state
P16's caution on any vendor-reported platform figure before it enters the
business case.

**●●● (steelman, then weigh).** Write the strongest case that the
memcached wrapper era was a mistake: teams spent years on forks, proxies,
and invalidation machinery around a best-effort engine, and the
purpose-built engines that existed later removed the
need. Then weigh it against the register's record, and end with the
transferable rule.

*Steelman and verdict (assessment):* (i) the steelman: the wrapper stacks
were reactive engineering, built by organizations large enough to absorb
them, and the engineering cost of app-side invalidation at Facebook
scale or a Twitter fork buys nothing that a purpose-built engine does not
give a modern team by default. (ii) the weigh: the register's record
defeats the anachronism, because the purpose-built extension is dated by
its own cases (C57's fleet is 2022-and-on, C60's account is 2025) while
the wrapper era solved the 2010s' problem with the 2010s' products, and
the wrapper logic never died: Flipkart's Kubernetes Operator and unified
service layer is the wrapper concept continued by other means. The
transferable rule: the wrapper is not overhead you remove but ownership
you place; placing it in the engine costs fleet operations, placing it in
the app costs discipline, and the mistake is leaving it unowned.

**◆ (diagnostic: no hints).** One sentence each: who should own
correctness? (i) A page-render cache at 90% hit rate, 60-second staleness
tolerated. (ii) A fraud check against live payment flows. (iii) A session
store where a whole zone's loss must not evict sessions. (iv) A rate
limiter on a <5-person team with no platform group.

*(Diagnostic answers, assessment: (i) the application, staleness is
tolerated and invalidation is an app concern, the generalist cache holds;
(ii) the engine, this is C57's shape and the wrapper is the wrong layer;
(iii) the infrastructure, zone-aware replication is C29's registered job;
(iv) the managed product's defaults plus the app's own idempotency: the
team has no capacity for a wrapper, P8 binds, and defaults are the last
owner to trust blindly, so design the app to survive both staleness and
loss.)*

## Sources

All from the register; URLs copied exactly.

- C29 Netflix, EVCache wiki (Netflix, FP): https://github.com/Netflix/EVCache/wiki
- C30 USENIX NSDI'13, Scaling Memcache at Facebook (Facebook, FP): https://www.usenix.org/conference/nsdi13/technical-sessions/presentation/nishtala
- C31 Twitter Engineering, Caching with Twemcache (Twitter, FP): https://blog.x.com/engineering/en_us/a/2012/caching-with-twemcache
- C57 Aerospike, PhonePe customer story (Aerospike, VP): https://aerospike.com/resources/customer-stories/phonepe/
- C57 PhonePe tech, Building PhonePe's Real-Time Transaction Model to prevent Frauds: A Journey (PhonePe, FP): https://tech.phonepe.com/building-phonepes-real-time-transaction-model-to-prevent-frauds-a-journey/
- C60 Aerospike, Inside Flipkart's journey to 90 million QPS: Scaling with Aerospike and Kubernetes (Aerospike, VP): https://aerospike.com/blog/flipkart-journey-90-million-qps/
- C60 Aerospike, press release: Aerospike Database on Kubernetes Enabled 95 Million Transactions Per Second on E-commerce Platform for Festive Sale (Aerospike, VP): https://aerospike.com/press-release/aerospike-database-on-kubernetes-enabled-95-million-transactions-per-second/
- C60 BigDATAwire, Aerospike Database on Kubernetes Enabled 95 Million Transactions Per Second on E-commerce Platform (BigDATAwire, PR): https://www.bigdatawire.com/this-just-in/aerospike-database-on-kubernetes-enabled-95-million-transactions-per-second-on-e-commerce-platform/