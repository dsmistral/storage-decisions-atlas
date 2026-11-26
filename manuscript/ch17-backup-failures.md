# Chapter 17: Backup failures: the class that kills

## 1. Context: Part III opens where the migrations end

Part II read transitions that changed something on purpose. Part III reads
failures, and it opens with the class whose defining property is that nothing
was being changed at all: the data store was running, the migrations were done
(or never attempted), and the loss rode in through the tier that exists
precisely for bad days. The BOOK-PLAN's row 17 names the single idea: "Half the
catastrophic cases are backup failures, not database failures; restore-path
diversity is the deliverable". The survey's synthesis line is the chapter's
warrant: the post-mortem set, from GitLab's silently failing backups through
Sidekick's and Ma.gnolia's missing backups to Code Spaces' shared blast radius,
"shows that storage failures are overwhelmingly operations failures: backups,
failover, monitoring, and verification, not engine selection"
([corpus/SURVEY.md](#sources), principles, line 19).

The corpus formalizes the class as FC1. The P-index's row reads "Backups are
the failure class: unverified, co-located, or in-blast-radius copies are not
backups", and its face list is this chapter's table of contents: "F02, F04,
F05, F06". ANALYSIS's failure-class cell gives each row its mechanism in one
parenthesis: "FC1 backup-failure: F02 (silently failing pg_dump, version
mismatch), F04 (no verified backup), F05 (backups on same broken stack), F06
(backups inside compromise domain)". Four phrasings, one geometry: in every
case a copy of the data existed somewhere, and in every case no copy could be
restored in time, if at all.

One migration-side thread ties Part II to this chapter. The reversibility
pattern (P20) guarded every successful move there on the requirement that the
old path survive the cutover. ANALYSIS's P20 cell stresses the boundary case:
"F06: no out-of-domain backup = nothing to return to". The migration chapters
treated the surviving old path as a planning input; this chapter reads what
happens when there is no such input, in production, at the worst moment. Two
of the four rows (F02, F06) also carry the P20 pattern's migration lesson in
their DNA: a wiped directory on the wrong host, and a recovery that shared its
credentials with the attacker.

## 2. The four rows

| Failure | Service | Year | The failure in one line | Restore that existed | Evidence |
|---|---|---|---|---|---|
| F02 | GitLab.com | 2017 | wrong-host deletion of the primary (~300 GB); all five backup channels unusable | a 6-hour-old staging backup, restored over ~18 hours | FP |
| F04 | Sidekick / Danger | 2009 | SAN work corrupted main and backup databases; ~800k users' data unreachable | month-old offsite tapes, partially, after ~2 weeks | PR |
| F05 | Ma.gnolia | 2009 | ~0.5 TB MySQL corrupted; primary and backups failed together | none; community web caches recovered fragments | PR |
| F06 | Code Spaces | 2014 | attacker deleted data, snapshots and offsite backups from the console | none; every path sat in the compromise domain | PR |

The evidence column is uneven and honest about it: F02 is first-party, the
company's own incident page with the timeline and the named backup failures;
the three 2009/2014 rows are press-recorded (PR), reconstructed from reporting
and the surviving service statements. The chapters of Part I favored
comparisons; a failure chapter compares postures, and the posture table is the
comparison this chapter can defend.

## 3. The mechanics, case by case

### GitLab: five backup channels, none reliable (F02)

The post-mortem ("GitLab.com database incident", published February 1, 2017)
begins with the loss line: "We lost six hours of database data (issues, merge
requests, users, comments, snippets, etc.) for GitLab.com". The proximate chain
is a textbook escalation of small decisions. Spam pressure came first:
"spammers were hammering the database by creating snippets, making it
unstable". Replication lagged, troubleshooting escalated, and during the
repair someone wiped the secondary's data directory to force a clean re-sync: a
planned step on the wrong side of a lagging pair. Then the wrong-host deletion:
the command meant for the now-empty secondary landed on the primary, and "After
a second or two he notices he ran it on `db1.cluster.gitlab.com`, instead of
`db2.cluster.gitlab.com`". The Availability Digest's third-party summary
registers the scale: "a technician accidentally deleted 300 GB of data from a
production database" ("GitLab Suffers Massive Backup Failure Due to a Fat
Finger", April 2017).

With the primary gone, the backup architecture answered, and the post-mortem's
own inventory is the chapter's centerpiece. Five channels, five failures:

- "LVM snapshots are by default only taken once every 24 hours".
- "Regular backups seem to also only be taken once per 24 hours, though
  team-member-1 has not yet been able to figure out where they are stored",
  and per a teammate "these don't appear to be working, producing files only a
  few bytes in size".
