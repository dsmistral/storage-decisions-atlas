# Chapter 3 - Key-Value: One Pattern, Three Scales

**Governing idea.** The key-value family has the smallest interface in the
book (get and put, one record per key), and that interface is the same at
every scale of use: single-key access, no joins, and (in the family's
foundational design) availability chosen over read-modify-write consistency.
Because the schema lever is gone, the discipline that remains is *call
discipline*: how many calls each feature makes, and how stale a call result
is allowed to be. Four documented cases, one design origin and three scales
of use, show the pattern holding steady while the cost structure changes
(P1, P2, P16).

**Cases in this chapter:** C10 (Amazon Dynamo, the design origin), C11
(Amazon Financial Ledger on DynamoDB, giant scale), C12 (Duolingo on
DynamoDB, product scale), C13 (Mixmax on DynamoDB, small scale). Register
IDs resolve in Appendix A; every source URL is copied from `corpus/REGISTER.md`.

---

## 1. Context: one interface, one decision to make

A key-value store keeps opaque values under unique keys, with no schema, no
joins, and (classically) no secondary indexes. Everything a relational or
document system spreads across tables, the key-value store concentrates into
one decision: *what is the key, and what is the value's life story?*

The family does not remove design work; it moves all of it upstream into that
single decision. That is why the family's documented record looks invariant:
the same get-and-put shape serves a shopping cart, a financial ledger, a
language lesson, and a mid-size SaaS, at four different scales. What changes
with scale is not the interface but the bill: in the managed modern form,
per-call pricing turns discipline about calls into the dominant cost lever
(P16), and in the availability-first form, discipline about conflict
resolution is the price of always-writable storage (P1).

## 2. The cases, each against its time

### The design origin: Amazon Dynamo (C10)

Amazon's retail services (cart, sessions, wish lists) ran on relational
databases that, per the paper, "would lead to inefficiencies and limit scale
and availability" ([C10: Dynamo SOSP 2007](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf), FP).

The design is the family's constitution: consistent hashing for placement,
quorum replication for availability, vector clocks to detect conflicting
writes, and (the part later histories forget) *application-side conflict
resolution* pushed to the caller: a storage-level policy, the authors judged,
would be too simplistic for any application, since Dynamo has no knowledge of
the internal structure of the values it stores. The paper's running example is
the
cart: it must accept writes during a partition or outage, and when two writes
conflict, the cart reconciles by taking the union of items.

The reason this matters structurally: availability-first storage gives up
single-writer semantics for read-modify-write cycles. Any system that copies
the interface must also copy the discipline: the application has to own the
merge policy. When Twitter's Manhattan-era services later needed uniqueness
guarantees ("one email address may get attached to two accounts"), they had
to retrofit strong consistency years after adopting the availability-first
model (C36; P1's fails-when is that case; the intake lives in the transition
chapters).

### Giant scale: the Amazon Financial Ledger (C11)

The Financial Ledger, 120 TB, moved off Oracle onto DynamoDB with latency
down 40% and cost down 70%, as part of Amazon's consumer-business Oracle exit
([C11: AWS News Blog](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/), FP; the full 75 PB, ~7,500-database exit is C46, held for Chapter 12).

Read the case through the family lens: a ledger is the workload where you
would least expect to give up relational semantics, and Amazon did not give
them up; it workload-matched the split (ledger-style access to DynamoDB,
transactional relational to Aurora, analytics to Redshift; P2's flagship
face). The residual cost surfaced elsewhere in the same exit: savepoint
semantics and procedural code that did not port cleanly (C46, PR-corroborated
at ~92% done: "intermittent timeouts"). Semantic translation, not call
volume, was that migration's true bill: a caution the family chapters
inventory as P2's fails-when evidence.

### Product scale: Duolingo on DynamoDB (C12)

