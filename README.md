# The Storage Decisions Atlas

**Real-World Case Studies in Choosing and Switching Data Stores**

This repository is the **main source of truth and resumption point** for the Storage
Decisions Atlas book project. If this repo and anything else disagree, the repo wins.

## What this project is

A source-backed, case-study-driven book about how real companies chose, implemented,
transitioned between — and failed with — their data stores. Every claim traces to a
cited source; every hindsight judgement is explicitly labelled as an assessment.
The corpus covers ten database families: 66 documented cases (C01–C66), 30
documented inter-family transitions (T01–T30), and 12 failure post-mortems
(F01–F12), with multiple compared cases per topic.

## Status (updated 2026-11-26, session 30)

**Chapters 1-14 are complete: Chs. 1-13 AUTHORED + REVIEWED (Part I complete,
session 23; Part II: Ch. 11 reviewed session 25, Ch. 12 reviewed session 27,
Ch. 13 reviewed session 29), and Ch. 14 The NewSQL absorptions AUTHORED +
AUTHOR-CHECKED session 30 (T03, T04; C25, C26, C27, C28; + the R6 T25 Zomato
reverse flow; P9, P11, P15; battery 10/10 URLs register-copied, 26/26 live
source spans verbatim, 14/14 corpus fragments; chapter stands at 5a87137b).
Next unit: Phase 3 adversarial review of Ch. 14, then Chapter 15 (The
engine-swap twins; T07, T08, T16; P12).**
The R6 fold-in of SURVEY dossiers proceeds at authoring time
(Ch. 1's C56/C61 done as dossiers 22a/22b). This repo remains self-sufficient
for a cold resume from GitHub alone.

## Cold resume protocol (fresh chat, this repo alone)

This repo is deliberately self-sufficient: all durable state lives in `PROJECT.md`
and `corpus/` — no external chat history is needed to continue.

1. Read this file, then `PROJECT.md` (current state, P-index, chapter map, session
   log, protocol), then `corpus/STATE.md` (rounds R1–R6, deferrals D1–D8,
   watch-lists W1–W5), then `corpus/CONVENTIONS`-adjacent rules in
   `CONVENTIONS.md`.
2. Rebuild local mirrors **before** any editing:
   - Fetch every file with `get_file_contents` and mirror it byte-identically
     into the local Knowledge topic `repo/` folder (the platform may prune
     root-level staging; durable masters are the repo and that mirror).
   - If the platform lost the Knowledge topics entirely, recreate them from this
     repo: the book topic from `PROJECT.md`, the survey topic from
     `corpus/STATE.md`. Conflicts always resolve in favor of the repo.
3. Mark a session-log row IN PROGRESS in `PROJECT.md`, complete the unit, then
   ALWAYS-SYNC: push records in the same session and byte-verify (repo file
   sizes vs local byte counts, e.g. `wc -c`).
4. Register-first discipline: never register cases from memory; copy exact URLs
   from `corpus/REGISTER.md` / `corpus/SURVEY.md`; evidence levels FP/VP/PR/SR;
   vendor-hosted accounts are VP; hindsight is labelled "Verdict (assessment)".
5. Tooling: GitHub App connector (`push_files`, `get_file_contents`), owner
   `dsmistral`, repo `storage-decisions-atlas`, branch `main`. The App cannot
   create repositories (this one was created manually by the user).

## Repo map and reading order (cold start)

| Path | What it is | Read when |
|---|---|---|
| `PROJECT.md` | Project record: current state, Master Pattern Index (P1–P20), chapter map, session log, working protocol | **First.** This is the resume entry point |
| `CONVENTIONS.md` | Evidence ladder, gathering rules, chapter structure, records discipline | Before authoring or gathering |
| `corpus/STATE.md` | Survey ledger: research rounds, deferrals, watch-lists | Before any new research |
| `corpus/REGISTER.md` | Master register of every case (C-series), transition (T-series), and failure (F-series) with evidence level + exact source URLs | Before writing any chapter |
| `corpus/ANALYSIS.md` | Per-pattern firing evidence: which cases instantiate each P-index pattern | When designing chapters |
| `corpus/SURVEY.md` | Master copy of the research report (the readable survey the book is built from) | Reference |
| `manuscript/00-front-matter.md` | Title page, preface, how-to-read, TOC | Authoring starts here |
| `manuscript/BOOK-PLAN.md` | Chapter specs: scope, cases (register IDs), structure per chapter | Before each chapter |

## Rules

1. **The repo is the master of record.** The platform may prune or re-normalize local
   canvases; read from the repo for anything not freshly authored in this session,
   and resolve every conflict in favor of the repo.
2. **ALWAYS-SYNC.** Every record edit is pushed in the same session it is written.
   The repo is never left stale at session close.
3. **No fabrication.** Every factual claim in the manuscript traces to a source URL
   in `corpus/REGISTER.md`. Sources are never reconstructed from memory — copy the
   exact URL.
4. **Facts vs assessments.** Sourced facts carry citations; hindsight judgements are
   labelled "Verdict (assessment)". The two are never mixed.
5. **Extend, never rewrite.** Published chapters are extended or corrected via a
   revision appendix, not silently rewritten.
