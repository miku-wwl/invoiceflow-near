# InvoiceFlow NEAR

**Verifiable Cross-Chain Invoicing & Settlement**

> Status: **v0.1.1-prep — Phase 0.5 research complete; architecture CONDITIONAL_GO**, refreshed 2026-10-29 (Pacific/Auckland). P0 and conditional P1 scope are frozen; core implementation, product tests, deployment and actual payments are **NOT STARTED / NOT RUN**. Build window: **2026-11-01 13:00 to 2026-11-15 12:59 NZDT**.

InvoiceFlow NEAR is a proposed merchant-friendly product for issuing stablecoin invoices, accepting supported cross-chain payments through NEAR Intents, independently verifying settlement evidence, reconciling each payment exactly once, and exporting auditable receipts.

## Proposed demo (target, not completed)

1. Merchant issues a USDC invoice with a specified NEAR mainnet wallet (100 USDC is the design example; real demo amount requires separate authorization).
2. Payer views a 1Click `EXACT_OUTPUT` quote for a supported source network, including fees and quote expiry.
3. Payer signs a real wallet transaction; the application tracks deposit and settlement outcomes.
4. Backend verifies the destination transfer and atomically reconciles the invoice.
5. Auditor downloads the receipt and re-checks source/destination evidence with an independent verifier; business binding and external uniqueness retain explicit trust limits.
6. A duplicate transaction cannot pay a second invoice.

**P0 / fallback:** Real NEAR Testnet native Transfer on a separately denominated testnet NEAR invoice, FINAL RPC/receipt verification, atomic idempotent reconciliation and audit JSON. It cannot pay a USDC invoice by an invented exchange rate. There is no official production Intents Testnet. Simulations/dry quotes are labelled. P1 real-settlement gate: **2026-11-08 18:00 NZDT**.

## Technical direction

- Next.js App Router + TypeScript + Tailwind CSS
- Prisma ORM **7.x** (exact supported versions pinned after opening) + PostgreSQL; BigInt arithmetic / JSON digit strings / numeric(78,0) money
- `near-api-js`, Wallet Selector, NEAR RPC
- Conditional 1Click `EXACT_OUTPUT`: one Base USDC -> NEAR USDC wallet route, subject to real source/destination evidence
- Vitest, Playwright, GitHub Actions; target deployment on Vercel or equivalent
- No custody of user wallet secrets; no custom bridge or token issuance

## Documents

| Document | Description |
|---|---|
| [Grand Prize Strategy](docs/GRAND_PRIZE_STRATEGY.md) | Competition research, positioning, prize structure and differentiation |
| [Architecture](docs/ARCHITECTURE.md) | Modules, data model, payment attribution and trust boundaries |
| [Delivery Plan](docs/DELIVERY_PLAN.md) | Two-week milestones and explicit stop-loss rules |
| [Test Plan](docs/TEST_PLAN.md) | Negative tests, concurrency and release gates |
| [Demo Script](docs/DEMO_SCRIPT.md) | Judge-replayable `do X -> see Y` narrative |
| [Implementation Prompt](docs/IMPLEMENTATION_PROMPT.md) | First post-opening Codex implementation cycle |
| [Research Sources](docs/RESEARCH_SOURCES.md) | Primary-source links and observed API checks |
| [Final Technical Research](docs/FINAL_TECHNICAL_RESEARCH_2026-10-29.md) | Oct 29 real dry quote/RPC evidence, open risks and architecture-freeze decision |

## Competition

Primary track: **Real-World Money** (`realworld`) at [NEAR HACKATHON](https://nearhackathon.com/). Development opens 2026-11-01 00:00 UTC and submission closes 2026-11-14 23:59 UTC. Re-read the [official live HACKATHON.md](https://nearhackathon.com/HACKATHON.md) before submission. Repository, demo post, wallet and submission data must be human-approved before publication or submission.

This repository is a **planning baseline**, not a claim of implementation or payment. Prize conversion, payout terms and multiple-award eligibility are not assumed to be confirmed.
