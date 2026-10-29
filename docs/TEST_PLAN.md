# Test and Acceptance Plan — Phase 0.5

**2026-10-29 · Pacific/Auckland.** Product tests below are **PLANNED / NOT RUN**. This phase ran only the read-only research probes in [research §2](FINAL_TECHNICAL_RESEARCH_2026-10-29.md#2-verified-findings). API availability is not product acceptance. [Architecture](ARCHITECTURE.md) owns predicates and state policies; no executable tests/core code are created before opening.

## Evidence classes

- `PASS`: an actual command/probe ran and its stated assertion held.
- `PARTIAL`: some required evidence exists, with missing scope named.
- `FAIL`: an executed assertion failed.
- `BLOCKED`: a required dependency/access is unavailable.
- `NOT RUN`: designed but not executed. Stubs/simulations are explicitly labelled.

Every post-opening run records command, commit SHA, UTC/NZDT time, environment, real/stubbed provider, assertion counts/result, and sanitized evidence location. Live payments additionally record network, tx/signer, receipt identity, block/finality, invoice/attempt IDs and DB counts. Never store wallet seeds or session/API secrets. Fixtures retain enough public receipt data to reproduce decisions.

## Core and adversarial matrix

All rows initially **NOT RUN**. U = unit; DB = integration against real PostgreSQL; P = provider contract/fixture; E = browser E2E; L = real chain. A stubbed network test is not L.

| ID | Scenario | Expected result / invariant | Type |
|---|---|---|---|
| T01 | Draft/create/read/edit/cancel/archive invoice | Authorized owner only; opaque public token; issued asset/amount/payee immutable; referenced records not hard-deleted | DB, E |
| T02 | Negative/zero, fraction over asset precision, exponential notation, NaN, uint overflow | Reject before DB coercion; no float/rounding path | U, DB |
| T03 | 1 NEAR (10^24) and u128 boundary persisted | Exact integer round-trip through BigInt/string/numeric(78,0); SQL bigint overflow avoided | U, DB |
| T04 | Real direct matching native testnet Transfer | FINAL + matched payment receipt, one PAID invoice/ledger/claim/original receipt | L, DB, E |
| T05 | Fake/malformed tx hash | Malformed rejected; unknown well-formed hash bounded retry then unverified; never PAID | U, P, E |
| T06 | Wrong network / endpoint redirects to mainnet | Chain ID/checkpoint mismatch; no fallback across networks | P, L read-only |
| T07 | Wrong sender / pasted third-party transaction | Claim session/snapshot mismatch; no credit or evidence reservation | U, DB, E |
| T08 | Forged wallet callback / reused auth nonce / wrong domain | Signature/key/session challenge rejected; no attempt ownership | U, DB, E |
| T09 | Wrong recipient / self-transfer | MISMATCH, unpaid | U, P |
| T10 | Wrong token or spoofed same-symbol log/executor | MISMATCH; native/wrapped/FT identities distinct | U, P |
| T11 | Wrong amount / underpayment | Exact-match failure; no partial credit/aggregation | U, P, E |
| T12 | Overpayment | EXCEPTION_REVIEW; no automatic PAID/refund; retained evidence | U, DB |
| T13 | Replay same destination transfer for invoice B | Exactly one committed EvidenceClaim; B unpaid/CONSUMED | DB, E |
| T14 | Duplicate reconciliation on same invoice/evidence | Same reconciliation/receipt IDs; no new paid event | DB |
| T15 | 20 then 100 concurrent workers, separate DB connections | One reconciliation, one receipt, one destination claim; P1 also one source claim; losers replay/conflict safely | DB |
| T16 | Two different valid payments race for same invoice | One accepted ledger; second EXCEPTION_REVIEW, never silently discarded/reused | DB |
| T17 | RPC timeout / 429 / 5xx / shard missing / old history pruned | Retry/backoff/archival provider, unpaid; resume without losing attempt | P, DB |
| T18 | Included/EXECUTED_OPTIMISTIC/INCLUDED_FINAL/EXECUTED only | No MATCH_FINAL, even when top-level status looks successful | U, P |
| T19 | Incomplete receipt list/dependencies, missing child, parent success+child failure | Await required data or reject failure; never infer complete net credit from list length | U, P |
| T20 | Native multi-action/delegate/FunctionCall with deposit | Unsupported in P0; attached gas deposit not mistaken for invoice transfer | U, P |
| T21 | Refund after initial processing / FT resolve returns all/part | Before commit: unpaid unless final exact net credit independently matches; after commit: append discrepancy, preserve claims/history | U, P, DB |
| T22 | Forced DB failure after claim/paid update/before receipt insert | Atomic rollback: no new claim/reconciliation/receipt/PAID; safe retry | DB with fault injection |
| T23 | Crash before commit vs after commit/before HTTP response | Before: rollback; after: retry returns same receipt | DB |
| T24 | Cancellation/expiry and reconcile concurrently | Row/conditional state guards choose one valid outcome; late transfer review, not silent credit | DB |
| T25 | Historical payment before attempt / duplicate active native tuple | Reject stale evidence / refuse ambiguous second active attempt | U, DB |
| T26 | Refresh or lost wallet callback | Authenticated hash recovery resumes pending attempt, not an extra payment | E, DB |
| T27 | Runtime UPDATE/DELETE of audit/claims/evidence | DB permissions/trigger reject; correction append allowed; migration/admin trust disclosed | DB |
| T28 | Provider mismatch on hash/block/amount/finality | Block reconciliation; preserve conflicting observations | P, DB |
| T29 | Merchant A accesses B; CSRF/session misuse | Authorization denied; public fields minimal | DB, E |
| T30 | Arbitrary URL in proof, oversized JSON/logs, injection | No SSRF; strict schema/size bounds; parameterized SQL; safe errors | U, DB |
| T31 | Secret in client/tracked files, overly verbose logs | Release gate fails; redact; secrets never in proof | Build/CI inspection |
| T32 | SQLite or mocked DB used for concurrency acceptance | Insufficient evidence; real Postgres test remains NOT RUN | Acceptance review |

## Conditional 1Click matrix

All rows are **NOT RUN as product tests**; the separate research quote is not I01 integration acceptance.

| ID | Scenario | Expected result | Type |
|---|---|---|---|
| I01 | Live token list + EXACT_OUTPUT dry quote | DRY RUN label; no deposit address/funding/PAID | P, E, live read-only |
| I02 | Authorized Base USDC -> NEAR USDC real payment | Quote bound first; finalized source deposit; final exact destination wallet credit; source/destination claimed once | L mainnet, DB, E |
| I03 | Invoice 100 USDC field semantics | Request amount 100000000; source units 6 decimals; amountIn buffer distinct from final cost; no USD/USDC equivalence promise | U, P |
| I04 | Expired/changed quote | Funding disabled/new attempt; preserve old monitoring; server uses conservative cutoff | U, DB, E |
| I05 | INCOMPLETE_DEPOSIT / low balance / slippage exceeded | No credit; distinguish wallet funding error from provider processing/refund | P, E |
| I06 | Swap PROCESSING / destination delay | Persist retry state; no invented settlement SLA or timeout-to-PAID | P, DB, E |
| I07 | SUCCESS but missing/mismatched destination evidence | No PAID; API assertion displayed separately | P, DB, E |
| I08 | FAILED/REFUNDED / refund dust/fee | No credit; observed net refund and amount distinguished from assumed full return | P, E |
| I09 | Input buffer refund while merchant got exact output | Invoice remains settled; source refund is not merchant reversal | U, P |
| I10 | Repeated/out-of-order status or webhook | CAS state updates; re-fetch; one ledger result; terminal contradiction audited | P, DB |
| I11 | Quote signature invalid / echoed fields changed | No funding instruction or reconciliation; API signature != chain proof | P |
| I12 | NEAR destination signer missing; tx_status unsupported/incomplete | Public lookup then RPC confirmation, alias only on method-not-found; unresolved -> UNVERIFIED | P, live read-only |
| I13 | FT recipient not registered, nested callback refund | Preflight warns/blocks route; full receipt net credit checked | P, L only when authorized |
| I14 | Base latest/safe but not finalized | Source proof pending, despite NEAR destination success; full P1 claim not passed | P |
| I15 | Same source deposit reused with another quote/destination | EvidenceClaim source uniqueness rejects second settlement | DB |
| I16 | Batched source/split delivery/memo route/unknown schema | Unsupported route fails closed; never advertise support by token-list presence alone | U, P |
| I17 | Fee docs and live appFees differ | Retain effective quote; no hardcoded total/fee guarantee; explain before live funding | P, E |

## Portable proof tests

| ID | Scenario | Expected result | Type |
|---|---|---|---|
| P01 | JSON reorder/whitespace, string money, canonical known vectors | Same JCS digest; no unsafe-number precision loss | U |
| P02 | Amount/recipient changed without hash update | integrity MISMATCH; reject | U |
| P03 | Attacker changes amount and recomputes hash | Digest may match; chain predicate fails | U, P |
| P04 | Attacker changes only invoice ID and recomputes hash | Transfer may still verify; invoiceBinding remains UNVERIFIED; never claim trusted invoice discharge | U, P |
| P05 | Duplicate JSON keys/unsupported versions/extra fields | Strict reject before canonicalization | U |
| P06 | Download receipt; run separate verifier process with app/DB unavailable | Fresh RPC check reproduces scoped destination result; no call to invoice PAID API | P, L read-only, E |
| P07 | Replay receipt / ask for global uniqueness | Issuer DB rejects reuse; external claimUniqueness UNVERIFIED, never false global PASS | DB, E |
| P08 | Missing source/link/archive/provider, forged embedded verdict | Relevant scope UNVERIFIED; embedded verificationResult never authority | P |
| P09 | PII/public listing/default share | No names/email/line items; opt-in capability; disclose wallet/amount linkability | DB, E |
| P10 | Audit middle alteration/truncation by admin | Retained digest detects relevant alteration; no unsupported guarantee of full-history immutability | U, DB |

## Acceptance and evidence gates

**P0:** lint/typecheck/build, unit/provider tests, real PostgreSQL concurrency/rollback tests and deterministic browser tests; at least one **new real testnet native Transfer after opening** with FINAL receipts and DB counts. Chosen wallet ownership-signing must work. Audit export must include honest scope boundaries. Basic demo sharing/self-host instructions must be runnable. Wallet browser automation may use mocked wallets for deterministic CI, but that is labelled E2E simulation; record a separate real wallet/chain walkthrough.

**P1:** P0 remains green; I02 has one reproducible actual authorized route plus saved public evidence; re-query both legs; exact NEAR USDC credit; complete signer/receipt attribution; separate verifier demonstrates P06; API/link limitations visible. A prior recorded valid payment can support a fast video replay with its original timeline, not a new fabricated live swap. Live failure injections that cost funds are not required: use honest provider fixtures.

**CI and commands:** Define actual scripts after opening and report their real names/results, including PostgreSQL provisioning/migrations. Intended commands are lint, typecheck, unit, Postgres integration, Playwright and production build; none exists or ran in this docs-only repository. Never invent a CI URL or coverage percentage.

**Current status:** research probes PASS within their limited assertions; all product gates NOT RUN. No payment, app E2E, concurrency execution, deployment or demo was performed in Phase 0.5.
