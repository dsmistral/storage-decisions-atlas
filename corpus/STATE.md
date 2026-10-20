# corpus/STATE.md — Survey ledger

## Rounds

| Round | Date | Focus | Outcome |
|---|---|---|---|
| R1 | 2026-10-20 | Failure post-mortems (the user's "we learn from mistakes" ask) | 9 failures registered: F01–F07 + in-dossier mistakes; strong first-party post-mortems for Foursquare, GitLab, GitHub 2018, S3 2017, Sidekick, Ma.gnolia, Code Spaces |
| R2 | 2026-10-20 | Family flagship use cases (wide-column, columnar, in-memory, time-series, graph, vector, document, NewSQL) | Filled C14–C35; Cloudflare ClickHouse, Bloomberg, ICIJ/Zurich/ShopBot, Uber M3, Netflix Atlas, DoorDash CRDB, EVCache/FB Memcache/Twemcache, turbopuffer/Voyager, eBay Mongo |
| R3 | 2026-10-20 | Remaining transition-matrix gaps | Document→relational returns (slice, Infisical — first-party but small/mid-size); vector unwind both directions (Confident AI vs Pinecone counter-blog); Twitter Twemcache as third in-memory case |
| R4 | 2026-10-20 | Consolidation | Full report assembled as the survey master (`SURVEY.md`); family-by-family comparisons with 2–3 cases each; transition matrix ~20 rows; failure synthesis; ~100-row source notes table |
| R5 | 2026-10-21 | User-directed freshness pass before authoring: watch-list refresh (W1–W3), thin-chapter strengthening, Ch. 1 pre-verification | 6 new cases (C48–C53: Uber Docstore, Netflix CRDB portfolio, 37signals S3 exit, Netflix Cassandra current-state, Tinybird CH ops, 37signals Fizzy SQLite-per-customer abandoned); 2 new transitions (T21 Langfuse→CH, T22 Schemaless→Docstore); W2 refined (TiDB confirmed in production Oct 2026 + migration-mechanics post); W3 substantially resolved for Netflix + Uber; Ch. 1 inputs verified current (no newer GitHub DB posts; Craver caveat logged). Honest negatives: no first-party graph case beyond ICIJ/Zurich/ShopBot (D6); no first-party document-family case beyond Stripe/LinkedIn found (D7); no first-person 100M+ vector migration in either direction (W1 stays open) |
| R6 | 2026-10-21 | User-directed verification pass against a third-party report's case list ("did you find these too?") | 9 of the report's items already in corpus (Discord C38/T07, Uber C37/T11, Slack F10→formalized as C66, Cloudflare T05, Amazon C46, Dropbox C39, 37signals C50, Pinterest C26, Infisical C45). 13 new cases registered (C54–C66: Figma, Notion, Zerodha, PhonePe, Zomato, Freecharge, Flipkart, OpenAI, Yelp, Deliveroo, Bhoos, Voucherify, Slack-Vitess formalization); 8 new transitions (T23–T30: Yelp PG→MySQL standardization, Deliveroo PG→DynamoDB, Zomato TiDB→DynamoDB, Freecharge Aerospike→DynamoDB, Bhoos DynamoDB→PG, Uber+Zerodha ES/ELK→CH, Zerodha Timescale→CH, Zerodha PG→CH); T19 extended with Voucherify; Stripe DocDB and Pinterest HBase→TiDB details confirmed already covered (C07/C26). Honest negative: D8 (Razorpay/Swiggy). W5 added (OpenAI write-scaling). Zerodha's claimed "PG→CH 20TB" verified first-party via z-connect (T30), not the earlier "experimenting" line in the 2023 Postgres post. SURVEY.md dossiers for R6 cases pending — fold in at authoring time |

## Deferral ledger (do not re-search blindly; check these first)

| ID | Item | Why deferred |
|---|---|---|
| D1 | Apple Cassandra→FoundationDB | Press-only since 2015 (PR-level); Apple still ships Cassandra patches — register as direction, not completed fact. First-party confirmation unlikely to exist |
| D2 | Large-scale (100M+ vector) first-party unwind of a dedicated vector store | No first-party account found; both directions documented only at small/mid scale |
| D3 | Big-tech document→relational return | First-party returns exist only from small/mid teams (slice, Infisical); big-tech reversal undocumented — likely survivor gap, not absence |
| D4 | Coinbase time-series case | Searched, miss — nothing registerable found |
| D5 | Foursquare venue-db Mongo→Postgres claim | Only weakly sourced (Quora); do NOT claim without a better source. (The PG→Mongo direction IS sourced: HN, engineering lead) |
| D6 | First-party production graph case beyond ICIJ/Zurich/ShopBot | R5 sweep found only vendor/marketing material; re-check later — graph companies rarely blog first-party |
| D7 | First-party document-family production case beyond Stripe/LinkedIn | R5 sweep found only how-to content and Reddit; eBay remains the strongest (VP); re-check later |
| D8 | Razorpay / Swiggy database migrations | R6 sweep (from the user's third-party report): no genuine engineering write-ups found — hits were unrelated or a student paper. Do not cite either company as a storage-decisions case without a first-party account |

## Watch-list

| ID | Item | Trigger to re-check |
|---|---|---|
| W1 | pgvector convergence vs dedicated vector stores | Still OPEN after R5: no first-person 100M+ migration either direction. Third-party consensus (PR-level, unverified): pgvector competitive below ~10–50M vectors; pgvectorscale claims to extend the range (Mar 2026, vendor). Re-check on any large-scale first-party migration |
| W2 | Pinterest TiDB long-term outcome | REFINED R5: TiDB confirmed in production use (Oct 2026, InfoQ: petabyte CDC pipelines from MySQL, TiDB, KVStore); zero-downtime HBase→TiDB migration mechanics published. Long-term regret/verdict still pending — keep watching |
| W3 | Post-migration drift (Netflix, Twitter, Uber) | SUBSTANTIALLY RESOLVED R5: Netflix — Cassandra powers Member/Billing/Recommendations/Subscriptions as of May 2026, with KV/TS/Graph abstractions + CRDB in the portfolio (C51, C49); Uber — Schemaless evolved into Docstore, in production (C48). Remaining: Twitter/X post-2022 state (no first-party account found) |
| W4 | Netflix KV abstraction + Docstore trajectories | NEW R5: Netflix's abstraction layer over multiple DBs and Uber's Docstore are both live builds to watch — do they converge into products (P15 test)? |
| W5 | OpenAI write-scaling trajectory | NEW R6: C61 shows 800M users on unsharded PG with replicas — but OpenAI says the single writer can't scale writes, remaining write-heavy workloads are migrating to sharded systems, and cascading replication is being worked on with Azure. The honest limit of the counter-example; re-check on any OpenAI first-party update |

## Standing notes

- All gather-time URLs are preserved inline in `REGISTER.md` and `SURVEY.md`; never reconstruct.
- Vendor-published cases (eBay/MongoDB, Zurich/Neo4j, DoorDash/CRLabs, Cloudflare/ClickHouse, turbopuffer's own posts) are marketing-adjacent; state VP level when used.
- Survivorship bias: nobody blogs "we migrated and it was fine"; Stack Overflow / 37signals matter as the silent majority done well.
