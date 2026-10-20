# The Storage Decisions Atlas

**Real-World Case Studies in Choosing and Switching Data Stores**

This repository is the **main source of truth and resumption point** for the Storage
Decisions Atlas book project. If this repo and anything else disagree, the repo wins.

## What this project is

A source-backed, case-study-driven book about how real companies chose, implemented,
transitioned between — and failed with — their data stores. Every claim traces to a
cited source; every hindsight judgement is explicitly labelled as an assessment.
The corpus covers ten database families, ~20 documented inter-family transitions,
and 12 failure post-mortems, with multiple compared cases per topic.

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
