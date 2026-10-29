# Architecture — target design, not yet implemented

## Stack and boundaries

```text
Next.js App Router + TypeScript + Tailwind
  Merchant dashboard / Public invoice / Checkout / Receipt verifier
       |
  HTTP API + typed service layer
    ├─ Invoice service / token-aware decimal math
    ├─ Payment adapters: NEAR native (P0); 1Click (conditional)
    ├─ Chain evidence verifier: NEAR RPC / source-chain RPC
    ├─ Reconciliation service: database atomic commit
    └─ Audit service: canonical evidence bundle / independent re-verifier
       |
  PostgreSQL + Prisma
```

No backend-controlled signing keys, private seed phrases, custodial balances, self-issued token or new cross-chain bridge. The payee's wallet stays under the payee's control. `PAID` never results from a frontend flag alone.

## Minimal data model

| Entity | Critical fields | Constraints |
|---|---|---|
| Merchant | id, displayName, settlementAccount | merchant access restricted |
| Invoice | id, publicToken, merchantId, amountAtomic, assetId, network, payeeAccount, expectedPayer?, status, issuedAt, dueAt | publicToken unguessable; amount immutable after issue |
| CheckoutQuote | id, invoiceId, correlationId, sourceAssetId, destinationAssetId, amountInAtomic, amountOutAtomic, depositAddress?, deadline, state | unique correlationId; attached to one invoice |
| PaymentAttempt | id, invoiceId, quoteId?, sourceChain, txHash, sender, state, failureReason, verifiedAt | unique chain + txHash if simple Transfer; for complex tx also action/receipt identity |
| SettlementEvidence | id, attemptId, sourceTxHash, destinationTxHash, destinationAsset, receivedAtomic, receiver, finality, verifierVersion, evidenceJson | unique consumed transfer identity |
| Reconciliation | id, invoiceId, evidenceId, amountAtomic, reconciledAt | unique paid reconciliation per invoice |
| AuditReceipt | id, invoiceId, receiptVersion, evidenceDigest, receiptJson, createdAt | immutable snapshot |

Use atomic integer strings / `BigInt`, explicit token decimal metadata and strict runtime schemas. Avoid JavaScript floating-point token arithmetic.

## Independent verification contract

Proposed interface: `verifyPayment(invoiceSnapshot, chainEvidence, provider)` returns a typed decision:

- `MATCH_FINAL`: recorded network, asset contract, recipient, minimum/exact amount and terminal success match the immutable invoice.
- `PENDING_RETRYABLE`: unknown tx, temporary RPC error or insufficient finality; never paid.
- `MISMATCH`: wrong chain, asset, recipient, amount, payer or timestamp.
- `REFUNDED` / `FAILED`: terminal negative outcomes, never paid.
- `CONSUMED`: transfer already assigned to another invoice.

For NEAR P0 use `tx`/`tx_status`, a known `tx_hash`, `sender_account_id`, and `wait_until: FINAL`; inspect `transaction.receiver_id`, `actions.Transfer.deposit`, successful outcome, relevant receipt outcomes and block time. Reject ambiguous multi-action claims until action-level attribution is implemented. Network is fixed by the configured endpoint and verified `chain_id`.

**Crucial limitation:** Native NEAR Transfer does not include an invoice-specific memo. A known transaction hash + payer account is the P0 association mechanism. Fully automatic discovery of arbitrary incoming transfers requires indexer support or a distinct per-invoice receiving scheme. Do not claim the former exists without implementing it.

For conditional 1Click: persist quote `correlationId` and `depositAddress` with the invoice; poll status (`PENDING_DEPOSIT`, `KNOWN_DEPOSIT_TX`, `PROCESSING`, `SUCCESS`, `FAILED`, `REFUNDED`, `INCOMPLETE_DEPOSIT`). Verify actual source/destination chain hashes and final destination transfer separately when available. A quote `dry: true` must never be treated as a payment; NEAR Intents has no general testnet deployment.

## Atomic reconciliation

Within one database transaction:

1. Verify evidence outside the critical section; attach a deterministic evidence identity.
2. Attempt a unique insert for consumed transfer/evidence (`network, txHash, actionIndex` or chain-specific transfer identity).
3. Conditionally update the matching unpaid invoice to `PAID`.
4. Insert one `Reconciliation` row with unique `invoiceId`, then an immutable receipt snapshot.
5. Commit all-or-nothing; on a unique conflict, return the existing result or a clear `CONSUMED` response.

Never trust a caller-supplied status, arbitrary `txHash`, client clock or naked payment webhook. Retryable jobs and API calls converge to one result.

## State machines

```text
Invoice: ISSUED -> PENDING -> PAID
         ISSUED -> EXPIRED / CANCELLED
         PENDING -> ISSUED (candidate rejected)
         PENDING -> PENDING (retryable RPC/confirmation)

PaymentAttempt: DETECTED -> VERIFYING -> VERIFIED
                                  -> REJECTED
                                  -> RETRYABLE -> VERIFYING

1Click: PENDING_DEPOSIT -> KNOWN_DEPOSIT_TX -> PROCESSING
                                           -> SUCCESS / FAILED / REFUNDED
       PENDING_DEPOSIT -> INCOMPLETE_DEPOSIT
```

An invoice and the 1Click swap have **different states**; never equate `swap SUCCESS` with an invoice being `PAID` until business match and destination verification pass.

## Proposed route/API skeleton (implementation after opening)

```text
app/(merchant)/invoices
app/pay/[publicToken]
app/proof/[receiptId]
app/api/invoices
app/api/invoices/[id]/attempts
app/api/invoices/[id]/status
app/api/proofs/[id]/verify
lib/chain/near.ts
lib/intents/one-click.ts
lib/reconciliation/
lib/proofs/
prisma/schema.prisma
tests/unit tests/integration tests/e2e
```

## Security and trust notes

- Merchant mutation APIs require authentication and authorization; public links expose only necessary fields.
- Validate server-side asset allowlists and the immutable settlement-account snapshot.
- Store API credentials in server-side deployment secrets, not browser bundles; protect reconciliation jobs.
- Verify provider results, rate-limit public endpoints, bound polling and surface honest error states.
- A receipt hash detects representation changes only; actual settlement confirmation depends on rechecking consensus data and provider assumptions.
