# InvoiceFlow NEAR — Grand Prize Strategy

**v0.1 · 2026-10-28 · Pacific/Auckland**  
**Research and product strategy; not contest implementation.**

**Technical refresh 2026-10-29:** [Final research](FINAL_TECHNICAL_RESEARCH_2026-10-29.md) records live evidence; [Architecture](ARCHITECTURE.md) is the single frozen design and [Delivery Plan](DELIVERY_PLAN.md) owns dates. P0 is native Testnet; P1 is one conditional Base USDC -> NEAR USDC mainnet wallet route. No real payment/product implementation yet. Oct 28 observations below remain explicitly historical.

## Executive recommendation

Keep InvoiceFlow, but **do not build only an invoice CRUD page with an onchain transaction hash**. The intended product is **Verifiable Cross-Chain Invoicing & Settlement**: merchants request an exact stablecoin amount; customers pay on a supported source chain using NEAR Intents; the backend independently validates destination-chain settlement, reconciles exactly once and issues an independently re-checkable evidence bundle. The differentiator is the trustworthy *business-level settlement workflow*, not simply integration of the existing 1Click API.

### Positioning

> Create an invoice in stablecoins. Get paid across chains. Verify every settlement. Reconcile without ambiguity.

Primary track: **Real-World Money**. An all-event Grand Prize is a stretch goal, not an expected outcome. We do not add a generic AI Agent to fit other tracks.

## Official prize structure (newly verified)

