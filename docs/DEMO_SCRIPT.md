# Judge Demo Script — intended end-state

This is a **target script**, not a presently runnable demo. All transactions must be real and traceable or specifically labelled `SIMULATED TEST` / `DRY RUN`.

## Primary path — 90–120-second edited walkthrough

| Action (`do`) | Expected observation (`see`) |
|---|---|
| Create an invoice for 100 USDC with a merchant settlement account | Unique invoice identifier and public checkout URL |
| Open the public URL as a payer | Amount, asset, recipient, supported chains and clear status |
| Choose a supported source-chain asset | 1Click `EXACT_OUTPUT` quote, atomic amounts, fees, slippage and deadline |
| Sign an authorized real payment in a wallet | Source-chain transaction hash, not an app-generated fake hash |
| Observe status changes | Deposit pending/detected/processing, then verified settlement |
| Open paid invoice | `PAID` only after actual final destination evidence and DB commit |
| Open JSON receipt or proof link in a separate browser | Payment provenance, validator version, chain explorer links |
| Press `Verify` | Fresh chain data rechecked independently of DB's paid flag |
| Attempt to assign the same tx to invoice B | Explicit `CONSUMED` / replay-rejected result; invoice B remains unpaid |

The waiting time for actual chain settlement may be edited in a short video, but the underlying transaction IDs, chain outcome and time sequence must remain verifiable. Do not promise a fixed settlement time.

## Fallback path

If mainnet 1Click settlement is not fully proven, demonstrate a **real NEAR testnet native Transfer** and correctly label it as a separate single-chain capability. The README and video must explicitly avoid any claim of completed cross-chain payment.

## Reviewer setup (to fill after implementation)

- Live URL: `TBD — NOT DEPLOYED`
- Public repository URL: `TBD — USER CREATES`
- Test invoice / public link: `TBD — NOT CREATED`
- Real transaction ID / explorer link: `TBD — NOT CREATED`
- Test command / CI run: `TBD — NOT EXECUTED`
- X demo post link: `TBD — USER PUBLISHES`
