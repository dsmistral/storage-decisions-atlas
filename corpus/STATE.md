# corpus/STATE.md — Survey ledger

## Rounds

| Round | Date | Focus | Outcome |
|---|---|---|---|
| R1 | 2026-10-20 | Failure post-mortems (the user's "we learn from mistakes" ask) | 9 failures registered: F01–F07 + in-dossier mistakes; strong first-party post-mortems for Foursquare, GitLab, GitHub 2018, S3 2017, Sidekick, Ma.gnolia, Code Spaces |
| R2 | 2026-10-20 | Family flagship use cases (wide-column, columnar, in-memory, time-series, graph, vector, document, NewSQL) | Filled C14–C35; Cloudflare ClickHouse, Bloomberg, ICIJ/Zurich/ShopBot, Uber M3, Netflix Atlas, DoorDash CRDB, EVCache/FB Memcache/Twemcache, turbopuffer/Voyager, eBay Mongo |
| R3 | 2026-10-20 | Remaining transition-matrix gaps | Document→relational returns (slice, Infisical — first-party but small/mid-size); vector unwind both directions (Confident AI vs Pinecone counter-blog); Twitter Twemcache as third in-memory case |
| R4 | 2026-10-20 | Consolidation | Full report assembled as the survey master (`SURVEY.md`); family-by-family comparisons with 2–3 cases each; transition matrix ~20 rows; failure synthesis; ~100-row source notes table |

## Deferral ledger (do not re-search blindly; check these first)

| ID | Item | Why deferred |
|---|---|---|
| D1 | Apple Cassandra→FoundationDB | Press-only since 2015 (PR-level); Apple still ships Cassandra patches — register as direction, not completed fact. First-party confirmation unlikely to exist |
| D2 | Large-scale (100M+ vector) first-party unwind of a dedicated vector store | No first-party account found; both directions documented only at small/mid scale |
| D3 | Big-tech document→relational return | First-party returns exist only from small/mid teams (slice, Infisical); big-tech reversal undocumented — likely survivor gap, not absence |
| D4 | Coinbase time-series case | Searched, miss — nothing registerable found |
| D5 | Foursquare venue-db Mongo→Postgres claim | Only weakly sourced (Quora); do NOT claim without a better source. (The PG→Mongo direction IS sourced: HN, engineering lead) |

## Watch-list

| ID | Item | Trigger to re-check |
|---|---|---|
| W1 | pgvector convergence vs dedicated vector stores | Any large-scale (100M+ vectors) first-party migration published in either direction |
| W2 | Pinterest TiDB long-term outcome | Largest public "regret the build, buy the product" datapoint; outcome not yet written |
| W3 | Post-migration drift (Netflix stack today, Twitter post-2022, Uber's current Schemaless status) | Current-state claims stop at cited dates; re-verify if a chapter needs present tense |

## Standing notes

- All gather-time URLs are preserved inline in `REGISTER.md` and `SURVEY.md`; never reconstruct.
- Vendor-published cases (eBay/MongoDB, Zurich/Neo4j, DoorDash/CRLabs, Cloudflare/ClickHouse, turbopuffer's own posts) are marketing-adjacent; state VP level when used.
- Survivorship bias: nobody blogs "we migrated and it was fine"; Stack Overflow / 37signals matter as the silent majority done well.
