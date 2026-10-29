# Delivery Plan — Pacific/Auckland

## Hard dates

- **2026-11-01 13:00 NZDT**: official start and earliest contest-specific core code.
- **2026-11-14 18:00 NZDT**: internal submission target.
- **2026-11-15 12:59 NZDT**: organizer deadline.

## Releases

| Dates | Engineering goal | Ship gate |
|---|---|---|
| 10/28–10/31 | Rules/architecture/read-only API checks, docs only | No contest core implementation |
| 11/1–11/3 | Next.js app, schema, private merchant UI, public invoices | Local build and invoice integration tests pass |
| 11/4 | Native NEAR testnet transfer and RPC verifier | One real finalized transfer and one failure case recorded |
| 11/5–11/6 | Reconciliation state machine, unique DB constraints, atomic commit | Parallel/replay events result in one paid row |
| 11/7–11/8 | Conditional 1Click EXACT_OUTPUT live path and independent destination check | Source/destination evidence verifiable; otherwise cut from claims |
| 11/9–11/11 | Open receipt verifier, negative cases, merchant UX | Public evidence verifier works without merchant login |
| 11/12–11/13 | Production-style deployment, CI, README, screenshots/video | Fresh reviewer can follow `do X -> see Y` |
| 11/14 before 18:00 | Human-approved public repo, demo post and submission | One verified submission only |

## Branching decision

If 1Click does not have a fully verified settlement path by **11/8**, prioritize finishing the NEAR testnet P0 and transparent proof verification. Never portray dry quotes or mock receipts as actual paid invoices.

## Must-have / conditional / excluded

- **P0:** invoice lifecycle, testnet real payment, final chain verification, exactly-once accounting, JSON/CSV receipt, online demo, tests and docs.
- **Prize target (conditional):** 1Click `EXACT_OUTPUT`, cross-chain settlement, terminal status recovery, independent destination-chain verification, portable proof.
- **Stretch:** merchant API and signed webhook with retries.
- **Excluded:** AI agents, ERP, mailing campaigns, custom bridge, unnecessary contract, fiat off-ramp, token issuance.

## Daily execution discipline

Every day: implement one vertical slice → run unit/integration/E2E tests → record actual result → fix regressions → update README/demo evidence. Keep features only if they survive real verification. No claimed test coverage without the test command and result.
