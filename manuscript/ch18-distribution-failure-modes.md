# Chapter 18: Distribution's own failure modes

## 1. Context: the price is paid after the migration

Part III opened with the backup class (Ch. 17), where the store was running and
nothing was being changed. This chapter's three rows share a different shape:
the migration succeeded, the system ran, and the failure grew inside what
distribution itself had added. The BOOK-PLAN's row 18 names the idea: "Adopting
distribution means adopting its failure model: unmonitored shard growth,
split-brain during brief partitions". The survey's cross-failure statement is
this chapter's warrant, in one sentence: "Unmonitored shard growth (Foursquare)
and failover split-brain (GitHub) are categories, not one-offs; adopting
distribution means adopting its failure model" ([corpus/SURVEY.md](#sources),
"The pattern across all failures"). ANALYSIS's failure-class cell gives the two
operational rows their mechanisms: "FC2 distribution's failure modes: F01
(unwatched shard growth; page-size/doc-size ratio), F03 (split-brain during
43s partition)". The P-index's row is the family name: "Distribution creates
new failure modes (unmonitored shard growth, split-brain failover)".

The plan's row also schedules a third row, F10, Slack's workspace-per-shard
design mistake, with P5 as its pattern: where FC2's rows are failures of
*operating* a distributed store, F10 is a failure of the single decision that
made the store distributed in the wrong shape. The chapter map's line omits F10
(and P5), the divergence first recorded in Ch. 13's review (session 29); this
chapter follows the plan, per the convention that has governed every such flag
since Chapter 6.

One migration-side thread ties Part II onward: P19's reading lens and P20's
reversibility discipline are about surviving the cutover. This chapter reads
the period after survival: the shard nobody was asked to watch, the failover
that was never tested against a real partition, and the key that could not be
re-chosen without a second migration.

## 2. The three rows

| Failure | Service | Year | The failure in one line | What distribution added | Evidence |
|---|---|---|---|---|---|
| F01 | Foursquare | 2010 | uneven shard growth outran RAM; the fix (a third shard) freed none, 11h outage | unmonitored growth + page-size physics on shared-nothing MongoDB | FP |
| F03 | GitHub | 2018 | a 43s network partition triggered Orchestrator split-brain; 24h11m degraded | an untested failover topology mis-aligned with the application | FP |
| F10 | Slack | 2017-2020 (3-year correction) | whole-workspace-per-shard key became immovable hotspots; Vitess re-shard by channel (Ch. 13) | a coarse tenant key: distribution inherited the wrong unit | SR/FP |

The evidence column is strong this time: F01 has the first-party blog plus the
friendliest post-mortem thread of the corpus (press-carried by InfoQ; the
thread's post-mortem itself posted by 10gen's Eliot Horowitz with Foursquare's
permission, quoted directly); F03 is a first-party post-incident analysis that names its
own automation and its own decision; F10 is self-reported and first-party at
once, the HN comment by the Chief Architect and the engineering post that
executed the correction. Chapter 13 already read F10's correction whole ("the
seductive wrong key" section); this chapter cites that record and reads only
what the failure class needs.

## 3. The mechanics, case by case

### Foursquare: the shard nobody was watching (F01)

InfoQ's summary line carries the shape: "Foursquare recently suffered a total
site outage for eleven hours. The outage was caused by unexpected uneven growth
in their MongoDB database that their monitoring didn't detect. The system
outage was prolonged when an attempt to add a partition didn't work due to
fragmentation, and required taking the database offline to compact it". The
first-party blog is unguarded in both directions: "we were down for about 11
hours, which is unacceptably long", and, on the cause, "one of these shards was
performing poorly because a disproportionate share of check-ins were being
written to it". For the next hour and a half the team tried to rebalance, "None
of these things worked", and the fix-by-growth had its canonical turn: a new
shard was introduced, but only some data was moved, per InfoQ's account the
team was "aiming to transfer 5% of the system data so all shards would fit
within RAM", and Horowitz's own words on the constraint: "we were trying to
move as little as possible to get the site back up as fast as possible".

The mechanism that made the third shard useless is the page-size arithmetic the
register's row preserves. High Scalability's reading of the team's thread: "Foursquare
checkins are small, 300 bytes, MongoDB uses 4K pages, so when you move a
checkin the 4K page will still be allocated, the result is no memory actually
freed. Memory will only really be freed when all data is moved off a page."
The mongodb-user thread (the post-mortem 10gen's Eliot Horowitz posted with
Foursquare's permission, carried in the register) spells the capacity rule the
incident taught: "In some cases you need all your indexes in ram, in other
cases it can just be a small working set." The survey's
dossier 23 sets the boundary this chapter keeps: so this is less "wrong engine
chosen" than "distribution adopted without the monitoring and capacity
discipline distribution requires." The blog's closer is the afterlife: "we
determined the most effective course of action was to re-index the shard",
about five hours of work, and "Because of our safeguards and extensive
backups, no data was lost." The apologize-honestly coda ("For the 32 of us
working here, that's quite humbling. We're really sorry.") is the same
transparency posture Ch. 17 found in GitLab's post-mortem.

### GitHub: 43 seconds of partition, 24 hours of degradation (F03)

GitHub's post-incident analysis (published October 30, 2018, on the October 21
incident) opens with the arithmetic gap that gives this chapter its spine:
"Connectivity between these locations was restored in 43 seconds, but this
brief outage triggered a chain of events that led to 24 hours and 11 minutes
of service degradation". The automation is
named first-party: "We use Orchestrator to manage our MySQL cluster topologies
and handle automated failover", built "on top of Raft for consensus", with the
design caveat the incident then executed: "It's possible for Orchestrator to
implement topologies that applications are unable to support, therefore care
must be taken to align Orchestrator's configuration with application-level
expectations."

