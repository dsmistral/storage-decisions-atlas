# The Storage Decisions Atlas
## Real-World Case Studies in Choosing and Switching Data Stores

*(front matter draft — v0, bootstrap)*

---

### Preface (draft)

Every database migration story you have read is a survivor's story. The companies
that blog are the extremes: the pioneers whose choices became products, and the
disasters whose choices became post-mortems. The silent majority — teams that
chose boring technology and quietly served more traffic than the distributed
deployments — almost never write.

This book is built from the public record, register-first: 47 documented cases
across ten database families, twenty documented transitions between families,
and twelve failures, each with its evidence level and exact sources. It asks the
same four questions of every case: what did they choose, why (really), how did
the migration go, and — with hindsight — could they have done better? The
hindsight answers are labelled *Verdict (assessment)* because they are judgement
from evidence, not sourced fact; everything else traces to a cited source.

What the cases jointly teach is that engine selection is rarely the decision.
The decisions that mattered were the sharding key, the backup architecture, the
verification layer, the rollback path, and the team's operational depth. Those
are the subjects of this book.

### How to read this book

- **Part I (families)** is readable in any order; each chapter compares 2–4
  production cases of one database family and states the conditions under which
  that family works.
- **Part II (transitions)** covers the documented moves *between* families —
  including the quiet returns and round trips — via the transition matrix.
- **Part III (failures)** is organized by failure class, not by company: backups,
  distribution, process, verification, wrong choices.
- **Part IV (synthesis)** distills the five real drivers, the migration playbook,
  and the decision framework.
- Patterns are referred to by ID (P1–P20); the Master Pattern Index is in the
  project record and Appendix A. Cases are referred to by register ID
  (C-series, T-series, F-series), resolved in Appendix A.

### Table of contents

**Part I — The families**
1. Relational: three ways to stay relational
2. Document: three reasons, one aggregate
3. Key-value: one pattern, three scales
4. Wide-column: three fleets and the JVM cure
5. Columnar: fan-in and fan-out
6. Graph: three jobs
7. Time-series: the family both giants built
8. NewSQL: three triggers, one destination
9. In-memory: memcached plus your ops wrapper
10. Vector: where the index lives

**Part II — The transitions**
11. The transition matrix: how to read it
12. Relational exits
13. Scale-out relational done right
14. The NewSQL absorptions
15. The engine-swap twins
16. Returns and round trips

**Part III — The failures**
17. Backup failures: the class that kills
18. Distribution's own failure modes
19. Process and verification failures
20. Wrong choices, wrong transitions

**Part IV — Synthesis**
21. The five real drivers
22. The migration playbook
23. The decision framework

**Appendices**
A. The case register · B. Methodology and the evidence ladder · C. Open questions, deferrals, and watch-lists
