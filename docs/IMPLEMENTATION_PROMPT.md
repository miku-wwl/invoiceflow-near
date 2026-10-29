# Codex implementation prompt — use ONLY after official opening

**Frozen Phase 0.5 · 2026-10-29.** This file is a plan, not permission to start early. Earliest core implementation: **2026-11-01 13:00 Pacific/Auckland**. At/after 2026-11-15 12:59 NZDT freeze source/deployment/materials; later work is read-only/private reporting unless the user explicitly changes scope outside the competition.

You are CTO, senior full-stack engineer and delivery lead for InvoiceFlow NEAR. Communicate in Chinese; code/technical docs may be English. Preserve the documentation-only baseline until opening. Do not convert design PASS into test PASS.

## Read first

1. README.md and docs/GRAND_PRIZE_STRATEGY.md.
2. docs/FINAL_TECHNICAL_RESEARCH_2026-10-29.md and docs/RESEARCH_SOURCES.md for observed findings/unknowns.
3. docs/ARCHITECTURE.md: the **single normative design**.
4. docs/TEST_PLAN.md and docs/DELIVERY_PLAN.md for acceptance/stage gates.
5. Current official SDK documentation, specifically Prisma **7.x** docs. Pin exact supported versions after opening, not unqualified latest (default Prisma docs now describe 8.x).

Do not start a second architecture. Change a frozen invariant only when evidence justifies it; document the actual impact and keep P0 intact. Event administration is user-owned; do not create personal/legal/funding/submission tasks.

## First task after opening: authenticated invoice vertical slice

1. Check current client time meets opening. Initialize Next.js App Router + TypeScript + Tailwind in existing repo; choose compatible supported Node/framework versions and lock them.
2. Pin Prisma 7 CLI/client/driver adapter and PostgreSQL; use versioned docs. Implement invoice/auth schema and migrations, exact integer storage `numeric(78,0)` via Prisma Decimal, BigInt computations and JSON digit strings. SQL signed bigint cannot store yoctoNEAR. Reject fractional/noncanonical/overflow inputs before DB coercion. Add asset bounds and DB constraints from architecture.
3. Implement merchant-only create/read/edit-draft/cancel/archive and unguessable public invoice view; immutable asset/network/recipient/amount/version after issue. No PII on chain/public proof. Native testnet invoice amount is NEAR, not an unchecked conversion of USDC.
4. Verify fresh migration/install, invoice ownership/validation/integer round-trip integration tests and production build. Report exact commands/results. Finish this slice before expanding into payments/UI extras.

## Next P0 slices (Nov 4–7)

1. Payer wallet control through expiring domain/network/invoice/attempt-bound challenge, signature/key/nonce checks; supported wallet only. Freeze attempt/checkpoint/expected payer/payee/amount. No public arbitrary-hash-to-PAID route; one active native tuple; authenticated hash recovery.
2. Real NEAR Testnet native single Transfer, FINAL tx/receipt checks. Fixed network/provider configuration; version-aware tx_status alias only on method-not-found. Inspect outcomes/dependencies/block times; attached FunctionCall deposit is not native invoice payment. Wrong/late/under/over/ambiguous evidence cannot auto-pay.
3. Deterministic evidence identities; one atomic short Serializable transaction for claims, ledger, conditional PAID update, audit event and original receipt. No RPC in transaction. Unique conflicts handled after rollback; bounded conflict retries. Same input safely returns same committed result.
4. Audit JSON v1 with JCS/SHA-256 and explicit trust scopes; append-only history. No digest-as-consensus/authenticity/global uniqueness claim.
5. Run unit/provider tests, real PostgreSQL 20/100-worker races, fault-injected rollback, browser tests and a separate **new real testnet payment** walkthrough. Deterministic mock-wallet CI is labelled simulation, not live payment evidence.

## Conditional P1, bounded by Nov 8 18:00 NZDT

Start bounded evidence spike only while P0 on track. Implement one Base USDC -> NEAR USDC route: EXACT_OUTPUT, ORIGIN_CHAIN deposit/refund, DESTINATION_CHAIN normal merchant wallet. Verify official quote signature, persist effective quote/address tuple before exposing funding. Enforce conservative expiry; never fund a dry or expired/unrecorded quote. No ANY_INPUT, custom recipient msg, confidential balance, smart-account batch, split output or general chain adapter.

Public token/RPC/status reads and dry=true quotes remain no-funds operations. **Any non-dry quote/real mainnet signing/spend needs explicit later authorization**; never reuse the research recipient/refund placeholders. Do not request keys or perform partner/funding/administrative applications.

Actual evidence must prove Base USDC source deposit and finalized inclusion, then exact NEAR USDC final wallet credit through the actual receipt graph. Resolve NEAR root signer; source payer differs from destination solver. API SUCCESS/nearTxHashes alone are not PAID. Investigate effective fee-policy discrepancy, actual TTL/cutoffs, recipient storage and returned hashes. Fail closed if unsupported/missing. Input-buffer refund is not merchant reversal.

Build independent proof CLI/page that runs with invoice DB unavailable and configured RPCs. Re-query and return fresh per-scope decisions. Missing business assertion/global claim registry remains UNVERIFIED; cross-chain link provider-attested unless independently established.

At gate: actual authorized settlement + repeatable source/destination verification + sound P0 -> GO. Otherwise P1 OFF, remove funding CTA/completed-cross-chain claims, retain optional DRY RUN preview and finish native P0. Do not build substitute bridge/contracts or let P1 consume P0 repair time.

## Reliability and delivery invariants

- No early core implementation, secrets/custody, float money, client/API-only PAID or silent partial/excess credit.
- Database constraints plus atomicity enforce exactly-once effects **within this database**, not global exclusivity.
- Receipt/claim/audit history cannot be arbitrarily rewritten by runtime role; post-settlement contradictions append exceptions/corrections, never delete/release evidence.
- Public proof excludes PII; wallet/amount/time linkage is disclosed and sharing opt-in.
- Report PASS/PARTIAL/FAIL/BLOCKED/NOT RUN, network, real vs fixture, exact commands, tx/receipt IDs, DB counts and limitations. No fabricated CI/deployment/evidence.
- After required checks pass, proceed to next delivery gate; cut optional CSV/SDK/visual extras before correctness.
- No public deployment/repo publication, push/PR, X post or final submission without the corresponding user authorization. This prompt does not grant it.

The user handles personal/legal eligibility, funding and organizer matters. Focus on implementation -> test -> fix -> deliver within frozen technical scope.