Duolingo stores 31 billion items in DynamoDB to deliver lessons ([C12: AWS/Duolingo talk (Karin Tsai)](https://www.youtube.com/watch?v=fhnAvn2YxZA), VP/FP), and later published
one of the best cost-tuning accounts in the register: extending a cache TTL
cut inter-service traffic by more than 60%, and stale data in DynamoDB tables
turned out to be pure spend ([C12: Duolingo blog](https://blog.duolingo.com/reducing-cloud-spending/), FP).

This is the chapter's cost lever stated by the practitioners themselves: in a
managed key-value store the bill is call volume and storage, not licenses.
The tuning that mattered was not schema work but call discipline (how stale
a cached value may be) and data lifecycle (which keys should age out). Under
P16's lens, the case is also a rare *auditable* claim (a later blog post
reporting the measured effect of a change), which is why the evidence level
pairs VP (the talk's scale figures) with FP (the company's own cost account).

### Small scale: Mixmax on DynamoDB (C13)

A small team moved a production workload from self-run MongoDB replica sets
to managed DynamoDB ([C13: Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb), FP; the same event is transition T10, read in full in Chapter 2 and
Chapter 16).

At this scale the story inverts the usual narrative: the team did not need
DynamoDB's scale-out properties or its availability model in an extreme form;
it needed a store it did not have to nurse. The register classifies this as a
clean exit from self-run replica sets to a managed KV, the "ops wrapper"
argument at its smallest. The family lesson: the interface is invariant, and
so is the value proposition's tail (a team that fits one node can still be
better served by the family, paying the same per call, freeing its only
engineers from on-call).

## 3. What happened (sourced outcomes, with evidence levels)

- **Dynamo (C10, FP).** The design became the intellectual ancestor of
  Cassandra, DynamoDB, and most eventually consistent stores ([C10: Dynamo SOSP 2007](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf)).
- **Financial Ledger (C11, FP).** 120 TB off Oracle; latency −40%, cost −70% ([C11: AWS News Blog](https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/); cost figures vendor-reported, per P16's caution).
- **Duolingo (C12, VP/FP).** 31B items on DynamoDB; TTL tuning later cut
  inter-service traffic >60% ([C12: Duolingo talk](https://www.youtube.com/watch?v=fhnAvn2YxZA); [C12: Duolingo blog](https://blog.duolingo.com/reducing-cloud-spending/)).
- **Mixmax (C13, FP).** Production MongoDB to managed DynamoDB, completed and
  documented at small scale ([C13: Mixmax](https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb)).

## 4. The comparison: the invariant and the lever

| Test | Dynamo (origin) | Financial Ledger (giant) | Duolingo (product) | Mixmax (small) |
|---|---|---|---|---|
| The key | cart/session ID | ledger entry key | lesson/user item key | app objects |
| The value | opaque, app-merged | transaction rows | lesson state + caches | app documents |
| Consistency stance | availability first; app reconciles | transactional enough for a ledger | per-item, app-tuned | managed defaults |
| Dominant discipline | conflict policy in the app | workload split + semantic translation | call discipline (TTL, staleness) | none needed: pay the managed bill |
| Evidence level | FP | FP | VP/FP | FP |

**Cross-case comparison paragraph.** The four cases agree on the interface
and disagree on everything else, which is the family's whole point: nothing
in the interface changes from the 2007 paper to the 2024 cost blog, but the
discipline that matters migrates. At the origin it is conflict resolution in
the application (P1); in the largest migration it is workload matching plus
the semantic residue of leaving a relational system (P2); in the steady-state
product it is call and lifecycle discipline (P16); at small scale it is
mostly the decision not to operate anything (C13's managed-primitive posture,
P3-adjacent under the project's reading). A reader who takes one lesson from
the family should take this: pick the key and the value's lifecycle as
deliberately as a relational designer picks a schema, because everything the
schema used to enforce is now the key's shape and the caller's problem.

## 5. Verdict (assessment)

*Everything in this section is judgment from evidence, labelled as such.*

- **Dynamo (C10):** exemplary; nothing procurable in 2007 offered
  partition-tolerant writes, and the app-owns-reconciliation insight has aged
  perfectly. The modern critique attaches to copiers, not to the design:
  adopting always-writable storage without owning a merge policy is exactly
  the C36 retrofit waiting to happen.
- **Financial Ledger (C11):** the exemplary workload match; the cost and
  latency gains are vendor-reported and unaudited (P16's standing caution),
  and the honest accounting came from elsewhere in the same exit (savepoint
  friction, C46). Verdict: right decision, celebratory ledger of costs.
- **Duolingo (C12):** the family's most transferable case, because it prices
  the discipline precisely (>60% inter-service traffic from one TTL change);
  the scale figures are from a vendor talk, the cost account is the company's
  own (FP). Verdict: exemplary claim on the lever; watch the audit asymmetry,
  which this case partially repairs.
- **Mixmax (C13):** correct at its scale; the exit was about operability, not
  the interface. Verdict: the smallest documented vote for the family, and a
  useful antidote to reading KV as a giant-only play.

No case in the chapter argues against the family where its fit test held;
the recorded friction lives at the edges (semantic translation in C11, and
the C36 uniqueness retrofit, which is P1's fails-when).

## 6. The pattern

- **P1 - Availability-first with app-side reconciliation.** For
  partition-tolerant writes, push conflict resolution to the application,
  which knows the semantics. Face: C10's cart union. Fails-when: C36's
  retrofit, where uniqueness semantics the app could not cheaply own forced
  strong consistency back in (Chapter 12's case).
- **P2 - Workload-match the split.** Split by access pattern, not by flagship
  product. Face: C11 (ledger to DynamoDB, relational to Aurora, analytics to
  Redshift). Fails-when: assuming semantic translation is free (C46's
  savepoint timeouts; Voucherify/slice's rebuilt invariants, Ch. 2).
- **P16 - Cost at scale drives the largest migrations.** Licenses, servers
  and per-call bills, not fashion. Faces: C11, C12, with C13 as the small-scale
  echo. Fails-when: the figures are vendor-reported and unaudited (the open
  question in Appendix C); impose the like-for-like TCO test before
  believing a "70%".

## Exercises

**●○○ (identify the discipline).** A team adopts a managed key-value store
for per-user settings; six months later the bill is high and p99 reads are
slow because each page load makes 25 point reads. Which of the chapter's
three disciplines does this measure, and what are the two structural options?

*Hint:* the key is right; count what happens per request.

*Worked answer (assessment):* call discipline (P16's lever). Options: widen
the value (one item per user aggregating the settings, accepting app-side
merging: P1's stance) or add a tuned cache in front with a deliberate
staleness budget (Duolingo's TTL lesson). The wrong fix is blaming the engine.

**●●○ (apply the framework).** A payments-adjacent product needs: (a)
always-available cart updates during regional outages; (b) a guarantee that
a coupon is consumed exactly once, even under contention; (c) sub-ms reads
of user profile state. The team wants one managed key-value store for all
three. Use the chapter's patterns to say what fits, what cannot, and the
minimum split.

*Worked answer (assessment, judgment):* (a) and (c) fit the family's home
turf: always-writable state with app-side merge, and per-key hot reads.
(b) does not: exactly-once coupon consumption under contention is
read-modify-write semantics with uniqueness, P1's fails-when shape (C36's
email-duplication case); a KV value cannot enforce it without locks or
conditional writes that may themselves fail under contention. The minimum split: KV for (a)+(c),
and a strongly consistent store (relational or a conditional-write design)
for coupon redemption. This is P2 applied inside one product.

**●●● (steelman, then weigh).** Amazon published the Financial Ledger's
"latency −40%, cost −70%" as part of the Oracle-exit celebration. Write a
200-400 word assessment that (i) states the strongest case for accepting the
figures, (ii) applies P16's audit caution (what would a like-for-like TCO
comparison need to include that the blog does not report?), and (iii)
compares this with Duolingo's cost post as evidence, and takes a position on
which claim a working architect should budget against. (i)-(ii) cite the
blog and the P16 context; (iii) is Verdict (assessment).

*Suggested answer sketch (assessment):* (i) Amazon ran the system itself and
the exit was real, 120 TB is not a pilot; (ii) a like-for-like TCO would need
the Oracle license costs avoided, engineer-time in both steady states, and
the C46 friction (savepoint timeouts) priced in; the blog reports none of
these; (iii) Duolingo's post is the better budget evidence because it
isolates one lever (TTL) with a measured effect (>60%); budget call-volume
curves, not vendor percentage.

**◆ (diagnostic: no hints).** Three teams, one sentence each: name the
family's pattern and the discipline at risk. (i) A session store that must
keep accepting writes during datacenter failover; (ii) a config service
whose values change rarely and are read millions of times a minute; (iii) a
team that wants "a simple key-value store" and has never written down what
happens when two regions write the same key at once.

*(Diagnostic answers, assessment: (i) the family's home turf; discipline =
conflict policy in the app before launch; (ii) fits, but the discipline is
call discipline: cache and staleness budget before per-call pricing bites;
(iii) the pattern is fine, the discipline at risk is P1's: without a merge
policy, this is the C36 retrofit in waiting.)*

## Sources

All from the register; URLs copied exactly.

- C10 Amazon Dynamo (SOSP 2007, FP): https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf
- C11 Amazon Financial Ledger (AWS blog, FP): https://aws.amazon.com/blogs/aws/migration-complete-amazons-consumer-business-just-turned-off-its-final-oracle-database/
- C12 Duolingo (AWS/Duolingo talk, VP/FP): https://www.youtube.com/watch?v=fhnAvn2YxZA
- C12 Duolingo cost post (FP): https://blog.duolingo.com/reducing-cloud-spending/
- C13 Mixmax (FP): https://www.mixmax.com/engineering/the-great-migration-from-mongodb-to-dynamodb
