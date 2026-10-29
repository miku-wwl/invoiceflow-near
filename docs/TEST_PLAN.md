# Test and Acceptance Plan

> Planned tests, **not yet executed**. Automated tests and evidence are created only after the official development start.

## Must-pass matrix

| ID | Test | Expected behavior |
|---|---|---|
| T01 | Create invoice with valid amount/asset/payee | Immutable id and public token, atomic amount persisted |
| T02 | Invalid negative/overprecision amount | Validation error; no invoice |
| T03 | Real testnet native Transfer with matching sender, receiver and amount | RPC `FINAL`, invoice `PAID`, one ledger row |
| T04 | A valid transaction sent to another recipient | Reject, no invoice mutation |
| T05 | Valid transaction with insufficient amount | No `PAID`; explicit failure reason |
| T06 | Wrong token contract/network | Reject, no credit |
| T07 | Failed transaction or incomplete receipts | Reject or await finality |
| T08 | Unknown hash / RPC timeout / rate limit | Retryable, no paid state |
| T09 | Same hash applied to two invoices | Exactly one invoice may consume the transfer |
| T10 | 20–100 duplicate/concurrent verification requests | Exactly one successful reconciliation and receipt |
| T11 | Wallet callback lost, client refreshes | Verification can be resumed by submitted hash |
| T12 | Invoice canceled/expired before payment per policy | No silent credit; explicit exception flow |
| T13 | Public receipt exported as JSON/CSV | Contains source, destination, amounts, network and timestamps, no PII leak |
| T14 | Receipt verifier in an unauthenticated browser | Independently queries source data and reproduces the decision |
| T15 | Merchant A reads/writes Merchant B's invoice | Unauthorized access prevented |
| T16 | Secret accidentally in client bundle / tracked files | CI gate fails |

## 1Click conditional matrix

| ID | Test | Expected behavior |
|---|---|---|
| I01 | Live supported asset selection + `EXACT_OUTPUT` dry quote | Quote view labelled **DRY RUN**, not a transfer |
| I02 | Real authorized 1Click payment | Origin/deposit → status → destination evidence → `PAID` |
| I03 | Quote expiration | New quote required; old attempt not falsely paid |
| I04 | `INCOMPLETE_DEPOSIT` | No invoice credit |
| I05 | `REFUNDED` or `FAILED` | No invoice credit; refund evidence preserved |
| I06 | 1Click reports `SUCCESS` but destination evidence mismatches | Do not reconcile; show verifiable discrepancy |
| I07 | Same deposit event replayed or polled repeatedly | Single deterministic outcome |

## Test layers

- **Unit:** monetary math, token decimals, pure verification decisions, transition table, public-receipt canonicalization.
- **Integration:** Postgres unique constraints, concurrency transactions, API authorization, provider stubs, retry semantics.
- **Live chain:** at least one real NEAR testnet native transfer; separate mainnet Intents test only if authorized.
- **E2E:** Playwright merchant → invoice → wallet/verified evidence → paid → independent receipt verifier, plus a negative path.
- **CI:** lint, typecheck, unit, integration, build and dependency/security sanity checks.

## Evidence required before claiming completion

Save timestamped command results, CI run URL, `txHash`/explorer URL, invoice ID, verification verdict, final DB row counts and screenshots. If tests are simulated or stubbed, label them as such. Do not record private keys or private customer data.