- The pg_dump chain, the register's F02 mechanism: the survey's dossier 24
  compresses it as "pg_dump backups had been silently failing for weeks
  (9.2-era binaries against a 9.6 database), and no disk snapshots existed",
  and the post-mortem's own bullet carries the mechanism: pg_dump "may be
  failing because PostgreSQL 9.2 binaries are being run instead of 9.6
  binaries", with the cause spelt out: "omnibus only uses Pg 9.6 if
  data/PG_VERSION is set to 9.6, but on workers this file does not exist. As
  a result it defaults to 9.2, failing silently".
- "Disk snapshots in Azure are enabled for the NFS server, but not for the DB
  servers".
- "Our backups to S3 apparently don't work either: the bucket is empty".

The summary sentence is the chapter's thesis in the victim's own words: "So in
other words, out of five backup/replication techniques deployed none are
working reliably or set up in the first place. We ended up restoring a
six-hour-old backup". The restore that worked was a staging database's backup,
outside the production stack: "We're working on recovering right now by using a
backup of the database from a staging database". Recovery took the
Digest-reported "roughly 18 hours" of the survey's dossier ("full service took
roughly 18 hours to restore"), and the Digest's arithmetic on the whole event
is "GitLab was down for about 36 hours before it was able to recover its
database". The survey's lesson line has become the chapter's shorthand: "A
backup you have never restored is a hope, not a backup".

### Sidekick: the trust that failed upward (F04)

The Sidekick outage of 2009 is the consumer-scale face of the class. Wikipedia
(retrieved live) opens: the outage "resulted in an estimated 800,000 smartphone
users in the United States temporarily losing personal data", and "At the time,
it was described as the biggest disaster in cloud computing history". The
survey's dossier 26 states the mechanism plainly: "Microsoft's Danger
subsidiary had no verified working backup; some data was restored from
month-old offsite tapes". The precipitating event, per the wiki, was "remedial
work performed by contractor Hitachi Data Systems on Danger's Storage area
network"; Microsoft's own statement called it "a confluence of errors from a
server failure that hurt its main and backup databases supporting Sidekick
users". Note the plural: the failure took the main and the backup databases
together, the same geometry Ma.gnolia will show in its purest form. The data
was mostly recovered ("Microsoft said they had been able to recover most or all
data and would begin to restore them"), but the announcement midpoint is the
class's recording: "On October 10, 2009, T-Mobile announced that personal
information stored on Sidekick phones would be permanently lost, which turned
out to be incorrect". The outages "lasted approximately two weeks". FC1's
mechanism for this row, "no verified backup", is exact: copies existed
somewhere (tapes, a backup database), but no path had ever been demonstrated
from the failure to the copy.

### Ma.gnolia: the backup shared the primary's fate (F05)

Ma.gnolia is the class's purest geometry: the redundancy existed but lived
inside the same failure domain as the primary. The Wikipedia record (the
service relaunched as Gnolia): "Ma.gnolia servers lost all data in a complete
outage on January 30, 2009. On February 17, Halff announced that due to data
corruption, all user data in the database was irretrievable, rendering the site
essentially dead". The Data Center Knowledge reporting (snippet-corroborated;
the page blocks automated fetches) carries the scale from the founder: "the
service's MySQL database included nearly half a terabyte of data", and the
recovery verdict: "a specialist had been unable to recover any data from the
corrupted hard drive". The survey's dossier 27: "the primary and its backups
went down together, and user data was never recovered". The company's own
Recovery Tools could only reach outside the broken stack, and the wiki's limit
line is the finding: the tools "rely on external sources to reconstruct users'
data, they were limited in how much data they could restore" (the
survey-corroborated measure: partial fragments at best). The afterlife closes
the case: a relaunch as the smaller invitation-only Gnolia, and shutdown
announced for September 29, 2010. The service never stopped existing because
its database engine failed; it stopped because nothing it kept could be
restored.

### Code Spaces: the compromise domain (F06)