The [NEAR Foundation announcement of 2026-10-19](https://www.near.org/blog/near-hackathon-build-beyond-boundaries) confirms the following USD-advertised amounts:

| Category | Each | Count | Total |
|---|---:|---:|---:|
| Grand Prize | $55,000 | 1 | $55,000 |
| Track 1st place | $10,000 | 13 | $130,000 |
| Track 2nd place | $3,000 | 13 | $39,000 |
| Track 3rd place | $2,000 | 13 | $26,000 |
| **Total** | | | **$250,000** |

**CONFIRMED:** One $10,000 first-place award exists for each of the 13 tracks; a separate $55,000 Grand Prize exists.  
**ASSUMED / UNCONFIRMED:** Whether a single project can collect more than one award; payout exchange-rate formula; payment schedule/lock-up mechanics; any other unpublished details. Track selection across multiple categories is permitted, but that does not prove stacked winnings. Do not state assumptions as official rules.

## What previous NEAR winners suggest

These are examples, not a confirmed 2026 rubric:

- [Roketo — MetaBUIDL Grand Prize](https://near.org/blog/metabuidl-hackathon-winners): simplified continuous, payroll-like payments through token streaming. Demonstrated a new financial primitive and easier user workflow.
- [Voog — MetaBUILD 2 first-place Grand Prize](https://www.near.org/blog/metabuild-2-hackathon-winners): visual contract generation and one-click deployment; strong abstraction of complexity.
- [ClickDapp — NEARCON Entrepreneurs winner](https://www.near.org/blog/meet-the-2023-nearcon-irl-hackathon-winners): accessible application-building experience for non-developers.
- [NEAR Useful Agents winners](https://www.near.org/blog/building-useful-agents-sf-hackathon-shows-the-power-of-practical-ai): concrete benchmarks and reproducible evaluation of usefulness.

**Inference:** An outstanding entry solves a real economic problem, uses NEAR technology for a material reason, makes its benefit visible quickly, and backs claims with actual execution and metrics. The 2026 FAQ names technical implementation, design, impact, originality and sustainability as *historical* judging considerations; no official weights have been published.

## Competitive alternatives and differentiation

| Existing offer | Proven overlap | Our narrow differentiator |
|---|---|---|
| [Request Finance](https://docs.request.finance/invoices) | Crypto invoices and payment handling | Open, vendor-independent evidence replay and explainable settlement reconciliation |
| [Copperx](https://copperx.io/invoicing) | Invoices, payment links, crypto checkout and conversion | Explicit origin/destination provenance + receivable state correctness |
| [NEAR Intents Cross-Pay / PingPay](https://intents.near.org/use-cases/cross-pay) | Cross-chain links; payer chooses asset and recipient gets preferred stablecoin | Business invoice identity, exception handling and audit-grade evidence |
| [Payclave](https://www.payclave.com/guides/onchain-payment-verification) | Non-custodial settlement, duplicate checks, merchant reconciliation | A portable open verifier spanning the cross-chain evidence and the invoice |

**Do not pretend** invoices, cross-chain payments, idempotency or onchain verification were invented here. Our potential novelty is the reproducible **composition** and openness of a cross-chain business settlement proof. This must be demonstrated, not marketed as a world-first.

## Real technical research already completed (2026-10-28)

1. A NEAR testnet RPC `status` query returned `chain_id: testnet` and synced node state.
2. A live testnet transaction (`28aDHmLoLrQYAEZecB9vDHb44fCEsLjnxEpUw2j2jatb`, signer `maker.tidepool-sandbox.testnet`) was queried with `tx`, `wait_until: FINAL`; RPC returned finalized success. **Not** an InvoiceFlow payment or a native Transfer proof.
3. A live `GET /v0/tokens` call returned **208 token entries at that time**, including Base USDC and NEAR USDC.
4. A read-only 1Click `EXACT_OUTPUT` **dry quote** for 1 NEAR-chain USDC paid from Base USDC succeeded, returning `amountOut=1000000`, `amountIn=1010156`, `minAmountIn=1000054` (six-decimal atomic units) and a `correlationId`. It is a time-specific demonstration quote, not a guaranteed rate or fee.
5. **No actual 1Click transfer occurred.** Dry quoting creates no deposit address. Official [NEAR Intents FAQ](https://docs.near-intents.org/resources/faqs) confirms no production Intents testnet deployment. Actual settlement is a mainnet feature requiring a separate authorized transaction.

Technical sources: [NEAR RPC Transactions](https://docs.near.org/api/rpc/transactions), [1Click Swap Types](https://docs.near-intents.org/integration/distribution-channels/1click-api/swap-types), [1Click API Status](https://github.com/near/agent-skills/blob/main/skills/near-intents/rules/api-status.md), [Official examples](https://github.com/near-examples/near-intents-examples).

## The three core capabilities

### 1. Exact-output cross-chain checkout

- Merchant names the *actual settlement asset, network, account and atomic-unit amount*.
- Payer chooses from the live supported token list, receives a transparent `EXACT_OUTPUT` quote including source amount, fees, minimum input, deadline and refund destination.
- A quote's correlation ID and deposit address are bound to **one invoice/attempt** before attribution.
- The solution must avoid false fiat pricing promises; a USD invoice can map to an explicitly selected stablecoin settlement amount, not an unchecked floating-point exchange estimate.

### 2. Deterministic reconciler

- Separate source deposit, intent progress, final destination transfer, invoice and ledger entry.
- Validate token contract, network, sender/recipient conditions, intended output amount, success/finality and invoice time policy.
- 1Click `SUCCESS` is one piece of evidence; inspect actual destination-chain movement independently wherever the API provides its hashes.
- Failed/refunded/incomplete/expired states must never turn into `PAID`.
- Unique transaction consumption and database atomicity prevent duplicate settlement, including after 20+ parallel and replayed events.

### 3. Replayable Proof of Settlement

- Export `invoiceId`, quote correlation ID, origin/destination hashes, network, contract IDs, amounts, settlement account, verifier version and decision reason.
- Provide `Verify` that queries actual blockchain evidence again without trusting the invoice database's `PAID` field.
- Optionally canonicalize and hash the receipt for tamper detection. A hash alone is **not** a trustless chain-finality proof; explain RPC and upstream oracle trust.
- Public proofs should not leak customer personal data.

## Golden 90–120 second judge narrative

1. Merchant creates a 100 USDC invoice.
2. Payer chooses Base USDC and sees the 1Click exact-output quote for merchant settlement on NEAR.
3. A real authorized wallet transaction occurs; the page tracks `PENDING_DEPOSIT -> PROCESSING -> SUCCESS`.
4. Backend verifies destination-chain payment and atomically marks the invoice paid once.
5. In a separate unauthenticated view, an auditor re-runs verification from a downloaded proof.
6. Attempted reuse of the same transaction for a second invoice is rejected with an explicit reason.

Any simulated negative example must say `SIMULATED TEST`. Any unexecuted mainnet path must say `DRY RUN` or `NOT VERIFIED`.

## Qualitative evidence checklist (not official weights)

| Dimension | Evidence to demonstrate |
|---|---|
| Technical correctness | Real finality/receipt checks, replay protection and rollback/concurrency tests |
| Originality | Open verifier with honest invoice/cross-chain trust boundaries |
| Real-world value | Merchant receives the exact chosen asset; receivable reconciles once |
| UX | Amounts, fees, waiting/recovery and audit results are clear |
| Sustainability | Runnable self-host instructions, reproducible verification and limited dependencies |

No official numerical weights or probability of winning are asserted. A JSON hash is not authenticity/consensus proof; external invoice binding and global uniqueness cannot be inferred from a chain transfer.

## Scope and stop-loss

**P0 floor:** invoice, real NEAR testnet native Transfer, `FINAL` RPC verification, exactly-once reconciliation, audit receipt, tested public demo. Native `Transfer` has no invoice memo: require authenticated payer control, frozen attempt/checkpoint, exact recipient/amount and invoice-bound claim; business purpose remains off-chain. Native fallback invoices request testnet NEAR, not USDC.
**Prize-focused target:** mainnet 1Click exact-output settlement *if transaction authorization and actual evidence are available*, terminal-status handling, destination-chain verification, open proof replayer.  
**Stretch:** merchant API / verified event webhook / reusable SDK.  
**Out:** AI gimmicks, ERP, custom bridges, token issuance, premature multi-network adapters and nonessential contracts.

If verified Intents settlement cannot work reliably by **2026-11-08 18:00 NZDT**, stop that extension and ship the stable testnet P0 honestly. One complete reliable flow beats a partially faked cross-chain flow.

## Two-week execution gates (NZDT)

[DELIVERY_PLAN.md](DELIVERY_PLAN.md) is the authoritative schedule: Nov 1–3 invoice slice (start Nov 1 13:00), Nov 4–5 real native payment, Nov 6–7 reconciliation/audit, Nov 8 18:00 P1 gate, Nov 9–11 approved cross-chain/verifier/UI work, Nov 12–13 E2E/security/fixes, Nov 14 deployment/video, final technical check before Nov 15 12:59. Every stage specifies evidence, GO/NO-GO and fallback. No technical artifact mutation after the deadline.

## Competitive assessment

Plain testnet invoice + RPC: **adequate engineering MVP, weak Grand Prize differentiation**.  
Real exact-output settlement + independent proof + idempotent accounting: **credible Real-World Money first-place candidate, though not guaranteed**.  
Open, reproducible business settlement primitive with real user value and polished evidence: **a more serious Grand Prize stretch**.

We optimize for actual execution, correctness, clarity and reproducibility, not a maximal feature count.
