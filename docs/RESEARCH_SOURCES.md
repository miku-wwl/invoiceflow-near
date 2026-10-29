# Research Sources & Evidence Levels

> Reviewed 2026-10-28. API observations are timestamped snapshots, not guarantees of future availability.

## Organizer primary sources

- [NEAR Hackathon official website](https://nearhackathon.com/) — event tracks, submission, dates.
- [Official HACKATHON.md](https://nearhackathon.com/HACKATHON.md) — precise submission contract, required human approval and immutable submission; **read live again before submit**.
- [NEAR Foundation announcement 2026-10-19](https://www.near.org/blog/near-hackathon-build-beyond-boundaries) — $250k prize table ($55k Grand Prize; $10k/$3k/$2k per track).

## Official technical primary sources

- [NEAR RPC transaction methods](https://docs.near.org/api/rpc/transactions) — `tx`, `tx_status`, `wait_until: FINAL`.
- [NEAR public RPC providers](https://docs.near.org/api/rpc/providers) — mainnet and testnet endpoints.
- [NEAR Wallet Selector](https://github.com/near/wallet-selector) — testnet support; client-side signing.
- [NEAR Intents FAQ](https://docs.near-intents.org/resources/faqs) — no general testnet deployment.
- [1Click Quickstart](https://docs.near-intents.org/integration/distribution-channels/1click-api/quickstart) — production mainnet caveat.
- [1Click Swap Types](https://docs.near-intents.org/integration/distribution-channels/1click-api/swap-types) — `EXACT_OUTPUT`.
- [1Click Fee Configuration](https://docs.near-intents.org/integration/distribution-channels/1click-api/fee-config) — quote fees and units.
- [NEAR maintained example SDK](https://github.com/near-examples/near-intents-examples) — quote/deposit/status recipes.
- [NEAR maintained 1Click integration rules](https://github.com/near/agent-skills/blob/main/skills/near-intents/rules/api-quote.md) — request fields and response structure.
- [NEAR maintained 1Click status reference](https://github.com/near/agent-skills/blob/main/skills/near-intents/rules/api-status.md) — terminal and nonterminal states.

## Competitive primary sources

- [NEAR Intents Cross-Pay](https://intents.near.org/use-cases/cross-pay)
- [Request Finance invoice API](https://docs.request.finance/invoices)
- [Copperx invoicing](https://copperx.io/invoicing)
- [Payclave payment verification](https://www.payclave.com/guides/onchain-payment-verification)

## Historical winner primary sources

- [Roketo and MetaBUIDL](https://near.org/blog/metabuidl-hackathon-winners)
- [MetaBUILD 2 winners](https://www.near.org/blog/metabuild-2-hackathon-winners)
- [NEARCON winners](https://www.near.org/blog/meet-the-2023-nearcon-irl-hackathon-winners)
- [Useful Agents winners](https://www.near.org/blog/building-useful-agents-sf-hackathon-shows-the-power-of-practical-ai)

## Classification

- **CONFIRMED, official page:** prize breakdown, timelines and allowed submission format.
- **OBSERVED, read-only API test:** testnet RPC queries and a 1Click dry exact-output quote succeeded on 2026-10-28.
- **NOT VERIFIED:** a self-initiated InvoiceFlow transfer, actual Intents cross-chain settlement, independent destination verification, running application, CI, deployment and demo.
- **ASSUMED:** unpublished award stacking, prize conversion/payout mechanics and pre-existing-code reuse boundaries.
