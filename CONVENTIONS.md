# CONVENTIONS.md — Evidence, gathering, authoring, and records discipline

## Evidence ladder

Every register entry and every manuscript claim carries one evidence level:

| Level | Name | Meaning |
|---|---|---|
| FP | First-party | The company itself documents it: engineering blog, paper (SOSP/VLDB/NSDI), official post-mortem, official docs |
| VP | Vendor-published | A vendor publishes a case study about a named customer (marketing-adjacent; customer facts are vendor-reported) |
| PR | Press-reported | Third-party press or analyst reporting; useful for figures and context, weaker for internals |
| SR | Self-report | A participant's personal account (HN comment, solo post-mortem) — most valuable for failure analysis, but single-voiced |

Rules:
- FP > VP > PR > SR for any contested fact; state the level when the claim rests on VP/PR/SR.
- "Verdict (assessment)" marks every hindsight judgement. Assessments never cite sources as fact.
- Cost/performance figures from the subject company are vendor- or self-reported and unaudited; say so when used.
- Never invent or reconstruct a URL. Copy exact URLs from the register (which itself only contains URLs inspected at gather time).

## Gathering rules (survey track)

- Gather register-first: an item enters the corpus only with case identity, key facts, evidence level, and exact source URLs.
- Batches of parallel searches; inspect results before registering; do not register from memory.
- Deferrals: an item that can't be sourced well enough goes to the deferral ledger in `corpus/STATE.md` (D-series), never silently dropped.
- Watch-lists (W-series): open questions worth re-checking on future market changes.
- The survey feeds the book and never edits the manuscript directly.

## Chapter structure (case-study lecture form)

1. **Context** — the company, scale, moment; why the decision was live.
2. **The choice** — what they did, and what was procurable then (judge each choice against its time, not now).
3. **What happened** — sourced outcomes, with evidence levels.
4. **The comparison** — the sibling cases in the same family/move, and the conditions under which each works.
5. **Verdict (assessment)** — hindsight, explicitly labelled, including "could they have done better."
6. **The pattern** — which P-index IDs this chapter instantiated, and its fails-when condition.
7. **Exercises** — graded: ●○○ (identify the pattern in a described scenario), ●●○ (apply the decision framework to a mid-size fictional case built only from real constraints), ●●● (critique one of the chapter's cases against its critics). At least one ◆ diagnostic per part (reader diagnoses which pattern applies, no hints).

## Records discipline

- `PROJECT.md` current-state paragraph, session log, and chapter map update every session; pushed the same session (ALWAYS-SYNC).
- Published chapters are never rewritten; corrections extend via the revision appendix.
- Pushes are verified structurally: repo listing vs local byte counts; distinctive-substring census when text-level checks are needed.
- Keep pushes split into moderate commits (system limits); split-on-trouble on any chapter that fights the tooling.
- Knowledge topics (`storage-decisions-book`, `storage-decisions-survey`) mirror the repo record for platform-side resume; the repo remains the master of record. Until the GitHub repo exists, the durable master copy lives in the `storage-decisions-book` Knowledge topic (path `repo/`) — the platform prunes non-canonical root directories between sessions.