Code Spaces is the row where the backups were real and still useless. The
company had claimed resilience in its marketing; eSecurity Planet's report
preserves the cached site's own words, with the source's ellipsis kept: "Backing
up data is one thing, but it is meaningless without a recovery plan", and the
claim's continuation, "one that is well-practiced and proven to work". In June
2014 attackers gained the AWS console credentials, and when staff tried to
reclaim the account, the attacker, per the company's own statement preserved in
the report, "proceeded to randomly delete artifacts from the panel". The
deletion inventory: "he had removed all EBS snapshots, S3 buckets, all AMIs,
some EBS instances and several machine instances", and the company's summary:
"In summary, most of our data, backups, machine configurations and offsite
backups were either partially or completely deleted". Ars Technica's headline
(dead-fetch this session; title corroborated via search snippets and the
eSecurity Planet report) scores the irony with quotation marks around the
claim: "AWS console breach leads to demise of service with 'proven' backup
plan". The company folded. The survey's dossier 28 draws the boundary that FC1
encodes: the "backup plan" was real but shared "the same single point of
compromise as production".

## 4. What the four share: the restore is the deliverable

Read side by side, the four rows separate cleanly along one axis: where did a
working copy sit relative to the failure?

| Row | Failure mode (ANALYSIS FC1) | Where the copies sat | What the restore needed that the copies lacked |
|---|---|---|---|
| F02 | unverified (pg_dump failing silently) | five channels in and around production, none exercised | timeliness and proof: the only demonstrated restore was staging's |
| F04 | no verified backup | main + backup databases together, month-old tapes | a demonstrated path from loss to copy at all |
| F05 | co-located (same broken stack) | backup copies inside the primary's failure domain | a copy in a different failure domain |
| F06 | in-blast-radius copies | all copies behind the same credentials as production | a copy outside the compromise domain |

The GitLab case is the instructive one for working engineers because it fails
comprehensively and yet survives: a staging backup, itself a restore path that
the production stack did not include, happened to be six hours old. Nobody
designed that survival; the post-mortem's own tone carries the accident. Side
by side, F05 (same stack) and F06 (same credentials) are the same finding
overlapped differently, and F04 is the warning for anyone consuming a
platform's durability story instead of operating their own verify loop.

**Verdict (assessment).** The four rows support the BOOK-PLAN's idea with one
clarification: "half the catastrophic cases" understates what the corpus shows,
because the other failure rows in Part III carry backup-shaped components too,
and the class's boundary is broader than the four-row list. What the four
prove, without exception, is that the deliverable of a backup program is the
restore: a path, exercised, timed, held outside the failure's domain and
outside the credentials of production. Every hour of the GitLab recovery and
every unrecovered bookmark in Ma.gnolia is a restore drill that was never run.

## 5. The pattern: FC1, with P20 as its migration-side guard

**FC1's rule**, at P-index grain: "Backups are the failure class: unverified,
co-located, or in-blast-radius copies are not backups". Its three adjectives
are the comparison table's rows and the chapter's four dossiers in one clause:
unverified (F02, F04), co-located (F04, F05), in-blast-radius (F06). The
positive form, per the survey's dossier lessons: restore-path diversity, "is
the actual deliverable" (dossier 27, fragments split at the source's em dash:
"Restore-path diversity", "different engine, different site, different failure
domain", "is the actual deliverable"); "separate
accounts, separate credentials, deletion that requires more than one human or one key" (dossier
28's fix list, quoted as the survey's guidance); and the drill, from dossier
24's lesson: a backup becomes a backup at its first successful, timed restore.

**P20's guard.** The migration chapters of Part II required that the old path
survive cutover; ANALYSIS's P20 cell names this chapter's scope as the price of
violating it: "F06: no out-of-domain backup = nothing to return to". The
continuity is exact: a rollback plan is a verified, out-of-domain restore, and
a backup is a rollback plan for an unwanted event rather than a planned one.
Chapters 15 and 16 kept rollback replicas (C41) and intact old paths (T13/F09)
at the center of their successful moves because the same failure mode, violated
deliberately, ends here.

**The operations frontier.** The survey's synthesis gives the class its
boundary against the rest of Part III: failures in storage are "operations
failures: backups, failover, monitoring, and verification, not engine
selection". FC1 owns the backups quarter; Ch. 19's operational rows (FC3, FC4)
and the process rows of Ch. 18 (FC2) and Ch. 20 take the others.

## 6. Spec state and what the chapter hands forward

**Spec state.** BOOK-PLAN's row 17 schedules "F02, F04, F05, F06" with pattern
"FC1", and the chapter map's line reads "F02, F04, F05, F06 (FC1)": plan and
map agree exactly this time, and the chapter records the agreement per the
convention that has been flagging divergences since Chapter 6. The chapter also
records its sourcing shape: F02's spine is first-party (the GitLab post-mortem
plus the Availability Digest's third-party analysis, both register URLs), the
three PR rows lean on the survey's dossiers 26 through 28 (corpus quotes
verified at disk) plus two live Wikipedia pages, one snippet-corroborated DCK
page and one snippet-corroborated WIRED page; the Code Spaces mechanics carry a
non-register corroborating source (eSecurity Planet, quoting the company's
statement and cached site claims), noted as not in the register, flagged for
the survey pass.

