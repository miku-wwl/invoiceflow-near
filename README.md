# InvoiceFlow NEAR

**Verifiable Cross-Chain Invoicing & Settlement**

> Status: **v0.1.0-prep — pre-launch documentation only**, prepared 2026-10-28 (Pacific/Auckland). No contest core code, deployment, or payment has been implemented yet. The official build window is **2026-11-01 13:00 to 2026-11-15 12:59 NZDT**.

InvoiceFlow NEAR is a proposed merchant-friendly product for issuing stablecoin invoices, accepting supported cross-chain payments through NEAR Intents, independently verifying settlement evidence, reconciling each payment exactly once, and exporting auditable receipts.

## Proposed demo (target, not completed)

1. Merchant issues a 100 USDC invoice with a specified settlement account.
2. Payer views a 1Click `EXACT_OUTPUT` quote for a supported source network, including fees and quote expiry.
3. Payer signs a real wallet transaction; the application tracks deposit and settlement outcomes.
4. Backend verifies the destination transfer and atomically reconciles the invoice.
5. Auditor downloads the receipt and re-checks source/destination evidence with an open verifier.
6. A duplicate transaction cannot pay a second invoice.

**Fallback:** A real NEAR testnet native transfer + finalized RPC verification + idempotent reconciliation, clearly differentiated from mainnet NEAR Intents settlement. Mock data will be labelled as simulated.

## Technical direction

- Next.js App Router + TypeScript + Tailwind CSS
- Prisma + PostgreSQL
- `near-api-js`, Wallet Selector, NEAR RPC
- Optional (conditional) NEAR Intents 1Click `EXACT_OUTPUT` integration
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

## Competition

Primary track: **Real-World Money** (`realworld`) at [NEAR HACKATHON](https://nearhackathon.com/). Development opens 2026-11-01 00:00 UTC and submission closes 2026-11-14 23:59 UTC. Re-read the [official live HACKATHON.md](https://nearhackathon.com/HACKATHON.md) before submission. Repository, demo post, wallet and submission data must be human-approved before publication or submission.

This repository is a **planning baseline**, not a claim of implementation or payment. Prize conversion, payout terms and multiple-award eligibility are not assumed to be confirmed.
