# Research Sources & Evidence Levels

**Technical refresh: 2026-10-29 Pacific/Auckland.** Exact API observations and requests are in [final research §2 and appendix](FINAL_TECHNICAL_RESEARCH_2026-10-29.md). Historical strategy/administrative research dated Oct 28 is retained as historical; it was not re-audited as part of this technical phase.

## Classification

| Label | Meaning |
|---|---|
| CONFIRMED | Explicitly stated by the linked official technical source on review date |
| VERIFIED_BY_API | The stated response was actually observed by a bounded public/dry request; not proof of broader behavior |
| ASSUMED | Our engineering policy/inference, not a vendor guarantee |
| UNKNOWN | Evidence unavailable, ambiguous or not exercised |

PASS is the outcome of a particular check, not a substitute for those evidence levels. No real payment, product tests, CI/deployment or mainnet settlement ran. An Oct 28 quote was historical; use Oct 29 observations for the current snapshot.

## Official technical sources — reviewed 2026-10-29

| ID | Primary source | Supports / limitation |
|---|---|---|
| S01 | [NEAR Intents FAQ](https://docs.near-intents.org/resources/faqs) | No production testnet deployment; recommendation to use separate mainnet test accounts |
| S02 | [1Click Quickstart](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart/introduction) | Production flow and small-amount mainnet testing |
| S03 | [Swap Types](https://docs.near-intents.org/integration/distribution-channels/1click-api/swap-types) | EXACT_OUTPUT output denomination, input buffer/refund behavior |
| S04 | [Quote API reference](https://docs.near-intents.org/api-reference/oneclick/request-a-swap-quote) | dry flag, request/response fields, address types, deadlines and fee units; schema/auth/deadline discrepancies noted in report |
| S05 | [Status API reference](https://docs.near-intents.org/api-reference/oneclick/check-swap-execution-status) | Polling key, enum, origin/destination/NEAR hash arrays; schema, not live settlement evidence |
| S06 | [Making a Request](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart/making-a-request) | Deposit/status workflow and provider meaning of SUCCESS |
| S07 | [API Keys](https://docs.near-intents.org/integration/distribution-channels/1click-api/authentication) | Unauthenticated access/fee policy; partner JWT, X-API-Key/Bearer |
| S08 | [Fees](https://docs.near-intents.org/resources/fees) | Published fee schedules, partner-specific conditions; not a fixed all-in cost |
| S09 | [Supported tokens API](https://docs.near-intents.org/api-reference/oneclick/get-supported-tokens) | Token metadata discovery; a listed token is not a guaranteed executable route |
| S10 | [Quote signature verification](https://docs.near-intents.org/integration/distribution-channels/1click-api/verify-quote-signature) | API payload authenticity and official SDK helper; not chain finality |
| S11 | [Going Live](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart/going-live) | Delayed processing/refund troubleshooting; no guaranteed latency |
| S12 | [NEAR RPC Transactions](https://docs.near.org/api/rpc/transactions) | tx, tx_status/version alias, FINAL, errors and receipt availability |
| S13 | [Transaction lifecycle](https://docs.near.org/protocol/transactions/transaction-execution) | Execution vs finality and cross-contract failure/refunds |
| S14 | [Transaction anatomy](https://docs.near.org/protocol/transactions/transaction-anatomy) | Signer/receiver/actions; one receiver, possibly many actions |
| S15 | [RPC providers](https://docs.near.org/api/rpc/providers) | Configured mainnet/testnet/archival choices; version/rate/shard/history differences |
| S16 | [FT standard guide](https://docs.near.org/primitives/ft/standard) | ft_transfer, ft_transfer_call/resolution, storage and decimal metadata |
| S17 | [NEP-141 specification](https://github.com/near/NEPs/blob/master/neps/nep-0141.md) | FT semantics and transfer event shape/executor trust |
| S18 | [NEP-413](https://github.com/near/NEPs/blob/master/neps/nep-0413.md) | Off-chain wallet challenge signing; implementation/wallet support not tested here |
| S19 | [nearcore proofs](https://near.github.io/nearcore/architecture/how/proofs.html) | Inclusion/proof concepts; no light client implemented |
| S20 | [Circle USDC contracts](https://developers.circle.com/stablecoins/usdc-contract-addresses) | Mainnet Base and NEAR USDC contract identities; also lists a distinct NEAR Testnet USDC |
| S21 | [Base derivation specification](https://docs.base.org/specifications/base-protocol/consensus/derivation) | L1-derived finalized L2 head; no sequencer-inclusion equals finality claim |
| S22 | [Ethereum JSON-RPC](https://ethereum.org/developers/docs/apis/json-rpc/) | chain ID, receipt, block/log lookup and finalized tag semantics |
| S23 | [PostgreSQL 18 numeric types](https://www.postgresql.org/docs/18/datatype-numeric.html) | bigint range, exact numeric precision/scale and rounding behavior |
| S24 | [PostgreSQL 18 constraints](https://www.postgresql.org/docs/18/ddl-constraints.html) | Unique, NOT NULL, CHECK and FK design; NULL caveats |
| S25 | [PostgreSQL 18 locking](https://www.postgresql.org/docs/18/explicit-locking.html) | Row-lock concurrency semantics |
| S26 | [Prisma release status](https://www.prisma.io/docs/orm/release-status) | Default 8.x transition and supported 7.x alternative; recheck exact versions on opening |
| S27 | [Prisma 7 transactions](https://www.prisma.io/docs/orm/v7/prisma-client/queries/transactions) | Interactive atomic transactions, isolation/conflict handling, idempotency |
| S28 | [Prisma 7 schema reference](https://www.prisma.io/docs/orm/v7/reference/prisma-schema-reference) | Decimal mapping and compound keys; migrations still needed for DB-specific rules |
| S29 | [RFC 8785 JCS](https://www.rfc-editor.org/rfc/rfc8785) | Deterministic JSON canonicalization; large integers use strings |

Canonical documentation indexes: [Intents llms.txt](https://docs.near-intents.org/llms.txt), [NEAR llms.txt](https://docs.near.org/llms.txt). References that redirect use final canonical targets above where known. The general 1Click directory and guessed light-client page were inaccessible to the web tool; the actual API references and nearcore proof specification were used instead. No unresolved key finding relies on those inaccessible pages.

## Current read-only endpoints (observed, not service guarantees)

- `GET https://1click.chaindefuser.com/v0/tokens`: HTTP 200, 209 entries through PowerShell. Initial Python urllib request returned 403; this client-dependent failure is retained in the report.
- `POST https://1click.chaindefuser.com/v0/quote`: **dry=true only**, anonymous EXACT_OUTPUT for 100 and 1 NEAR-chain USDC from Base USDC, HTTP 201. One atomic output unit rejected (HTTP 400); no universal minimum inferred.
- `https://test.rpc.fastnear.com`, `https://archival-rpc.testnet.near.org`: status + tx/tx_status/EXPERIMENTAL_tx_status on one existing public transaction, all HTTP 200/FINAL. No payment sent.
- `https://mainnet.base.org`: chain ID and finalized block read; no transfer queried/submitted.
- `https://free.rpc.fastnear.com`: mainnet status and NEAR USDC ft_metadata view call; no signed transaction.

## Historical sources retained from Oct 28

These inform strategy only and are not new technical API observations:

- Organizer: [website](https://nearhackathon.com/), [HACKATHON.md](https://nearhackathon.com/HACKATHON.md), [Foundation announcement](https://www.near.org/blog/near-hackathon-build-beyond-boundaries). No scoring weights are asserted as official.
- Integrations: [Wallet Selector](https://github.com/near/wallet-selector), [official examples](https://github.com/near-examples/near-intents-examples), [NEAR agent quote/status notes](https://github.com/near/agent-skills/tree/main/skills/near-intents/rules). Primary API docs above govern this freeze.
- Alternatives: [Cross-Pay](https://intents.near.org/use-cases/cross-pay), [Request Finance](https://docs.request.finance/invoices), [Copperx](https://copperx.io/invoicing), [Payclave](https://www.payclave.com/guides/onchain-payment-verification).
- Historical winners: [MetaBUIDL](https://near.org/blog/metabuidl-hackathon-winners), [MetaBUILD 2](https://www.near.org/blog/metabuild-2-hackathon-winners), [NEARCON](https://www.near.org/blog/meet-the-2023-nearcon-irl-hackathon-winners), [Useful Agents](https://www.near.org/blog/building-useful-agents-sf-hackathon-shows-the-power-of-practical-ai).

No historical observation constitutes InvoiceFlow implementation or a production settlement test.