During the partition, "Orchestrator, which had been active in our primary data
center, began a process of leadership deselection, according to Raft
consensus", and "The US West Coast data center and US East Coast public cloud
Orchestrator nodes were able to establish a quorum and start failing over
clusters to direct writes to the US West Coast data center." When connectivity
was restored, "our application tier immediately began directing write traffic"
to new primaries, and the split-brain geometry, writes accepted on both coasts,
became the incident's real cost. The company's own choice is recorded in its
own words: "the 30+ minutes of data written to the US West Coast data center
prevented us from considering options other than failing-forward in order to
keep user data safe", closing with the verdict the survey carries forward:
"We believe that the extended degradation of service was worth ensuring the
consistency of our users' data." The blast radius was wide but bounded:
"For the majority of the incident, GitHub was also unable to serve webhook
events or build and publish GitHub Pages sites", and "Ultimately, no user data
was lost; however manual reconciliation for a few seconds of database writes is
still in progress." The survey's dossier 25 lesson is the FC2 tooling rule:
failover automation must be tested against partitions, not just dead nodes:
split-brain during a *brief* partition is the case the tooling misses.

### Slack: the key that chose the failure (F10)

Slack's row is the design root the other two rows grew out of: the workspace as
the shard unit. The register's row records the mechanism (whole-workspace-per-shard
assumption corrected via Vitess re-shard by channel); Chapter 13 read the
correction whole. The failure side, in the Chief Architect's own words (HN,
May 18, 2021, "kmavm", whose self-statement "I was Chief Architect at Slack
from 2016 to 2020" pins the identity): "The assumption that tenants are
perfectly isolated is actually the original sin of early Slack infrastructure
that we adopted Vitess to migrate away from." The engineering post demonstrates
the damage in production terms: "Our migration to Vitess began in 2017 and
Vitess now serves 99% of our overall query load", and, on the hot spots: "We
found that we were hitting some major hotspots, while also massively
underutilizing the majority of our database fleet." The register's coarse-tenant
mechanism is verbatim on the page: enterprise onboarding meant "An unfortunate
outcome with this architecture was that we were unable to spread the load of
these large customers across the fleet". ANALYSIS's P4 cell carries the
boundary in half a line: "F10/P5: the tenant exists but chosen coarse".

## 4. What the three share: the failure model is in the adoption

Read side by side, the rows separate along the axis the BOOK-PLAN's idea names:
which part of the distribution's failure model was left unowned?

| Row | What was adopted, unowned | The moment it fired | What ownership would have required |
|---|---|---|---|
| F01 | capacity physics of shared-nothing (page-size waste, uneven growth) | the third shard freed no RAM | per-shard working-set monitoring and a growth headroom model |
| F03 | failover automation's topology choices | a 43s partition, quorum on the wrong coast | partition drills against the actual topology, app-aligned config |
| F10 | the shard key's granularity | enterprise tenants hitting hotspots | key designed at the hot unit, not the org-chart unit |

F01 and F03 are operating failures: the system was distributed and sound until
an unwatched inequality or an untested path turned a 300-byte document and a
43-second blip into an 11-hour outage and a 24-hour degradation. F10 is the
design failure both depend on: a wrong key makes even healthy operations
carrying a load that cannot be spread, and the correction is Chapter 13's whole
re-shard program rather than a config change. The class's verdict (assessment):
distribution did not kill these services; unowned parts of distribution's
failure model did, and each was visible in principle before it fired, in
mongostat output, in a partition drill, in a tenant-size histogram.

## 5. The patterns: FC2 and P5

**FC1's sibling rule, FC2**, at P-index grain: "Distribution creates new
failure modes (unmonitored shard growth, split-brain failover)", faces "F01,
F03" per the P-index. ANALYSIS's cell adds the mechanisms in parenthesis:
F01's "(unwatched shard growth; page-size/doc-size ratio)" and F03's
"(split-brain during 43s partition)". The survey's own generalization, quoted
in §1, promotes the pair from incidents to categories.

