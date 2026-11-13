# Vellio | Project Instructions

## Project
Vellio is a cross-chain invoicing and payment reconciliation platform for freelancers and agencies receiving international payments.

Tagline: One invoice. Payments without borders.

A freelancer creates an invoice and shares a payment link. A client pays with a supported asset and network. Standard payments use the official NEAR Intents 1Click API and settle in USDC on NEAR. Vellio verifies settlement before marking the invoice paid.

Read `VELLIO_BUILD_BRIEF.md` before making implementation decisions. It is the full product and technical specification.

## Hackathon track and judging priorities
Target track: NEAR Hackathon 2026, Real-World Money.

Prioritise:
1. NEAR technology used in the critical path, including official 1Click, NEP-141 USDC on NEAR, NEP-366 meta transactions and PingPay.
2. Real execution through actual APIs, transactions and verifiable settlement.
3. A useful cross-border invoicing and stablecoin payment workflow.
4. A complete, secure and demonstrable end-to-end product.

Imports, placeholders, README claims and mocked responses do not count as completed integrations.

## Required product workflow
- A freelancer can create and manage an invoice.
- Each invoice has a shareable public payment page.
- A client can select a supported source asset and network.
- The standard payment path uses the official NEAR Intents 1Click API.
- The intended destination asset is USDC on NEAR, identified from the live supported-token list.
- The backend monitors provider status and verifies destination settlement.
- Only verified successful settlement can mark an invoice paid.
- Payment attempts and invoice statuses are stored separately.
- The freelancer can review reconciliation records and export an accountant-friendly CSV.
- A separate agent payment path must integrate PingPay using its documented x402 flow.

## Required NEAR integrations
- Official NEAR Intents 1Click API for standard cross-chain payments.
- NEP-141 USDC on NEAR as the settlement asset.
- NEP-366 delegate actions and a restricted relayer for gasless Vellio contract actions.
- PingPay facilitator for the documented x402 Base USDC to NEAR recipient use case.

Verify sponsor documentation before implementing PingPay. Do not guess its endpoints, SDK methods, authentication, request fields or response fields.

Official 1Click has no official testnet deployment. Use its official mainnet API for the real 1Click path. NEAR contract and relayer development can use testnet. Tidepool is an optional independent community sandbox only. Never describe Tidepool as official 1Click infrastructure.

## Payment integrity and security rules
- Never simulate or fake a production payment.
- Never create fake transaction hashes, balances or payment confirmations.
- A dry-run quote is not a completed payment.
- Never mark an invoice paid based on a source-chain deposit alone, a client-submitted hash, an x402 payload or an HTTP success response.
- Verify the provider's successful destination settlement and actual destination amount on the backend.
- Use exact token IDs and decimals from trusted provider data. Do not assume or hardcode token identifiers.
- Use integer base units or precise decimal arithmetic. Never use floating-point arithmetic for money.
- Process status updates idempotently and preserve payment history.
- Keep invoice status separate from payment-attempt status.
- Keep secrets and privileged credentials server-side. Never commit `.env.local` or private keys.
- Restrict the NEP-366 relayer to Vellio's own contract and explicitly permitted methods. Validate signatures, nonce, expiry, gas limits and replay protection.
- Do not store private client information on-chain.
- Clearly separate test and production records and environments.

## Approved technology direction
Use the stack specified in `VELLIO_BUILD_BRIEF.md`. The current recommendation is:
- Next.js and TypeScript.
- Tailwind CSS and shadcn/ui.
- PostgreSQL and Supabase.
- A small Rust NEAR smart contract using near-sdk.
- NEP-366 delegate actions and a restricted relayer.

Do not replace an approved technology or provider without explaining the reason and asking first.

## Brand and interface
- Warm Cream: `#F5F0E6`
- Deep Ink: `#17172A`
- Tangerine: `#FF642D`
- Violet: `#8057EF`
- Warm Grey: `#D8D1C5`

Orange and violet are the core brand accents. Green and amber may be used for functional payment statuses only.

Use Space Grotesk for display text and DM Sans for interface text unless the full brief or owner changes this decision.

Visual direction: warm cream backgrounds, strategic partial grid patterns, bold clean outlines, simple geometric illustrations, generous negative space and balanced editorial layouts. Keep grids subtle and selective. Do not place decorative grids behind invoice data or checkout information.

Avoid generic crypto imagery, coin illustrations and blockchain symbols.

## How to work
- The project owner is not a developer. Explain technical decisions, errors and required actions in plain language.
- Before coding, inspect the project and read both `AGENTS.md` and `VELLIO_BUILD_BRIEF.md`.
- Start by proposing a clear implementation plan. Do not edit files until the owner has reviewed and approved the plan.
- Build in small, testable stages, following the priorities in the full brief.
- Do not build the entire application in one pass.
- After each stage, run the app and relevant tests. Explain what changed and how the owner can test it.
- Test edge cases, not only the happy path.
- Explain the likely cause of an error before changing code to fix it.
- If a required integration cannot be verified from official documentation, stop and report what is unknown. Do not invent an implementation.
- Keep the owner informed about external accounts, credentials, wallet funding, deployments or other manual actions they must complete.
- Do not expose secrets in code, logs, screenshots or client-side bundles.
- After a feature works and has been tested, propose a clear Git commit message and create a save point only with the owner's approval.
- Do not deploy, spend funds, submit transactions or make external account changes without the owner's explicit approval.
- Keep this file and `VELLIO_BUILD_BRIEF.md` updated when the owner approves a change to project decisions.

## First implementation stage
The first stage is project foundation, application setup, database schema and the invoice creation workflow. Keep the architecture ready for payment integrations, but do not claim those integrations are complete until they are implemented and tested.
