# InvoiceFlow NEAR — Final Technical Research & Architecture Freeze

**Phase 0.5 · 2026-10-29 · Pacific/Auckland (NZDT, UTC+13)**\
**Pre-launch technical research only. CORE_IMPLEMENTATION_STARTED=NO.**

This is the evidence/decision record, not a second architecture. [ARCHITECTURE.md](ARCHITECTURE.md) is normative for models, states and proof format; [TEST_PLAN.md](TEST_PLAN.md) owns acceptance; [DELIVERY_PLAN.md](DELIVERY_PLAN.md) owns dates. Existing direction and repository structure are retained.

## 1. Executive Decision

**CONDITIONAL_GO to freeze architecture and start P0 after 2026-11-01 13:00 NZDT.** P0 and narrow P1 scope are FROZEN; actual P1 settlement feasibility is PARTIAL. No app, adapter, reconciler, contract, UI, deployment or transaction was implemented/executed.

- P0: real NEAR Testnet native Transfer -> FINAL receipt verification -> atomic PostgreSQL reconciliation -> immutable audit JSON -> reproducible tests/runnable demo.
- P1: one **Base mainnet USDC -> merchant NEAR mainnet USDC wallet** route through 1Click EXACT_OUTPUT, source/destination checks, independent verifier and clear UX.
- No production Intents Testnet per [official FAQ](https://docs.near-intents.org/resources/faqs). Native Testnet payment is a single-chain fallback. Official testing guidance is small mainnet swaps on separate dev/test accounts; this phase does not authorize one.
- **Nov 8 18:00 NZDT**: proceed with P1 only if actual authorized settlement and reproducible verification exist; otherwise protect P0. No mainnet spend authorization was requested or used.
- Freeze semantics/constraints, not an unverified SDK patch or production-readiness claim. No new contract or general multi-chain abstraction.

## 2. Verified Findings

**CONFIRMED** = explicit official documentation; **VERIFIED_BY_API** = limited actual observation; **ASSUMED** = engineering policy/inference; **UNKNOWN** = unproven/ambiguous. PASS describes the named probe only, never product acceptance.

| ID | Actual finding | Evidence level | Limit |
|---|---|---|---|
| V01 | FAQ/quickstart: no Intents testnet; separate mainnet dev/test accounts/small swaps recommended | CONFIRMED | No actual swap ran |
| V02 | Token GET: HTTP 200, **209 entries**, Base/NEAR USDC, 02:14:00 UTC / 15:14:00 NZDT | VERIFIED_BY_API | Oct 28's 208 entries remains historical |
| V03 | Anonymous 100-USDC EXACT_OUTPUT dry quote: HTTP 201, 02:14:34 UTC; amountOut=100000000, amountIn=101015943, minAmountIn=100005783 | VERIFIED_BY_API | No address or returned execution deadline/inactive time |
| V04 | Anonymous 1-USDC dry quote: HTTP 201, 02:14:35 UTC; amountOut=1000000, amountIn=1010179, minAmountIn=1000077 | VERIFIED_BY_API | Not proof of executable real minimum |
| V05 | Output=1 atomic USDC unit: HTTP 400, 02:14:36 UTC, generic quote failure | VERIFIED_BY_API | Exact cause/minimum UNKNOWN |
| V06 | Two testnet providers: testnet, not syncing, nearcore 2.14.0-rc.5; tx/tx_status/EXPERIMENTAL_tx_status all FINAL success for one public tx | VERIFIED_BY_API | Not InvoiceFlow/native payment |
| V07 | tx returned 1 receipt body, tx_status/alias 2; all had 2 receipt outcomes | VERIFIED_BY_API | List counts do not prove all required payment semantics |
| V08 | Base RPC: chain ID 0x2105=8453, non-null finalized block; NEAR RPC: mainnet, 2.14.0, not syncing | VERIFIED_BY_API | No transfer checked |
| V09 | NEAR USDC ft_metadata: USDC/6 decimals; Base/NEAR contracts match Circle registry | VERIFIED_BY_API + CONFIRMED | Metadata name alone is not token authenticity |
| V10 | EXACT_OUTPUT input-side buffer, status hash arrays, receipt/finality distinctions documented | CONFIRMED | Actual non-dry delivery schema not exercised |
| V11 | Default Prisma docs moved to 8.x; supported versioned 7.x docs remain | CONFIRMED | Pin 7.x exact versions after opening; no packages installed now |

### Failures and discrepancies retained

1. Initial Python urllib GET tokens returned **403**; PowerShell Invoke-WebRequest succeeded without auth. Client/WAF differences are an availability risk.
2. [Quote reference](https://docs.near-intents.org/api-reference/oneclick/request-a-swap-quote) renders partner auth required; [auth docs](https://docs.near-intents.org/integration/distribution-channels/1click-api/authentication) describe anonymous use. Anonymous dry quotes worked. Anonymous **live settlement/quota** remain unverified.
3. [Fee docs](https://docs.near-intents.org/resources/fees) publish extra 25 bps anonymous fees and authenticated defaults 20 bps / 1 bp for stablecoin/same-asset routes. Actual anonymous effective appFees contained **fee=1**, with near-parity minAmountIn. That field does not establish total cost. **Fee-policy/runtime discrepancy UNKNOWN**; do not invent its cause or promise a fixed charge.
4. Quickstart calls deadline quote expiry; request schema relates it to refund timing; response deadline describes address cutoff; timeWhenInactive indicates colder/slower processing. Dry omits execution fields. Distinct pricing TTL and actual address lifecycle remain UNKNOWN. Use conservative funding cutoff; no invented universal TTL.
5. Guessed directory/light-client URLs were inaccessible through web reader; specific API references and official nearcore proofs were used instead. No key finding depends on inaccessible pages.

Public sample: `28aDHmLoLrQYAEZecB9vDHb44fCEsLjnxEpUw2j2jatb`, signer `maker.tidepool-sandbox.testnet`, receiver `verifier.tidepool-sandbox.testnet`. Action is **mt_batch_transfer FunctionCall**, attached deposit `1`, with NEP-245 event. It proves neither native Transfer nor production Intents Testnet. No project payment/private-key inspection occurred.

## 3. Official Sources

Reviewed **2026-10-29**; full S01–S29 register and links in [RESEARCH_SOURCES.md](RESEARCH_SOURCES.md).

- [Intents FAQ](https://docs.near-intents.org/resources/faqs), [quickstart](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart/introduction), [swap types](https://docs.near-intents.org/integration/distribution-channels/1click-api/swap-types).
- [Quote API](https://docs.near-intents.org/api-reference/oneclick/request-a-swap-quote), [status API](https://docs.near-intents.org/api-reference/oneclick/check-swap-execution-status), [auth](https://docs.near-intents.org/integration/distribution-channels/1click-api/authentication), [fees](https://docs.near-intents.org/resources/fees), [quote signature](https://docs.near-intents.org/integration/distribution-channels/1click-api/verify-quote-signature).
- [NEAR RPC](https://docs.near.org/api/rpc/transactions), [lifecycle](https://docs.near.org/protocol/transactions/transaction-execution), [providers](https://docs.near.org/api/rpc/providers), [FT standard](https://docs.near.org/primitives/ft/standard), [NEP-141](https://github.com/near/NEPs/blob/master/neps/nep-0141.md).
- [Circle USDC contracts](https://developers.circle.com/stablecoins/usdc-contract-addresses), [Base finality/derivation](https://docs.base.org/specifications/base-protocol/consensus/derivation), [Ethereum RPC](https://ethereum.org/developers/docs/apis/json-rpc/).
- [PostgreSQL numeric](https://www.postgresql.org/docs/18/datatype-numeric.html), [Prisma 7 transactions](https://www.prisma.io/docs/orm/v7/prisma-client/queries/transactions), [release status](https://www.prisma.io/docs/orm/release-status), [JCS](https://www.rfc-editor.org/rfc/rfc8785), [nearcore proofs](https://near.github.io/nearcore/architecture/how/proofs.html).

No official scoring weights introduced. Older administrative/competition research was not re-audited in this technical phase.

## 4. Intents / RPC Capability Matrix

| Capability / A1 question | Evidence status | Freeze implication |
|---|---|---|
| Production Intents Testnet | CONFIRMED absent | Never call native Testnet cross-chain Intents |
| Real 1Click settlement | CONFIRMED production/mainnet route; project execution UNKNOWN | P1 conditional; no funds now |
| Official testing | CONFIRMED separate mainnet accounts/small swaps | Dry now, authorized real test later |
| dry=true quote | CONFIRMED + VERIFIED_BY_API | No address/initiation/payment |
| API key/partner needed? | Anonymous dry VERIFIED_BY_API; docs allow anonymous, schema disagrees | Generic EXACT_OUTPUT not documented partner-only; anonymous live execution UNKNOWN |
| Restricted modes | CONFIRMED ANY_INPUT partner-only | Excluded; no application for access needed now |
| Minimum amount | 100/1 USDC dry succeed, 1 atomic fails | Real/per-route minimum UNKNOWN; not a universal constant |
| Fees | Published policy + observed effective fields differ | Actual cost/refunds/gas separate from quote buffer; no fixed all-in promise |
| Assets | 209-entry live snapshot | Only one app-tested route; token presence != pair liquidity |
| NEAR wallet destination | Documented DESTINATION_CHAIN; dry recipient accepted | Real delivery/storage/graph UNKNOWN |
| Intents balance destination | Documented INTENTS | Different receipt claim; excluded |
| Foreign-chain recipient | Documented DESTINATION_CHAIN | Additional destination verifier; excluded initially |
| NEAR FINAL/receipt method | CONFIRMED + VERIFIED_BY_API on two providers | Require actual receipt analysis, version-aware alias |
| Base finalized head | Documented + VERIFIED_BY_API | Source capability, not payment proof |
| Consensus proof/light client | NOT RUN / not implemented | RPC-backed verification only |

### A2. EXACT_OUTPUT field contract

100 USDC uses `amount="100000000"` only for the explicitly selected six-decimal destination token. No fiat floats.

| Field | Unit / business meaning | Trusted check |
|---|---|---|
| originAsset | Supported source assetId | Server allowlist + actual source contract |
| destinationAsset | Settlement assetId | Frozen invoice + issuer registry |
| recipient / recipientType | Address; wallet-vs-Intents mode | Merchant snapshot; accepted address != credit |
| refundTo / refundType | Refund address/mode | Payer-controlled in real flow; research placeholders never fundable |
| amount | Destination base-unit string | Exact invoice output |
| quote.amountIn | Source amount to fund with slippage buffer | Estimate, not realized cost |
| quote.minAmountIn | Quoted minimum source input | Not guaranteed future execution |
| amountOut / minAmountOut | Destination base units | Independent final net credit |
| slippageTolerance | Basis points; 100=1% | Input buffer, never invoice underpayment tolerance |
| deadline | ISO UTC; differing request/response descriptions | No unconditional completion/refund promise |
| quote expiry | Conservative app cutoff from available times | Distinct price TTL UNKNOWN |
| timeWhenInactive | Execution-address lifecycle time | Not silently equated with deadline |
| correlationId | API tracing identifier | Bound to attempt, not blockchain proof |
| depositAddress / depositMemo | Real-quote funding tuple | Persist before exposing; absent dry; include memo when returned |
| status / updatedAt | Provider progress/time | Never direct PAID authority |
| originChainTxHashes | Source candidate array | Actual chain deposit/log validation |
| destinationChainTxHashes | Destination candidate array | Actual wallet credit, not arbitrary internal tx |
| nearTxHashes / intentHashes | Internal execution references | Not final wallet-delivery proof |
| refundFee / withdrawFee | Origin/destination base units | Withdraw fee documented inside amountOut; final check still required |

See [swap types](https://docs.near-intents.org/integration/distribution-channels/1click-api/swap-types), [quote schema](https://docs.near-intents.org/api-reference/oneclick/request-a-swap-quote) and [status schema](https://docs.near-intents.org/api-reference/oneclick/check-swap-execution-status). None of these fields alone proves settlement.

## 5. Recommended Payment Architecture

Retain modules/stack; [Architecture §2–3](ARCHITECTURE.md#2-payment-attribution) specifies ownership, finality and attribution.

| Destination | What counts as receipt | Two-week assessment |
|---|---|---|
| Merchant normal NEAR account | Native Transfer P0 or actual NEP-141 wallet credit P1 | **Recommended**; one destination chain, storage/receipts still verified |
| Merchant Intents balance | Internal ledger change in intents.near | Separate balance claim, not wallet receipt; excluded |
| Foreign-chain merchant address | Final successful token credit there | Adds destination finality/log/provider work; excluded initially |

Internal exchange and final withdrawal are **different events**. Provider SUCCESS describes delivery according to [workflow docs](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart/making-a-request), not independently proven credit. INTENTS mode ends at internal balance; DESTINATION_CHAIN mode requires actual wallet receipt verification. Prefer a normal merchant wallet without custom recipient msg.

Native P0 invoice explicitly requests testnet NEAR; it cannot mark a 100-USDC invoice paid through invented FX. Stablecoin settlement remains P1.

## 6. P0 / P1 / P2

| Priority | Frozen scope | Acceptance evidence, currently NOT RUN |
|---|---|---|
| P0不可牺牲 | Core CRUD/lifecycle, real Testnet native payment, FINAL, integer money, atomic deterministic reconciliation/unique claims, audit JSON, reproducible tests/runnable demo | New real testnet transfer + Postgres races/rollback + browser evidence |
| P1主要竞争优势 | One mainnet EXACT_OUTPUT USDC route, final wallet receipt, source evidence, independent verifier, clear payment/audit UX | Authorized actual settlement + fresh replay by Nov 8 |
| P2默认砍掉 | Generic AI, subscriptions/ERP, enterprise roles, AMM/bridge/token/unnecessary contract, general multi-chain, light client/anchored Merkle/signing infrastructure | Not planned |

P0 runs without P1 credentials/funds. If P1 OFF, optional dry preview clearly labelled DRY RUN; no real-funding CTA or completed-cross-chain claim. Scope frozen does not mean implementation complete.

## 7. Data Model

[Architecture §4](ARCHITECTURE.md#4-postgresql--prisma-model) is the sole normative model. Preserve Invoice, PaymentAttempt, CheckoutQuote, SettlementEvidence, Reconciliation and AuditReceipt; normalize ChainTransaction, EvidenceClaim and append-only AuditEvent. Merchant stays minimal.

JS BigInt + JSON digit strings + PostgreSQL numeric(78,0), not signed bigint; asset bounds; unique(network, txHash); canonical receipt/log identity; one claim across invoices/roles; one full reconciliation/original receipt per invoice; immutable issued snapshot. Prisma 7 Serializable transaction atomically claims/transitions/records audit. DB-specific CHECK/FK/partial indexes/grants need post-opening migrations. No executable schema/migration authored now.

## 8. State Machine

[Architecture §6](ARCHITECTURE.md#6-states-and-recovery) owns states: invoice, local attempt, provider swap and proof scopes are separate. SUCCESS -> PAID requires independent match **and DB commit**.

Expiry -> new quote/attempt while monitoring old funded attempts. Insufficient/incomplete -> unpaid, no app-driven top-up. Slippage/refund/pending/delayed delivery -> observed status and recoverable polling, no invented terminal outcome. Duplicates -> same result. Late/cancelled/overpaid/ambiguous -> review. Input-buffer refund does not negate merchant delivery. Later contradiction -> append correction/review; never erase ledger or release claims.

## 9. Settlement Proof Specification

[Architecture §7](ARCHITECTURE.md#7-settlement-proof-v1) defines the envelope/fields/canonicalization/verifier. Required: proofVersion, invoiceId/amount, settlement asset/amount, network/recipient, source/destination hashes, finality evidence, settlement time, reconciliationId, integrityHash and scoped verificationResult.

**Chosen v1:** JCS JSON + SHA-256, append-only audit/history digest and chain re-query. P0 export; P1 separate DB-independent verifier. Signed receipts, Merkle proofs, light clients and on-chain attestation deferred. They would not automatically prove invoice purpose or global exclusivity.

Digest means consistency against a retained/trusted digest; attacker can recompute it. Re-query chain facts; ignore embedded verdict. External invoiceBinding/claimUniqueness are UNVERIFIED without trusted business/operator assertions. Cross-chain causal link is provider-attested unless separately proven. Exclude PII, disclose linkable financial metadata and make sharing opt-in. Not a bank/legal payment receipt.

## 10. Security & Idempotency Threat Model

| Threat | Frozen control | Residual boundary |
|---|---|---|
| Fake hash/wrong network | Configured providers/checkpoint; actual tx/receipt | RPC trust remains |
| Third-party hash theft | Payer control, frozen snapshot, authenticated attempt/nonce, block window | Native chain has no invoice memo/purpose |
| Wrong recipient/token/amount | Immutable invoice; allowlist, exact integers, executor/receipt | Unsupported graphs fail closed |
| Replay/concurrent workers | Unique evidence claim/invoice reconciliation, transaction/CAS | Exclusivity in this DB only |
| Refund/child failure | FINAL and net credit | Bridge/issuer external risk |
| Forged/stale webhook/status | Wake-up hint; chain re-query; preserved history | Provider-attested cross-chain link |
| Crash/rollback | Claims/ledger/PAID/receipt one commit | DB integrity/admin trusted |
| Recomputed proof digest | Fresh scoped verification, no blanket badge | No issuer signature/global uniqueness v1 |
| PII/secrets/SSRF | Opt-in redacted proof, server secrets, fixed RPC URLs | Chain metadata inherently linkable |

## 11. Testing Matrix

[TEST_PLAN.md](TEST_PLAN.md) provides expected result/test type for fake hash; wrong network/sender/recipient/token/amount; under/overpayment; replay/duplicate/concurrent reconciliation; timeout; incomplete receipts; refunds; DB rollback; plus auth/SSRF/proof tampering and P1 faults.

**Executed now:** token GET, three dry quotes (two success/one negative), two-provider status/FINAL reads, Base chain/finalized head, NEAR metadata view. **NOT RUN:** product unit/integration/E2E, new native transfer, real Intents settlement, proof verifier, DB race/rollback, CI/deploy/video. Test-design PASS is specification coverage, not executed acceptance.

## 12. Two-Week Delivery Plan

[DELIVERY_PLAN.md](DELIVERY_PLAN.md) supplies deliverables, acceptance, test evidence, GO/NO-GO and fallback at every stage:

- Nov 1–3 (start Nov 1 13:00): locked stack/schema/auth/invoice slice.
- Nov 4–5: real native payment + FINAL verification.
- Nov 6–7: atomic reconciliation/audit; bounded P1 feasibility only while P0 on track.
- Nov 8 18:00: actual cross-chain evidence GO/NO-GO.
- Nov 9–11: approved P1 checkout/verifier/UI, otherwise finish P0.
- Nov 12–13: E2E/security/fault fixes.
- Nov 14: authorized deployment, video, technical materials.
- Nov 15 before 12:59 NZDT: technical consistency check; freeze at deadline. User owns submission/administration.

### Existing conflicts and gaps resolved

| Original location/content | Conflict / gap | Resolution |
|---|---|---|
| GRAND_PRIZE_STRATEGY: Nov 1–4 baseline/Nov 5–8 Intents; DELIVERY_PLAN: Nov 4 payment/Nov 5–6 reconciliation | Dates differed from requested 1–3 / 4–5 / 6–7 rhythm | Single delivery plan; strategy links to it |
| DELIVERY_PLAN/TEST_PLAN required JSON/CSV; IMPLEMENTATION_PROMPT only JSON | CSV mandatory status inconsistent | JSON P0; CSV optional |
| README stablecoin target + native fallback | Missing explicit denomination boundary | Testnet NEAR invoice distinct from USDC |
| ARCHITECTURE atomic strings/BigInt without SQL capacity choice | SQL bigint overflows yoctoNEAR | numeric(78,0), strict integer parsing/bounds |
| ARCHITECTURE known hash+payer association | Ownership/historical hash attribution underspecified | Wallet control, nonce/session/checkpoint, exclusive active tuple, honest off-chain-binding limit |
| ARCHITECTURE minimum/exact amount | Under/overpayment ambiguous | Exact-only, under reject/over review |
| ARCHITECTURE optional digest/open verifier | Recomputed digest/global reuse boundary incomplete | Scoped result; no authenticity/consensus/global uniqueness claim |
| Strategy internal numerical rubric | Explicitly non-official but unnecessary numerical authority | Qualitative evidence checklist; no official weights invented |

These refinements preserve product direction and avoid a competing architecture.

## 13. Top 3 Technical Risks

1. **No production Intents Testnet; P1 actual settlement unproven.** Mainnet permission, fees, storage registration/provider stability can block it. Mitigation: dry now; bounded authorized spike after opening; sticky Nov 8 NO-GO; protect P0.
2. **Receipt/provider semantics can falsely suggest settlement.** Parent success, internal hash, missing signer/receipts, refunds and unclear fee/TTL are not final merchant credit. Mitigation: one route, exact net FT amount, FINAL graph, source finalized check, schema/signature validation; ambiguous -> unpaid/review.
3. **Invoice binding/uniqueness can be overstated.** No native memo, unsafe storage, races and recomputable JSON hash threaten correctness/claims. Mitigation: authenticated immutable attempts, exact-capacity money, atomic DB constraints, adversarial tests and scoped proof. Global attribution/uniqueness remains out of scope.

No unavoidable fatal flaw found in narrowed design. Earlier attribution/storage/proof-scope gaps would block release; design now addresses them, but implementation/tests must establish correctness. Untraceable actual P1 withdrawal remains P1 NO-GO, never justification for a false proof.

### Judge-facing competitive assessment

1. Stronger than ordinary hash-paste invoicing: exact merchant asset, separate upstream/destination evidence, atomic receivable credit, replay/exception handling and portable independent verification.
2. Fastest 90–120s trust moment: download proof -> separate verifier with invoice DB unavailable -> actual destination contract/payee/amount/FINAL -> reject reuse on invoice B. Show source/quote links. Edit waits transparently, retain real IDs/timeline.
3. Source/tests can substantiate money precision, receipt predicates, SQL constraints/concurrent rollback, canonicalization and independent RPC queries.
4. Weak extras: generic AI, many untested chains, fancy proof badge, unanchored Merkle tree, unnecessary contract/dashboard without real evidence.
5. P0 alone is a credible correctness MVP with weaker prize differentiation. Full P1 is materially stronger. No defensible win probability or official scoring weights are asserted.

## 14. Fallback Decision

If no reproducible actual P1 settlement by **Nov 8 18:00 NZDT**, disable real cross-chain CTA/claims. Ship real native NEAR Testnet invoices, FINAL verification, atomic reconciliation and honest audit proof/export/replay; optional DRY RUN quote preview. Never substitute native Testnet, mock transfer, custom bridge or Intents balance for mainnet cross-chain settlement.

Circle also lists a distinct NEAR Testnet USDC. A later bounded single-chain FT test is possible, but not required P0 or automatically authorized scope, and **not Intents Testnet**. Original native fallback remains recommended.

## 15. Architecture Freeze Decision

**ARCHITECTURE_FREEZE=CONDITIONAL_GO. P0_SCOPE=FROZEN. P1_SCOPE=FROZEN (conditional delivery).** Conditions: actual implementation evidence, chosen wallet ownership support, receipt attribution and authorized mainnet gate. No early code or architecture rewrite authorized.

First formal task **after Nov 1 13:00 NZDT**: lock compatible Next.js/TypeScript/Node + Prisma 7/PostgreSQL toolchain; implement authenticated invoice slice with integer schema/constraints and immutable issued snapshot; pass creation/ownership/money round-trip integration tests and build. Then real payment verification, not cross-chain UI first.

Remaining UNKNOWN: actual settlement/refund/latency and status field completeness; anonymous fee policy; real pricing/address cutoffs; merchant registration; withdrawal signer discovery/net receipt graph; chosen wallet support; exact opening-day patches. Dry quotes cannot close them. No signing, partner applications, real tx, deployment or submission attempted.

```text
PHASE=PRELAUNCH_FINAL_RESEARCH
OFFICIAL_DOCS_REVIEW=PASS
INTENTS_CAPABILITY=PARTIAL
RPC_FINALITY_RESEARCH=PASS
CROSS_CHAIN_TESTNET_SUPPORT=NO_OFFICIAL_INTENTS_TESTNET
PAYMENT_ARCHITECTURE=P0_NEAR_TESTNET_NATIVE__P1_BASE_USDC_TO_NEAR_USDC_MAINNET_CONDITIONAL
RECONCILIATION_DESIGN=PASS
SETTLEMENT_PROOF_DESIGN=PASS
P0_SCOPE=FROZEN
P1_SCOPE=FROZEN
ARCHITECTURE_FREEZE=CONDITIONAL_GO
CORE_IMPLEMENTATION_STARTED=NO
```

Design PASS means specified/reviewed, not executed product acceptance. Product validation remains NOT RUN.

## Appendix A. Reproducible read-only evidence

Observed **2026-10-29 02:14–02:17 UTC / 15:14–15:17 NZDT** with PowerShell Invoke-WebRequest, no Authorization/X-API-Key. Initial urllib failure followed by successful public GET. RPC used one JSON object per POST. No send_tx, broadcast, deposit-submit, non-dry quote or wallet-signature request occurred.

### Tokens

GET `https://1click.chaindefuser.com/v0/tokens`: HTTP 200, 209 entries. Selected identities (both decimals=6):

- Base: `nep141:base-0x833589fcd6edb6e08f4c7c32d4f71b54bda02913.omft.near`.
- NEAR: `nep141:17208628f84f5d6ad33f0da3bbbeb27ffcb398eac501a31bd6ad2011e36133a1`.

### 100-USDC dry quote

POST `https://1click.chaindefuser.com/v0/quote`, Content-Type application/json, anonymous. **Recipient `near` and the public documentation refund address are research placeholders, not user-controlled funding instructions. NEVER fund this request.** To reproduce later, change deadline to future UTC; prices/results may differ. No fixed-rate guarantee.

```json
{
  "dry": true,
  "swapType": "EXACT_OUTPUT",
  "slippageTolerance": 100,
  "originAsset": "nep141:base-0x833589fcd6edb6e08f4c7c32d4f71b54bda02913.omft.near",
  "depositType": "ORIGIN_CHAIN",
  "destinationAsset": "nep141:17208628f84f5d6ad33f0da3bbbeb27ffcb398eac501a31bd6ad2011e36133a1",
  "amount": "100000000",
  "recipient": "near",
  "recipientType": "DESTINATION_CHAIN",
  "refundTo": "0x2527D02599Ba641c19FEa793cD0F167589a0f10D",
  "refundType": "ORIGIN_CHAIN",
  "deadline": "2026-10-29T02:34:34.719Z"
}
```

Actual HTTP **201** response, not a simulated fixture:

```json
{
  "quote": {
    "amountIn": "101015943",
    "amountInFormatted": "101.015943",
    "amountInUsd": "100.960990327008",
    "minAmountIn": "100005783",
    "amountOut": "100000000",
    "amountOutFormatted": "100.0",
    "amountOutUsd": "99.945600000000",
    "minAmountOut": "100000000",
    "timeEstimate": 35,
    "refundFee": "2400",
    "withdrawFee": "0"
  },
  "quoteRequest": {
    "dry": true,
    "depositMode": "SIMPLE",
    "swapType": "EXACT_OUTPUT",
    "slippageTolerance": 100,
    "originAsset": "nep141:base-0x833589fcd6edb6e08f4c7c32d4f71b54bda02913.omft.near",
    "depositType": "ORIGIN_CHAIN",
    "destinationAsset": "nep141:17208628f84f5d6ad33f0da3bbbeb27ffcb398eac501a31bd6ad2011e36133a1",
    "amount": "100000000",
    "refundTo": "0x2527D02599Ba641c19FEa793cD0F167589a0f10D",
    "refundType": "ORIGIN_CHAIN",
    "recipient": "near",
    "recipientType": "DESTINATION_CHAIN",
    "deadline": "2026-10-29T02:34:34.719Z",
    "confidentiality": "public",
    "quoteWaitingTimeMs": 0,
    "insured": false,
    "appFees": [
      {
        "recipient": "5880ad2b362620fadf759cbceb1cd5737ce8c6ed7fb8e9942881e6731f9247dd",
        "fee": 1
      }
    ]
  },
  "signature": "ed25519:2xzEryNpXVMs3JKSo1ttA3BTRACixzx2HzgZxNzsSCJDwgPDAN1MDnzyXdBDw1ZA1Ta92cgx2JbqyJAfVt8krEbA",
  "timestamp": "2026-10-29T02:14:34.548Z",
  "correlationId": "17636827-e1a1-4172-8102-4576d54608a7"
}
```

Signature retained but **NOT cryptographically verified** this phase; SDK not installed. No depositAddress, returned deadline or timeWhenInactive in quote. Echoed request deadline does not establish executable address lifetime. A dry HTTP 201 does not mean a funded swap was created.

### Other quotes

| Output base units | HTTP | correlationId | Actual result |
|---|---|---|---|
| 1000000 | 201 | dedcb9c0-5973-4540-8fb9-4fce76c08838 | amountIn=1010179; minAmountIn=1000077; amountOut=minAmountOut=1000000; refundFee=2400; withdrawFee=0 |
| 1 | 400 | 08de0ba0-968c-49cc-8d9c-67978cddd56f | Generic quote failure; no inferred precise minimum |

Other request fields same as above; their requested deadlines were 02:34:35.572Z and 02:34:36.340Z. Their request times were 02:14:35.572Z / 02:14:36.340Z; all on 2026-10-29 UTC.

### NEAR testnet FINAL

Each endpoint `https://test.rpc.fastnear.com` and `https://archival-rpc.testnet.near.org`: status with params `[]`, then the following body with method tx, tx_status and EXPERIMENTAL_tx_status separately:

```json
{"jsonrpc":"2.0","id":"phase05-research","method":"tx_status","params":{"tx_hash":"28aDHmLoLrQYAEZecB9vDHb44fCEsLjnxEpUw2j2jatb","sender_account_id":"maker.tidepool-sandbox.testnet","wait_until":"FINAL"}}
```

All eight HTTP 200, no RPC error. Both status results testnet/2.14.0-rc.5/syncing=false; all six tx results FINAL/top-level SuccessValue and successful observed outcomes. This is the FunctionCall described in §2, **not native payment**. tx_status observations at 02:14:55.888Z and 02:15:05.289Z UTC.

Selected actual response identifiers (not full raw response):

- Transaction outcome block `uzUHgUQKWXmGJ1QWrR9bWNrYMYQ6n8GXecTn7qKKm72`.
- Execution receipt `BPvGr8HcFnSjzmtHvShW78yKdmZka9ZDGGAXKeP9oWnN`, executor verifier.tidepool-sandbox.testnet, block `u2J2e5MwLhEpdYsPZdHWCwMrGtT2BUa2A2y6jhLut3Y`.
- Subsequent receipt `6swuSfb9QMNeHfGtYT8vRFk8wdSdXaX6XdsPyqkwWNJf`, block `A4TRMgoM9bo8ynR8j4411CeXGa9eT5x7o14U7c5gFpZM`.
- [Public transaction](https://testnet.nearblocks.io/txns/28aDHmLoLrQYAEZecB9vDHb44fCEsLjnxEpUw2j2jatb) is a convenience link; observations above come from RPC, not explorer appearance.

### Route preflights

- Base `https://mainnet.base.org`: eth_chainId params `[]` -> `0x2105`; eth_getBlockByNumber params `["finalized",false]` -> number `0x32c25ad`, hash `0xe848b23c4ba4206210d277010a7280e8a555b87be033f92144d8f888e9db4d36`, at 02:17:01–02 UTC. No payment queried.
- NEAR `https://free.rpc.fastnear.com`: status -> mainnet/2.14.0/syncing=false. query params `{"request_type":"call_function","finality":"final","account_id":"17208628f84f5d6ad33f0da3bbbeb27ffcb398eac501a31bd6ad2011e36133a1","method_name":"ft_metadata","args_base64":"e30="}` -> ft-1.0.0/USDC/decimals=6, at 02:17:02–03 UTC. View call only, no signed transaction.

### Raw captures

Retained outside repo at `D:\Ollama\Temp\invoiceflow-phase05-_5gwff7_`; temporary local evidence, not app dependency/permanent publication guarantee. Requests/key responses are embedded above; RPC results explicitly summarized, not full fixtures.

| Capture | Bytes | SHA-256 |
|---|---:|---|
| tokens.txt | 52971 | `17040badf4a287dfd76d495bdffb8b974a6f0ede7b1d6444ed992dcfa940eafa` |
| quotes.json | 5782 | `69a9800aac827e698ddc2bdd4d6f9318bddf9bfc65d337e4c7a762317fbc637b` |
| rpc.json | 81056 | `7c994162f6d9e8b459d184c15b9df21a268ad0efe5a12be6b2c7c8e6eb538955` |
| route-reads.json | 64771 | `c4950a61aa0384baa4f81f5ca0d1fb93abce9e59301f1cbb31042065d2450643` |

Only Markdown belongs to this phase's repo diff. No core implementation files or application fixtures created.