**P5, the seductive wrong key**, at P-index grain: "the tenant is the shard"
is the most seductive wrong choice in SaaS, with F10 as the registered face
('F10 (Slack workspace-per-shard, its "original sin")') and the fails-when in
two fragments split at the source's em dash: "Tenant granularity is coarser
than the hot unit", "big tenants become immovable hotspots". ANALYSIS's P5 row
couples it to the survey's correction record: "F10 + Slack's own 3-year
correction (P19 discipline applied)".

**The operations frontier.** Ch. 17's FC1 owns the copies; this chapter's FC2
owns distribution's own modes; the frontier against Ch. 19's rows is process:
F07's typo'd runbook and F08's unprofiled internals are failures of
verification layers, not of distribution's model, and F03's own tooling rule
(test against partitions, not just dead nodes) is the bridge into the runbook
class.

## 6. Spec state and what the chapter hands forward

**Spec state.** BOOK-PLAN's row 18 schedules "F01, F03; F10" with patterns
"FC2, P5"; the chapter map's line reads "F01, F03 (FC2)", omitting both F10
and P5, the divergence recorded since Ch. 13's review (session 29). The
chapter follows the plan; the map correction is pending the next planning
pass. The row's title in the plan is "Distribution's own failure modes"; this
chapter's headline follows it. No non-register sources were needed: all seven
sources are register URLs, fetched byte-identical this session (the
mongodb-user thread and the HN page render at the 32,793-char fetch cap, with
the quoted spans verified inside the fetched portions).

**Forward pointers.** Ch. 19 reads the process and verification rows (F07,
F08; FC3, FC4); Ch. 20 closes the taxonomy with the communication class
(F09-F12), where F10's other half, the published justification, already waits;
Ch. 21's driver census will count these rows' drivers against the migration
chapters' set, and F03's "failing-forward" choice is the class's cleanest
example that a driver can be correct and still expensive; Ch. 22's playbook
inherits the partition drill and the per-shard alarms as rungs.

## Exercises

●○○ (read the evidence column). The rows split 2 FP / 1 SR-FP. For each row,
name what a reader may assert from its class, and say why F10 gets a double
label. (Answer: FP rows carry the operator's own timeline and mechanisms; F10's
HN comment is self-reported speech by the former Chief Architect, and the
Slack engineering post is first-party, so the row is SR for the comment's
self-assessment and FP for the correction record. The register's row decides.)

●●○ (apply P5's fails-when). Using the two fragments split at the source's em
dash, say why workspace-per-shard was not merely unlucky but precisely the
fails-when case, and quote the one production sentence that shows it.
(Answer: the tenant existed and was chosen, but coarse: "Tenant granularity is
coarser than the hot unit", so "big tenants become immovable hotspots". The
production sentence: "An unfortunate outcome with this architecture was that
we were unable to spread the load of these large customers across the fleet".)

●●● (design the partition drill). A team runs Orchestrator-style automated
failover for a multi-coast MySQL fleet. Draft the test program that would have
caught GitHub's topology mis-alignment before the incident. (Worked answer at
the survey's level of detail: a scheduled fault-injection drill that severs
inter-coast connectivity for tens of seconds, not minutes; an assertion that
the surviving quorum and the application tier agree on the failover destination;
a check that no topology Orchestrator selects requires cross-country writes on
the app's latency budget; and a reconciliation playbook rehearsed for the
split-brain case, since F03's cost was the 30+ minutes of writes that forced
the fail-forward choice.)

◆ (pattern diagnostic). A team's sharded service publishes per-shard dashboards
and has never rehearsed a partition. Which of this chapter's two failure modes
is it exposed to, and which chapter's pattern row would have named the risk at
design time? (Answer: both, and they are not symmetric: the unwatched-growth
exposure is monitoring debt, FC2's first mode; the untested-failover exposure is
tooling debt, FC2's second, with the P-index row "Distribution creates new
failure modes (unmonitored shard growth, split-brain failover)" naming both at
registration time. The design-time pattern row is P5: if the shard key was
chosen at the org boundary rather than the hot unit, the service carries F10's
exposure too.)

## Sources

All URLs byte-copied from `corpus/REGISTER.md`:

- F01 InfoQ report (PR): https://www.infoq.com/news/2010/10/4square_mongodb_outage/
- F01 Foursquare blog (FP, archived): https://web.archive.org/web/20131114075609/http://blog.foursquare.com/2010/10/05/so-that-was-a-bummer/
- F01 High Scalability analysis (PR): http://highscalability.com/blog/2010/10/15/troubles-with-sharding-what-can-we-learn-from-the-foursquare.html
- F01 mongodb-user thread (FP thread post-mortem, Horowitz/10gen): https://groups.google.com/g/mongodb-user/c/UoqU8ofp134
- F03 GitHub post-incident analysis (FP): https://github.blog/news-insights/company-news/oct21-post-incident-analysis/
- F10 HN comment by kmavm (SR): https://news.ycombinator.com/item?id=27199229
- F10 Slack Engineering Vitess post (FP): https://slack.engineering/scaling-datastores-at-slack-with-vitess/
