# Architecture — frozen design, implementation NOT STARTED

**Phase 0.5 · 2026-10-29 · Pacific/Auckland · CONDITIONAL_GO**

This is the single normative architecture. [Final research](FINAL_TECHNICAL_RESEARCH_2026-10-29.md) records evidence and decisions; [tests](TEST_PLAN.md) define acceptance; [delivery](DELIVERY_PLAN.md) owns dates. Tables/interfaces below are specifications, not implemented components. No contest core code before **2026-11-01 13:00 NZDT**.

## 1. Frozen stack and scope

Next.js App Router + TypeScript + Tailwind; PostgreSQL + **Prisma ORM 7.x**, with exact supported patch, PostgreSQL driver/adapter and compatible Node version locked on opening day. Use [versioned Prisma 7 docs](https://www.prisma.io/docs/orm/v7/prisma-client/queries/transactions). Default docs now describe a different 8.x API; do not install unqualified `latest` or mix APIs. No packages/scaffolding were installed this phase.

```text
Merchant dashboard / public checkout / audit receipt
                 |
Authenticated API -> invoice service -> payment-attempt service
                 |                         |
                 |                 P0: NEAR Testnet native Transfer
                 |                 P1: Base USDC -> NEAR USDC via 1Click
                 |                         |
                 |                 independent chain evidence verifier
                 |                         |
                 +-------------> atomic reconciler -> PostgreSQL
                                                  -> immutable audit receipt
Downloaded proof -> separate verifier -> configured public RPCs
```

**P0:** invoice draft/create/read/update/cancel, real testnet native payment, FINAL verification, deterministic reconciliation, audit JSON, automated tests and runnable demo. Edit/delete only drafts; issued money fields are immutable; archive where audit records exist. Stablecoin invoices can be issued but only the enabled matching adapter can pay them. A native P0 invoice is denominated in testnet NEAR; it cannot discharge a USDC invoice through an invented exchange rate. No partial-payment aggregation or automatic overpayment/refund handling.

**P1:** one source route: Base mainnet USDC to Circle USDC on NEAR mainnet; `EXACT_OUTPUT`, `depositType=ORIGIN_CHAIN`, `recipientType=DESTINATION_CHAIN`, `refundType=ORIGIN_CHAIN`, public/SIMPLE deposit. Merchant normal NEAR wallet receives tokens, not its Intents balance. Exclude confidential swaps, custom recipient messages, arbitrary assets/chains, smart-account deposit decoding and split settlement. Platform support is wider than app support. Cut P1 at the [Nov 8 gate](DELIVERY_PLAN.md).

No custody/signing service, custom bridge/AMM, token issuance or new contract. Wallets sign client-side. Mainnet spending still requires separate authorization after opening.

## 2. Payment attribution

### P0 native NEAR

1. Authenticate payer control through a nonce-bound, expiring NEP-413 challenge. Message binds application domain, network, invoice and attempt. Validate signature, unused nonce and public key FullAccess membership on the configured network. A client account string/wallet callback is not authentication. Merchant APIs separately require authentication and ownership.
2. Freeze attempt snapshot: invoice version, expected payer, payee, asset, exact base-unit amount, start finalized block/checkpoint, expiry, claim nonce. Permit only one active native attempt per `(network, expectedPayer, recipient, amountAtomic)`; use server transitions to close/renew.
3. Wallet submits one direct native Transfer. Authenticated payer submits the hash with an invoice/attempt-bound claim nonce/session (or signed hash binding). Reject another payer's claim. Merchant-assisted recovery needs authenticated ownership and an audit event, never an arbitrary public hash endpoint.
4. Independently validate against the snapshot. Reject transactions before the attempt checkpoint, self-transfers and ambiguous batches. Late payments become review exceptions. Cancel/expire races serialize with reconciliation.

Native Transfer has **no invoice memo**. Authentication/exclusive attempts reduce hash theft and ambiguous concurrent checkout; they do not put business purpose on chain. Public proof labels invoice association application-attested. Do not claim automatic discovery of incoming transfers or chain-enforced invoice identity. Unsupported wallet signMessage means select a supported wallet or block the route, never remove ownership checks. Historical key membership is not automatically proven by a current key lookup; portable invoice attribution remains a separate trust scope.

### P1 1Click

Server derives the quote from the immutable invoice. Retain request **and effective echoed request**, timestamp/signature, correlationId, address/memo, amounts and expiry fields, bound to one attempt. After opening verify quote signatures with the pinned official SDK/key policy; this authenticates an API payload, not payment. Dry quotes are never fundable or reconcilable.

Persist a real quote before showing funding instructions. A remote quote created before a persistence failure is never exposed; a new request gets a new attempt. Never reuse an old deposit address for a re-quote. Reject conflicting echoed invoice fields, missing execution fields and unsupported modes. Initial funding cutoff is the earliest of app-created-at + 120 seconds, request deadline, returned deadline and timeWhenInactive, minus a 30-second safety margin (absent optional inactive time is ignored). The 120/30-second values are conservative app policy, not an official pricing TTL. Missing real-quote address/deadline blocks funding; real semantics must be checked at the P1 gate. Bind the unique `(network, depositAddress, depositMemo-or-empty)` tuple. P1 accepts only a single direct EOA ERC-20 source transfer, with payer control verified through a network/domain/nonce-bound wallet session.

Poll server-side by address plus memo when supplied. Status/hash arrays are discovery hints. `SUCCESS` means delivery **according to 1Click**; independently verify the merchant credit. Source deposit, internal Intents execution and final withdrawal are separate events. Missing hashes, unknown NEAR signer, ambiguous receipts or unexplained fees remain unverified. Resolve a destination hash's signer through public explorer/indexer data or a candidate signer, then independently confirm with RPC: NEAR's `tx` query needs `sender_account_id`, not hash alone. No indexer-to-PAID shortcut.

## 3. Verification contract

Design interface: `verifyPayment(invoiceSnapshot, attemptSnapshot, evidence, configuredProviders)`.

| Decision | Effect |
|---|---|
| `MATCH_FINAL` | Eligible for DB reconciliation, not yet PAID |
| `PENDING_RETRYABLE` | Backoff/resume; no credit |
| `MISMATCH` | Reject candidate; unpaid |
| `EXCEPTION_REVIEW` | Late, excess, ambiguous or contradictory evidence; no automatic credit |
| `FAILED` / `REFUNDED` | No credit; keep evidence |
| `CONSUMED` | Return same-invoice committed result or reject cross-invoice replay |

Fixed configured HTTPS providers only. Check `status.chain_id`, syncing state and configured network/genesis checkpoint when initializing providers. A proof cannot choose its RPC URL. Two providers detect faults but do not constitute a light client.

### NEAR native P0

- Query `tx` and receipt details via `tx_status`, `wait_until: FINAL`. `tx_status` needs nearcore 2.14+ (including supported RCs); fall back to `EXPERIMENTAL_tx_status` only on method-not-found, without lowering finality. Both queried testnet providers support both names today.
- Verify returned hash, signer, receiver, exactly one Transfer, positive exact `deposit` in yoctoNEAR. Asset is native NEAR on the fixed network, not `wrap.near`. P0 rejects delegated, self-transfer, multi-action and FunctionCall candidates.
- Require `final_execution_status=FINAL`, successful transaction outcome and successful payment receipt; resolve relevant payment/refund outcome dependencies. `SuccessReceiptId` is a link, not delivery. Top-level SuccessValue alone is insufficient.
- Fetch transaction/settlement-receipt block headers. Record hashes, heights and nanosecond timestamps as strings. Settlement time comes from the payment receipt block; observation/reconciliation times are separate. Enforce the immutable attempt/issuance window. Receipt list lengths alone do not establish completeness; obtain enough bodies/outcomes to identify payment actions, fetching known receipt IDs with `EXPERIMENTAL_receipt` where supported. An unavailable body needed for attribution blocks verification.

### NEP-141 destination P1

The root receiver may be a token contract/orchestrator, not merchant. Trace receipt dependencies and decode actual calls. Only trust transfer logs from the allowlisted token contract's successful receipt `outcome.executor_id`; parse NEP-141 `EVENT_JSON` using receipt/log/data indices. Match token-owner sender, merchant receiver and integer amount. Attached 1 yoctoNEAR is not the token amount. Same-symbol logs from another executor are invalid.

For `ft_transfer_call`, follow `ft_on_transfer`/`ft_resolve_transfer` through FINAL and account for returned tokens to determine **net merchant credit**. Successful parent + failed child is possible. Missing paths stay pending; ordinary unused-gas refunds do not negate FT credit. Balance snapshots/metadata are diagnostics, not transaction-specific payment evidence. Read-only `storage_balance_of` preflight can detect missing recipient registration; storage registration is a later authorized transaction.

P1 initially accepts one unambiguous final wallet-credit transfer with no unresolved token return. Unrecognized withdrawal paths block P1 until fixture-backed verification exists; never sum every apparent log. Destination root signer/solver need not equal Base payer: record source owner and destination execution sender separately.

### Base source P1

Require `eth_chainId=8453`, successful receipt, canonical inclusion block and ERC-20 Transfer emitted by Base USDC `0x833589fcd6edb6e08f4c7c32d4f71b54bda02913`, from authenticated payer to bound deposit address for funded amount. Pin token identity against [Circle](https://developers.circle.com/stablecoins/usdc-contract-addresses), not symbol. Verify inclusion at/below `finalized` head and block hash against canonical block at that height. Unsupported tags or provider conflicts remain pending. Sequencer inclusion/a generic confirmation count is not L1-derived finality. Second-provider checking improves evidence; it does not independently verify the entire solver/bridge. The cross-chain causal link remains provider-attested unless separately proven.

### Availability policy

Use bounded timeouts, jittered exponential backoff, per-provider budgets and persisted next-check times. Initial app policy: 10-second client timeout, 2/4/8/16/30-second retry delays with jitter, at most five immediate retries then scheduled/manual resume. These are app settings, not SLAs. 429/5xx, timeout, shard mismatch, unavailable archives and missing receipts are retryable. Rotate only within the same network; use archival access where needed. Reject malformed hashes; unknown well-formed hashes get a bounded observation window then close unverified. FINAL queries for unknown hashes may time out; timeout proves neither failure nor nonexistence. Provider disagreements block credit.

## 4. PostgreSQL + Prisma model

Opaque UUID IDs; UTC `timestamptz`. Money is integer **base units**: JS BigInt for computation, decimal strings in JSON, **Prisma 7 `Decimal @db.Decimal(78,0)`** in PostgreSQL. SQL signed bigint is too small for yoctoNEAR (1 NEAR = 10^24). Validate canonical unsigned digits before insertion because numeric scale zero can round fractions. Enforce positive finite values and asset-specific bounds (NEAR/FT u128; ERC-20 uint256 where applicable). No floating-point money. Freeze decimals per asset: NEAR 24, selected USDC 6; 100 USDC = `"100000000"`.

| Entity | Critical fields | Constraints / ownership |
|---|---|---|
| Merchant | id, authenticated owner, displayName, settlementAccount | Owner check per mutation; no enterprise role system |
| Invoice | id, merchantId FK, publicTokenDigest, version, amountAtomic, assetId, network, recipient, status, issuedAt, expiresAt, paidAt?, exceptionReason? | Unique token digest; money/network/payee frozen on issue; positive amount; preserve paid record |
| CheckoutQuote | id, attemptId FK, provider, correlationId, requested/effective request JSON, quote JSON, signature/timestamp, source/destination IDs, amountIn/minAmountIn/amountOut, requestedDeadline, returnedDeadline?, timeWhenInactive?, appExpiresAt, sourceNetwork, depositAddress?, depositMemoKey, isDry | Unique(provider, correlationId); unique non-null network/address/memo tuple; one exposed real quote per attempt; dry cannot be fundable |
| PaymentAttempt | id, invoiceId FK, invoiceVersion, adapter, payerAccount, authenticationRef, claimNonceDigest, snapshot amount/asset/payee, checkpointHash/height, expiresAt, state, nativeBindingKey?, active, clientRequestKey, failureReason?, nextCheckAt?, revision | Unique(invoiceId, clientRequestKey); partial unique active nativeBindingKey; CAS revision protects worker state |
| ChainTransaction | id, network, txHash, signer?, blockHash/height?, finality, executionResult, queriedAt, provider, responseDigest | **Unique(network, txHash)**; observation alone does not consume money |
| SettlementEvidence | id, chainTransactionId FK, network, executionId, assetId, receipt/log/data indices, evidenceKey, sender, recipient, amountAtomic, finality, settlementBlock/time, verifierVersion, evidenceJson/digest | **Unique(network, evidenceKey)**; identity independent of caller-supplied wrapper; append-only verified snapshot |
| Reconciliation | id, invoiceId FK, attemptId FK, invoiceVersion, amountAtomic, policyVersion, decisionDigest, createdAt | **Unique(invoiceId)**; unique attemptId; composite FK ensures attempt belongs to invoice; one full settlement |
| EvidenceClaim | evidenceId FK, reconciliationId FK, role SOURCE/DESTINATION | **Unique(evidenceId)** across invoices/roles; unique(reconciliationId, role); destination required, source also required for P1 |
| AuditReceipt | id, reconciliationId FK, proofVersion, canonicalPayload, integrityHash, createdAt | Unique reconciliationId; one immutable original, corrections via audit events |
| AuditEvent | id, invoiceId FK, sequence, eventType, actorRef, redactedPayload, previousHash?, eventHash, createdAt | Unique(invoiceId, sequence); append-only; public projection excludes PII |

Absent memo uses non-null `depositMemoKey=""`; do not rely on nullable unique semantics. Validate chain-specific address/hash encodings: lowercase hex for Base, case-sensitive Base58 for NEAR hashes. Verifier-derived evidence keys:

- Native: `native/<paymentReceiptId>/<actionIndex>`.
- NEP-141: `ft/<contract>/<receiptId>/<logIndex>/<dataIndex>`.
- ERC-20: `erc20/<contract>/<txHash>/<logIndex>`.

One Transfer in P0 permits transaction-level replay rejection, while receipt identity avoids later aliasing. One complex transaction may have several transfers; do not consume by root hash alone. A ChainTransaction observation can exist without claiming a mismatched payment.

Post-opening migrations add CHECKs, partial indexes, composite FKs and append-only grants/triggers beyond Prisma schema syntax. Runtime role cannot UPDATE/DELETE receipts, claims, evidence or audit events; migration role controls schema. Restrict deletion of referenced invoices. A deferred constraint trigger must enforce required claim cardinality at commit (one destination, and one source for P1), with adapter/invoice consistency; table-level uniqueness alone cannot enforce that cross-row requirement. DB administrators can rewrite history: permissions are not consensus immutability. For audit chaining, hash JCS(event fields excluding eventHash), including sequence and previousHash; genesis previousHash is null and later events name the committed predecessor. A per-invoice hash chain exposes modified middle events against a retained digest, but cannot reveal total rewrite/truncation without an external checkpoint.

## 5. Atomic reconciliation

1. Verify public evidence **outside** the DB transaction; capture invoice version, attempt revision, policy, normalized transfer identity and response digests. No RPC calls while holding DB locks.
2. Start short Prisma 7 interactive transaction at `Serializable`. Re-read/lock invoice then attempt consistently (parameterized row lock where needed). Recheck ownership, frozen snapshot, version, open state, amount and receipt-time policy. Same-request replay returns stored result only when evidence identity matches.
3. Insert or reuse the same verified normalized evidence reference, then insert Reconciliation and EvidenceClaims (destination; source too for P1). Only positively matched evidence is claimable. Candidate hash submission never reserves it.
4. Conditionally update matching unpaid invoice to PAID; require affected count = 1. Close attempt; append audit event and canonical AuditReceipt with IDs/times matching DB rows. Commit all-or-nothing; never respond PAID before commit.
5. Constraint failure, receipt-insert failure or conditional update failure rolls back **every** claim, ledger write and status change. Roll back before handling unique conflicts; read committed winner in a fresh transaction. Same invoice+evidence returns its receipt; another invoice returns CONSUMED; paid invoice+different evidence returns conflict/review. Do not continue inside an aborted SQL transaction.
6. Bounded serialization/deadlock retries re-read state (Prisma 7 P2034 / SQLSTATE 40001 or 40P01 as appropriate to pinned driver). Exhaustion returns retryable. Crash after commit/before response safely replays.

These are exactly-once **effects within this database** under at-least-once delivery, not global exclusivity across applications.

## 6. States and recovery

```text
Invoice: DRAFT -> ISSUED -> PENDING -> PAID
                    |         |
                    +-> CANCELLED / EXPIRED
         PENDING -> ISSUED only when no active attempts remain
Attempt: CREATED -> AWAITING_PAYMENT -> VERIFYING -> VERIFIED -> RECONCILED
                                      |            (commit required)
                                      +-> RETRYABLE -> VERIFYING
                                      +-> REJECTED / FAILED / REFUNDED
                                      +-> EXCEPTION_REVIEW
```

VERIFIED means chain-match eligibility; RECONCILED requires DB commit. Worker updates use CAS/revision; append observations without regressing state from stale timestamps. Unknown upstream enums fail closed. Invoice state is never copied from swap status.

| Condition / upstream state | Recovery |
|---|---|
| Quote expires before funding | Disable funding; fresh quote/attempt. Preserve old tuple; keep monitoring observed deposits |
| PENDING_DEPOSIT / KNOWN_DEPOSIT_TX | Await/verify source, no credit |
| INCOMPLETE_DEPOSIT / insufficient funding | No credit; display shortfall; monitor until deadline/refund. App-driven top-ups excluded; upstream can still observe additional deposits |
| Slippage exceeded / unavailable route | Preserve observed status, wait for actual refund evidence; new quote only on a new attempt |
| PROCESSING / swap pending | Persist/resume polling; local timeout is not terminal chain failure |
| Internal execution done / delivery delayed / SUCCESS without evidence | VERIFYING/RETRYABLE, unpaid until merchant FINAL match |
| FAILED / REFUNDED before commit | No credit; keep evidence; separate new attempt allowed |
| Unused input/slippage buffer refunded after successful delivery | Record separately; does not reverse exact merchant credit |
| Repeated polling/webhook | Webhook only wakes a job; fetch authoritative status/evidence; same committed outcome |
| Cancel/expiry races with confirmation | Serialize on invoice; enforce receipt window; now-expired/cancelled invoices route late payment to review |
| Contradictory observation after PAID | Append exception, independently reverify; retain original ledger/receipt/claims. Source refund alone proves no merchant debit. Proven delivery reversal gets correction/dispute record, never delete or release claims |

No backend-initiated refunds in P0/P1. No manual status flag replaces evidence. An application funding cutoff does not promise immediate bridge refund.

## 7. Settlement Proof v1

P0 exports an audit JSON bundle. P1 adds independent local CLI/page reading that file plus public RPCs, without invoice DB credentials. Public receipt sharing is merchant opt-in via unguessable capability, no indexed listing. UUIDs, amounts, addresses and times are linkable financial metadata: disclose before sharing. Exclude names, email, line items, customer references, API keys, auth challenges and internal dumps.

| Field | Representation / meaning |
|---|---|
| proofVersion, verifierVersion, policyVersion | Explicit strings; unknown versions rejected |
| invoiceId, invoiceVersion | Opaque identity; business association application-attested |
| invoiceAmount | `{assetId, amountAtomic: string, decimals}`; no implied USD conversion |
| settlementAsset, settlementAmount | Actual contract/native ID and received atomic string |
| network, recipient | Destination network and merchant address snapshot |
| sourceTxHash | Nullable P0; required for advertised P1 route |
| destinationTxHash | Actual root transaction, never just intent hash |
| sourceEvidence, destinationEvidence | Network, sender/root signer, token, deposit tuple, receipt/log/data IDs, evidenceKey; enough to re-query |
| quoteEvidence | P1 correlationId, signed quote/non-sensitive request, timestamps and provider claims; linkage provider-attested |
| finalityEvidence | Per-leg RPC method/milestone, execution status, block hashes/heights, receipt IDs/statuses, observedAt, providers; Base finalized head |
| settlementTimestamp | Destination payment-receipt block time UTC; retain raw NEAR nanoseconds as string |
| reconciliationId, reconciledAt | Database references, not chain facts |
| verificationResult | Issuer's original timestamped per-scope assessment, never blindly reused |
| integrityHash | `sha256:` + lowercase hex, canonical payload digest |

Envelope `{payload, integrityHash}`: all fields except integrityHash go inside payload, including original verificationResult and prior-event digest if present. RFC 8785 JCS -> UTF-8 -> SHA-256. Money/heights/nanoseconds are canonical decimal strings. Reject duplicate JSON keys, invalid Unicode, extra schema fields and noncanonical amounts. Fresh verifier results stay outside the immutable payload.

An unkeyed hash detects changes against a **retained/trusted** digest. An attacker can alter JSON and recompute it. It authenticates neither issuer, invoice purpose nor consensus. Re-query chain predicates even if hash matches; changing invoice ID alone can leave a valid transfer, so external invoice binding remains unverified without a trusted invoice assertion.

Required result scopes:

| Scope | Permitted result / boundary |
|---|---|
| integrity | MATCH / MISMATCH, representation consistency only |
| destinationTransfer | VERIFIED_VIA_RPC / MISMATCH / UNVERIFIED |
| sourceTransfer | VERIFIED_VIA_RPC / MISMATCH / UNVERIFIED / NOT_APPLICABLE |
| crossChainLink | PROVIDER_ATTESTED / UNVERIFIED / NOT_APPLICABLE |
| invoiceBinding | APPLICATION_ATTESTED from issuer; UNVERIFIED externally without trusted snapshot |
| claimUniqueness | ENFORCED_IN_ISSUER_DB from issuer; UNVERIFIED externally; no global claim |
| consensusProof | NOT_INCLUDED |
| issuerSignature | NOT_INCLUDED in v1 |

Verifier: strict schema/version/size checks -> digest -> independently configured network checks -> fetch transaction/receipts -> evaluate asset/recipient/amount/finality -> fresh scoped result/time. No arbitrary proof/explorer URL fetch (SSRF). Never trust cached verdict or DB PAID. Missing archive/provider yields UNVERIFIED, not PASS. RPC replay is independently re-checkable under RPC trust, not a trustless consensus proof.

| Mechanism | Decision |
|---|---|
| JCS + SHA-256 | P0 mandatory: integrity, not authenticity |
| Append-only audit + simple per-invoice hash chain | P0 history aid, no public immutability claim |
| Signed audit receipt | Deferred; issuer-key lifecycle/trust distribution cost; never wallet custody keys |
| Merkle audit tree | P2; unanchored inclusion proves neither uniqueness nor finality |
| NEAR Light Client Proof | P2; trusted head/validator transitions and inclusion validation needed, not merely RPC proof JSON |
| On-chain attestation | P2; deployment/fees do not remove source/attribution assumptions |

This bundle is **not a bank payment document or legal determination of payment/discharge**. It supports only independently checked technical predicates.

## 8. Security and freeze

Merchant auth/object authorization; payer control; CSRF/session protection; immutable settlement snapshots; server asset/network allowlists; bounded proof/RPC payloads; parameterized SQL; server-only API secrets; protected jobs; no arbitrary RPC URLs. Polling/webhooks are untrusted. Stablecoin issuer/upgrade/freeze policy and provider/bridge availability remain external assumptions. Implementation and adversarial acceptance tests remain **NOT RUN**.

Architecture and P0/P1 boundaries are frozen; P1 feasibility is conditional on real authorized settlement, complete receipts/signer discovery and the Nov 8 gate. Opening-day version pinning does not authorize redesign or pre-opening code.
