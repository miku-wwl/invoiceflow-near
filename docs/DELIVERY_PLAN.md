# Delivery Plan — frozen Phase 0.5 baseline

**2026-10-29 · Pacific/Auckland (NZDT, UTC+13).** Milestones below are planned; no implementation/test/deployment is implied. [Architecture](ARCHITECTURE.md) owns scope, [Test Plan](TEST_PLAN.md) owns assertions.

## Hard dates and stop rules

- **2026-11-01 13:00 NZDT (00:00 UTC):** first permitted contest core implementation.
- **2026-11-08 18:00 NZDT:** internal P1 GO/NO-GO deadline; this time is an engineering decision, not an organizer rule.
- **2026-11-14 18:00 NZDT:** internal technical release/materials-ready target.
- **2026-11-15 12:59 NZDT (2026-11-14 23:59 UTC):** supplied absolute deadline. Freeze source/deployment/submission materials by then; no further mutation after deadline.

## Stage gates

| Dates | Deliverables | Acceptance criteria | Test evidence to retain | GO / NO-GO | Fallback |
|---|---|---|---|---|---|
| 10/29–10/31 | Official research, dry quote/RPC evidence, frozen docs | Markdown-only diff; no core code/payment/deploy | Research request/response summaries and doc checks | GO for post-opening P0; P1 conditional | No early implementation; unresolved real-settlement claims remain NOT RUN |
| 11/1 13:00–11/3 | Locked stack (Prisma 7), Postgres migrations, merchant auth, invoice CRUD vertical slice | Exact integer money, immutable issue snapshot, owner checks, public capability | Fresh install/migration/build; T01–T03/T29; browser invoice flow | GO only with working slice and correct DB schema | Cut cosmetics/CSV; fix foundations before payment |
| 11/4–11/5 | Payer ownership/session; native Testnet wallet payment; FINAL verifier and recoverable attempt | One new real matching Transfer; wrong sender/payee rejected; evidence traced to receipt | T04–T10, T17–T20, T26; tx/signer/headers/outcomes | GO when real P0 funds evidence is independently checked | Select supported wallet/provider; keep failures visible; never replace real transfer with mock PASS |
| 11/6–11/7 | Idempotent reconciler, atomic claims/ledger/PAID, audit JSON/hash chain; bounded P1 evidence spike after P0 slice | 20/100 real-Postgres races produce one result; rollback and replay safe; proof scopes honest | T13–T16/T21–T25/T27; P01–P05; DB counts/commands | GO if P0 end-to-end is solid; P1 work cannot displace fixes | Stop optional cross-chain spike if P0 regresses |
| 11/8 by 18:00 | P1 decision packet: one actual authorized Base USDC -> NEAR USDC settlement and independent replay | FINAL destination exact credit; finalized source; signer/receipts resolved; repeatable verification and fee explanation | I02/I07/I11–I15; source/destination hashes, saved proof, fresh replay; P06 | **GO only if all evidence exists.** Dry quote/API SUCCESS is NO-GO for real-settlement claims | Freeze P1 OFF if missing/unauthorized/unstable; preserve labelled dry preview only; protect P0 |
| 11/9–11/11 | If GO: cross-chain checkout/recovery, independent verifier, payment/audit UI. If NO-GO: finish native verifier UX | No status-to-PAID shortcut; explicit route/network labels; retry/exception/replay demonstrable | Full relevant I/P matrix, browser journey, P0 regression suite | GO only while P0 green and narrow route reliable | Remove cross-chain payment CTA/claims; keep native P0 |
| 11/12–11/13 | E2E, security review, fault fixes, CI and release candidate | Fresh-machine setup, real Postgres concurrency/rollback, auth/SSRF/PII checks, independent replay | CI logs/commit, T/P matrix results, sanitized artifacts | NO-GO for unverified core invariants; fix blockers first | Cut optional UI/CSV/SDK work, never money/auth/finality checks |
| 11/14 | Deploy runnable demo (later authorization applies), record 90–120s video, technical README/materials | Exact network/route claims; real evidence links; app works from reviewer instructions | Deployment smoke, finality replay, video timeline/screenshot checks | Internal ready by 18:00; report any deployment authorization/access block | Supply runnable local/self-host demo and evidence; do not claim live URL exists |
| 11/15 before 12:59 | Final technical link/material consistency check and handoff | Correct commit, tested URL or clearly labelled local instructions, no secret, no unsupported claim | Final matrix and known limitations; read-only final checks | Ready for user-controlled submission; no automatic submission | User handles administrative/publication matters; freeze at deadline |

## Frozen P0 / P1 / P2

**P0 cannot be traded away:** core invoice lifecycle, real NEAR Testnet native Transfer, FINAL evidence verification, deterministic reconciliation, unique claims/atomicity, immutable audit receipt, reproducible automated tests and runnable demo. Native payment invoices explicitly request testnet NEAR; stablecoin target is not falsely fulfilled by native value. JSON is required, CSV optional.

**P1 competitive target:** exactly one 1Click EXACT_OUTPUT stablecoin route, destination wallet delivery, source/destination public evidence, independent proof CLI/page, clear checkout/audit UI. Route acceptance requires a separately authorized actual mainnet test. Public chain queries and dry quotes do not spend money. Missing authorization by gate means P1 stays OFF; no requests about personal funding or event administration in this technical plan.

**P2 excluded:** generic AI agents, SaaS subscriptions, ERP/accounting suite, enterprise tenancy/roles, custom AMM/bridge, token issuance, unnecessary contract, fiat off-ramp, light client, anchored Merkle tree, signed audit-key infrastructure, general multi-network adapters. Merchant SDK/CSV only if demonstrably low-cost after all gates pass; not part of acceptance.

## Daily discipline and decision record

Implement one vertical slice -> run relevant checks -> retain real result -> fix -> update evidence/known limits. Use PASS/PARTIAL/FAIL/BLOCKED/NOT RUN accurately. Start bounded P1 feasibility while P0 work is on track so the Nov 8 gate is achievable; P1 has no permission to stall P0. A single real settlement plus repeatable **verification** is the minimum gate; a second payment is useful if separately authorized, not assumed.

P1 NO-GO is sticky for this submission unless a small, proven fix fits existing scope without reopening P0. Do not build a custom bridge or an Intents-balance verifier as a deadline rescue. P0 alone is an honest engineering MVP with weaker cross-chain prize differentiation; update video/README claims accordingly. Personal eligibility, funding, posts and submission administration remain user-owned and are not tasks here.