**Forward pointers.** Chapter 18 continues the failure taxonomy with the
wrong-choice class (F01, F03; FC2, plus F10 per BOOK-PLAN's row, the
session-29-flagged divergence); Chapter 19 reads the operational rows (F07,
F08; FC3, FC4); Chapter 20 closes the taxonomy with communication failures
(F09-F12, including this book's two recurring threads, F09's round trip and
F11's aged justification). Chapter 21's driver census will count these four
rows' drivers as a filter against the migration chapters' optimistic set, and
Chapter 22's playbook inherits the restore-drill requirement as its first
rung.

## Exercises

●○○ (read the evidence column). The four rows split 1 FP / 3 PR. Name the
difference in what a reader may assert from each class, then point at the
chapter's one statistic (the 800,000 users) and say which class it belongs to
and where the corpus carries it. (Answer: FP rows can carry the subject's own
timeline and named mechanisms; PR rows carry what press and press-quoted
statements support, and the register's row decides which URL count belongs to
each. The 800,000 estimate is PR-side, carried by F04's Wikipedia and survey
dossier 26.)

●●○ (apply FC1's three adjectives). Classify each of the four rows under
"unverified", "co-located", or "in-blast-radius", one primary adjective per
row, and say why F04 arguably fits two. (Answer: F02 unverified, F04
unverified-and-co-located, F05 co-located, F06 in-blast-radius. F04's backup
database sat in the same SAN under the same operations, so the copy failed
with the primary, which is co-location, and no restore path had been
demonstrated, which is the unverified half.)

●●● (design the drill). A three-node PostgreSQL service must never repeat
GitLab's five-channel collapse. Draft the restore program: which copies, where
they live, and the drill that proves each. (Worked answer at the survey's
level of detail: a nightly logical dump running on version-matched binaries
with its output size checked against a floor, a disk snapshot on the database
servers themselves (GitLab's Azure channel missed exactly this), an object
copy in a separate cloud account with separate credentials (Code Spaces'
missing separation), a staging restore rehearsed quarterly with a timed RTO,
and a one-line audit: "out of N techniques deployed, how many delivered a
restore in the last quarter?")

◆ (pattern diagnostic). A team's backup plan passes the classic audit: copies
exist, they are current, restores work. Name the Code Spaces failure mode that
still kills the plan, and connect it to the migration chapter that would have
caught it. (Answer: the in-blast-radius copy: every restore path sat behind
the same credentials and console as production, so the compromise domain
covered the backups too. ANALYSIS's P20 cell states the boundary: "F06: no
out-of-domain backup = nothing to return to"; Ch. 15's rollback replicas and
Ch. 16's surviving old paths are the same requirement applied on purpose.)

## Sources

All URLs byte-copied from `corpus/REGISTER.md`:

- F02 GitLab post-mortem (FP): https://about.gitlab.com/blog/gitlab-dot-com-database-incident/
- F02 Availability Digest analysis (PR, PDF): https://www.availabilitydigest.com/public_articles/1204/gitlab.pdf
- F04 Wikipedia (PR): https://en.wikipedia.org/wiki/2009_Sidekick_data_loss
- F04 Data Center Knowledge (PR, dead-fetch; title/snippets corroborated): https://www.datacenterknowledge.com/archives/2009/10/10/t-mobile-microsoft-lost-all-the-sidekick-backups
- F05 WIRED (PR, dead-fetch; title/snippets corroborated): https://www.wired.com/2009/01/magnolia-suffer/
- F05 Data Center Knowledge (PR, dead-fetch; snippets corroborated): https://www.datacenterknowledge.com/archives/2009/02/19/magnolia-data-is-gone-for-good
- F05 Wikipedia Gnolia (PR): https://en.wikipedia.org/wiki/Gnolia
- F06 Ars Technica (PR, dead-fetch; title corroborated): https://arstechnica.com/information-technology/2014/06/aws-console-breach-leads-to-demise-of-service-with-proven-backup-plan/

Not in the register, noted for the survey pass:

- F06 mechanics corroborator (eSecurity Planet report quoting Code Spaces' statement and cached site): https://www.esecurityplanet.com/networks/code-spaces-destroyed-by-cyber-attack/
