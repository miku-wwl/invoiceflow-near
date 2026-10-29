# Judge Demo Script — intended end-state

This is a **target script**, not a presently runnable demo. All transactions must be real and traceable or specifically labelled `SIMULATED TEST` / `DRY RUN`.

**Phase 0.5 refresh 2026-10-29:** [Architecture](ARCHITECTURE.md) and [Delivery Plan](DELIVERY_PLAN.md) govern. P1 is one Base USDC -> NEAR USDC mainnet wallet route, conditional on the Nov 8 18:00 gate. No payment or demo has run. Use a separately authorized real amount; 100 USDC is a design example, not a spending instruction.

## Primary path — 90–120-second edited walkthrough

| Action (`do`) | Expected observation (`see`) |
|---|---|
| Create an invoice for 100 USDC with a merchant settlement account | Unique invoice identifier and public checkout URL |
| Open the public URL as a payer | Amount, asset, recipient, supported chains and clear status |
| Choose a supported source-chain asset | 1Click `EXACT_OUTPUT` quote, atomic amounts, fees, slippage and deadline |
| Sign an authorized real payment in a wallet | Source-chain transaction hash, not an app-generated fake hash |
| Observe status changes | Deposit pending/detected/processing, then verified settlement |
| Open paid invoice | `PAID` only after actual final destination evidence and DB commit |
| Download JSON and open a separate verifier process/browser with invoice DB unavailable | Contract, recipient, atomic amount, actual receipt identities and verifier version |
| Press `Verify` | Fresh RPC checks; destination FINAL and source finalized evidence; invoice binding/uniqueness limits explicit, cross-chain linkage provider-attested |
| Attempt to assign the same tx to invoice B | Explicit `CONSUMED` / replay-rejected result; invoice B remains unpaid |

The waiting time for actual chain settlement may be edited in a short video, but the underlying transaction IDs, chain outcome and time sequence must remain verifiable. Do not promise a fixed settlement time.

API SUCCESS can precede our independent verification. Base source finality may lag sequencer inclusion; show pending until the required evidence exists. The proof is an RPC-backed evidence bundle, not a consensus/light-client proof. Replay prevention is enforced in this application's DB; an external verifier cannot prove global uniqueness. If a negative example uses a fixture, display SIMULATED TEST.

## Fallback path

If mainnet 1Click settlement is not fully proven, create a **testnet NEAR-denominated invoice** and demonstrate a real single Transfer, payer ownership, FINAL receipts, exactly one reconciliation/audit receipt and rejected reuse. Do not mark a USDC invoice paid with native NEAR. Clearly label the single-chain network; remove completed-cross-chain claims. A dry preview, if retained, says DRY RUN and has no funding CTA.

## Reviewer setup (to fill after implementation)

- Live URL: `TBD — NOT DEPLOYED`
- Public repository URL: `TBD — USER CREATES`
- Test invoice / public link: `TBD — NOT CREATED`
- Real transaction ID / explorer link: `TBD — NOT CREATED`
- Test command / CI run: `TBD — NOT EXECUTED`
- X demo post link: `TBD — USER PUBLISHES`
