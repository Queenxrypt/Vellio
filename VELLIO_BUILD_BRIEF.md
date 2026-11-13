# VELLIO
## One invoice. Payments without borders.

## 1. Project overview

Vellio is a cross-chain invoicing and payment reconciliation platform for freelancers and agencies receiving international payments.

A freelancer creates an invoice and shares a payment link with a client. The client can pay using a supported cryptocurrency on a supported network. Vellio routes the standard payment through the official NEAR Intents 1Click API, and the freelancer receives USDC on NEAR.

The freelancer should not need to understand bridges, swaps, gas fees or blockchain networks to receive payment. Vellio brings invoicing, payment execution, settlement verification and reconciliation into one workflow.

### Core problem

Freelancers and small agencies receiving international payments can face:
- Limited payment options across countries.
- Different currencies and blockchain networks.
- Manual conversion and transfer processes.
- Unclear payment statuses.
- Difficulty matching incoming payments to invoices.
- Poor accounting records across payment channels.

### Target users
- Independent freelancers.
- Remote professionals.
- Small agencies.
- International service providers.
- Businesses receiving stablecoin payments.

## 2. Product promise

A freelancer should be able to:
1. Create an invoice.
2. Share one payment link.
3. Receive payment in USDC on NEAR.
4. Track payment status without checking multiple block explorers.
5. Export accurate transaction records for accounting.

A client should be able to pay using a supported asset and network without needing to understand the conversion process.

## 3. Hackathon judging strategy

Target track: NEAR Hackathon 2026, Real-World Money.

| Judging criterion | Weight | Implementation target |
|---|---:|---|
| NEAR Tech Usage | 35% | Official 1Click API, NEP-141 USDC, NEP-366 relayer and PingPay integration. |
| Real, Not Simulated | 25% | Real API calls, real transactions, deployed contracts and verifiable settlement. |
| Track Fit | 20% | Practical cross-border invoicing and stablecoin payments. |
| Execution & Completeness | 20% | Working end-to-end workflow, secure configuration, error handling and reconciliation. |

The integrations must be implemented and called by the application. Imports, placeholders, README claims and mocked transactions do not count as completed integrations.

Every required integration must have:
- A real implementation.
- A clear entry point in the application.
- A real request or transaction path.
- Error handling.
- A way to inspect or verify its result.

## 4. Required NEAR integrations

### A. NEAR Intents through the official 1Click API

This is the primary payment infrastructure.

Official API base URL: `https://1click.chaindefuser.com`

Implement the actual payment lifecycle:
1. Fetch supported assets using `GET /v0/tokens`.
2. Identify supported source assets and the intended USDC destination asset.
3. Request a quote using `POST /v0/quote`.
4. Create a committed quote with `dry: false`.
5. Display the quote, expected amount and expiry.
6. Give the client the correct deposit instructions.
7. Accept or retrieve the source-chain transaction reference.
8. Notify 1Click using `/v0/deposit/submit` where appropriate.
9. Poll `GET /v0/status`.
10. Update the invoice only after successful destination settlement has been verified.

Use the actual asset IDs and decimals returned by the API. Do not invent token IDs or assume every network supports every route.

Distinguish quote previews from committed quotes. A dry quote does not create a deposit address and cannot complete a payment.

Important network constraint: the official 1Click API has no official testnet deployment. Real 1Click payment testing therefore requires small amounts on supported mainnet routes.

Documentation:
`https://docs.agentsonintents.com/integration/distribution-channels/1click-api/quickstart/introduction`

### B. NEP-141 USDC on NEAR

USDC on NEAR is the intended freelancer settlement asset.

The application must:
- Identify the correct USDC NEP-141 asset from the live supported-token list.
- Use the exact asset ID and token decimals.
- Specify the freelancer's NEAR destination address in the quote.
- Verify the actual destination amount and transaction result.
- Record the settlement reference.
- Never treat a source-chain deposit as successful destination settlement.

Do not hardcode an assumed USDC contract address. Verify the destination token against the live 1Click token list and the relevant NEAR token contract.

### C. NEP-366 meta transaction relayer

The freelancer should not need to hold NEAR to perform supported Vellio contract actions.

Implement a real relayer that:
- Receives a user-signed `DelegateAction`.
- Validates the sender and receiver.
- Checks the permitted contract and method.
- Validates nonce and expiry.
- Rejects unexpected actions.
- Submits the signed delegate action to NEAR.
- Pays transaction gas from the relayer account.
- Returns the transaction reference and execution result.

The relayer must only sponsor permitted calls to Vellio's own contract.

Potential sponsored operations:
- Create invoice.
- Update permitted invoice metadata or status.
- Record a verified payment reference.

Security requirements:
- Allowlist the Vellio contract address.
- Allowlist permitted method names.
- Set gas limits.
- Validate account IDs and signatures.
- Prevent replay.
- Reject expired delegate actions.
- Rate-limit requests.
- Protect the relayer private key.
- Never trust a client-submitted success response.

NEP-366 identity detail: when checking authorisation, the predecessor account represents the original user, while the relayer is the transaction signer. Do not authorise users using the relayer signer identity.

Specification:
`https://github.com/near/NEPs/blob/master/neps/nep-0366.md`

### D. PingPay Agent Pay facilitator

PingPay is explicitly named in the hackathon judging criteria and needs a real application use case and working integration.

The track note describes an agent paying an x402-protected resource in USDC on Base, with settlement to a NEAR recipient.

Proposed Vellio use case:
A client or business agent receives a Vellio invoice and initiates payment programmatically. The agent pays the invoice through a documented x402 payment flow using PingPay's facilitator. The payment settles to the configured NEAR recipient in the supported asset.

Expected application flow:
1. Receive the payment request.
2. Expose the appropriate x402 payment requirements.
3. Allow the agent to submit its payment.
4. Use PingPay's facilitator for the documented Base USDC to NEAR settlement flow.
5. Verify the facilitator's settlement result.
6. Match the verified payment to the correct invoice.
7. Update the invoice and reconciliation records.

Critical constraints:
- Follow PingPay's actual facilitator documentation and sponsor integration instructions.
- Confirm the supported x402 version, payment scheme, network and settlement asset.
- Do not invent PingPay endpoints, request formats, SDK methods, authentication or response fields.
- Do not mark an invoice paid based only on an x402 payment payload or HTTP success response.
- Verify actual settlement before updating invoice status.
- Keep this integration separate from standard 1Click checkout so both paths can be tested independently.

The PingPay feature should be small but real. One working agent payment against one Vellio invoice is more valuable than a broad agent dashboard without settlement.

Track reference:
`https://github.com/nearai/near-hackathon-prompt-examples/blob/main/tracks/08-realworld.md`

## 5. Payment architecture

Vellio has two distinct payment paths.

### Path A: Standard client checkout

1. Freelancer creates an invoice.
2. Vellio creates a public invoice link.
3. Client opens the link.
4. Client selects a supported source asset and network.
5. Backend requests a quote from official 1Click.
6. Client receives deposit instructions.
7. Client makes the source-chain deposit.
8. Vellio monitors the 1Click status.
9. 1Click confirms successful destination settlement.
10. Vellio verifies the destination amount and transaction reference.
11. Invoice changes to paid.
12. Payment appears in reconciliation and CSV export.

### Path B: Agent payment through PingPay

1. An agent requests invoice payment requirements.
2. Vellio exposes the supported x402 payment requirements.
3. The agent authorises payment in USDC on Base.
4. PingPay's facilitator handles the supported settlement route.
5. Vellio verifies the settlement result.
6. The payment is matched to the invoice.
7. Invoice changes to paid.
8. Payment appears in reconciliation and CSV export.

Both paths use the same invoice and reconciliation records, while retaining their individual provider and transaction details.

## 6. Invoice management

### Invoice creation

The freelancer should be able to enter:
- Invoice number.
- Client name.
- Client email.
- Invoice title.
- Description.
- Line items.
- Quantity.
- Unit price.
- Currency.
- Due date.
- Optional notes.

The application calculates subtotal and total using safe decimal arithmetic.

### Public invoice page

Each invoice receives a shareable URL. The client sees:
- Freelancer or business name.
- Invoice number.
- Description.
- Line items.
- Amount due.
- Due date.
- Payment options.
- Pay button.
- Payment progress.
- Final payment confirmation.

Hide unnecessary blockchain details from the client.

### Invoice statuses

Support:
- Draft.
- Unpaid.
- Pending.
- Paid.
- Overdue.
- Cancelled.
- Partially paid, only if properly implemented and verified.

A payment attempt has its own status, separate from the invoice status.

### Payment-attempt statuses

Support actual provider states where applicable:
- `PENDING_DEPOSIT`
- `KNOWN_DEPOSIT_TX`
- `PROCESSING`
- `SUCCESS`
- `INCOMPLETE_DEPOSIT`
- `REFUNDED`
- `FAILED`

Never mark an invoice paid simply because a deposit transaction was detected. Only verified successful destination settlement can trigger paid status.

## 7. Payment verification and reconciliation

The backend owns payment status. The frontend must never be allowed to mark an invoice paid directly.

For every payment attempt, store:
- Invoice ID.
- Payment attempt ID.
- Provider.
- Network environment.
- Source asset.
- Source network.
- Source amount.
- Destination asset.
- Destination network.
- Expected destination amount.
- Actual destination amount.
- Deposit address or payment destination reference.
- Source transaction hash.
- Destination transaction hash.
- Provider status.
- Created timestamp.
- Last status check.
- Settlement timestamp.
- Refund information, where available.

### Financial correctness

- Use integer base units or precise decimal arithmetic.
- Never use floating-point arithmetic for payment amounts.
- Respect token decimals.
- Store quoted amount separately from actual settled amount.
- Use provider-reported destination settlement amounts for accounting.
- Never infer settlement from a source transaction alone.
- Process status updates idempotently.
- Preserve payment history.
- Never overwrite a successful payment record with a later polling response that does not represent a valid state transition.

Do not promise that every failed or incomplete deposit will be refunded automatically. Display the actual provider status and refund result.

## 8. CSV export

The freelancer must be able to export an accountant-friendly CSV.

Include:
- Invoice number.
- Invoice ID.
- Client.
- Invoice date.
- Due date.
- Invoice amount.
- Invoice currency.
- Payment provider.
- Payment attempt ID.
- Source asset.
- Source network.
- Source amount.
- Destination asset.
- Destination network.
- Actual settled amount.
- Payment status.
- Source transaction hash.
- Destination transaction hash.
- Payment date.

Use clear column names, ISO-formatted dates and consistent decimal formatting. Distinguish test transactions from production transactions.

## 9. Application architecture

Recommended stack:

Frontend:
- Next.js.
- TypeScript.
- Tailwind CSS.
- shadcn/ui.

Backend:
- Next.js server routes or a dedicated Node.js service.
- TypeScript.
- PostgreSQL.
- Supabase for database and authentication.

Blockchain:
- NEAR smart contract written in Rust using `near-sdk`.
- NEP-366 delegate actions.
- NEAR RPC for contract and transaction verification.

Payment providers:
- Official NEAR Intents 1Click API.
- PingPay Agent Pay facilitator.
- Tidepool only as an optional development tool, never as a replacement for official 1Click.

Payment provider calls and secret handling must happen on the server.

The frontend must never contain:
- 1Click credentials.
- PingPay secrets.
- Relayer private keys.
- Database service-role keys.
- Other privileged credentials.

## 10. Database structure

Use a relational model. The following is a starting schema and may be refined after implementation planning.

### `users`
- `id`
- `email`
- `name`
- `business_name`
- `near_account_id`
- `created_at`

### `invoices`
- `id`
- `user_id`
- `invoice_number`
- `client_name`
- `client_email`
- `title`
- `description`
- `currency`
- `subtotal`
- `total`
- `due_date`
- `status`
- `public_token`
- `created_at`
- `updated_at`

### `invoice_items`
- `id`
- `invoice_id`
- `description`
- `quantity`
- `unit_price`
- `total`

### `payment_attempts`
- `id`
- `invoice_id`
- `provider`
- `environment`
- `source_asset`
- `source_network`
- `source_amount`
- `destination_asset`
- `destination_network`
- `expected_destination_amount`
- `actual_destination_amount`
- `deposit_address`
- `provider_reference`
- `status`
- `created_at`
- `updated_at`
- `settled_at`

### `payment_transactions`
- `id`
- `payment_attempt_id`
- `network`
- `transaction_hash`
- `transaction_type`
- `created_at`

### `payment_status_events`
- `id`
- `payment_attempt_id`
- `previous_status`
- `new_status`
- `provider_payload_reference`
- `created_at`

### `audit_logs`
- `id`
- `user_id`
- `action`
- `resource_type`
- `resource_id`
- `created_at`

Never store full private payment payloads or secret keys in logs. Enforce ownership and access controls at the database and server layers.

## 11. Smart contract scope

Keep the NEAR contract deliberately small. It should support invoice actions needed by the application and provide an on-chain record or reference for relevant operations.

Possible methods:
- `create_invoice`
- `update_invoice`
- `record_payment_reference`

Do not store client emails, private invoice details or unnecessary business information on-chain.

The database remains the primary source for detailed invoice records.

The contract should:
- Validate caller permissions.
- Prevent unauthorised invoice updates.
- Reject invalid state transitions.
- Use predecessor identity correctly.
- Keep stored data minimal.
- Emit useful events or maintain queryable state.

Do not implement token custody or a custom bridge. NEAR Intents handles cross-chain payment routing.

## 12. Network and environment strategy

| Component | Development/testing | Real payment |
|---|---|---|
| Web application | Local | Hosted deployment |
| Database | Development database | Production database |
| Vellio contract | NEAR testnet | NEAR mainnet |
| NEP-366 relayer | NEAR testnet | NEAR mainnet |
| Official 1Click | Mainnet only | Mainnet |
| PingPay | Environment confirmed by its documentation | Supported production environment |
| USDC settlement | Test asset where supported | Actual USDC on NEAR |

Official 1Click does not provide an official testnet.

Tidepool may be used as an independent community testnet sandbox. It must be labelled accurately and must never be represented as official 1Click testnet infrastructure.

### Strict environment separation

Every payment attempt must store:
- Provider.
- Environment.
- Network.
- Asset.
- Transaction references.

Testnet records must never update production invoices or be included in production revenue totals. The UI and README must clearly distinguish test payments from real payments.

## 13. Demo strategy

The strongest demonstration shows actual execution.

Preferred standard payment demo:
1. Freelancer creates an invoice.
2. Vellio creates a shareable invoice link.
3. Client opens the invoice.
4. Client selects a supported source asset.
5. Official 1Click returns a real quote.
6. Client completes a small real payment.
7. Vellio tracks the actual payment status.
8. Destination USDC settlement is verified on NEAR.
9. Invoice changes to paid.
10. Freelancer opens reconciliation and exports the CSV.

A separate short demonstration should show the NEP-366 relayer sponsoring a permitted invoice-related contract action.

The PingPay demonstration should show an agent paying an x402-protected Vellio invoice or payment endpoint using the documented Base USDC to NEAR settlement path.

If funds or provider access are unavailable, provide a clearly labelled dry-run mode for exploration. Never present it as a completed payment.

## 14. Design system

Product name: Vellio

Tagline: One invoice. Payments without borders.

Core colours:
- Warm Cream: `#F5F0E6`
- Deep Ink: `#17172A`
- Tangerine: `#FF642D`
- Violet: `#8057EF`
- Warm Grey: `#D8D1C5`

Orange and violet are the core brand accents. Green and amber may be used for functional payment statuses, not as additional brand colours.

Typography:
- Space Grotesk for display.
- DM Sans for interface text.

Visual direction:
- Warm cream backgrounds.
- Strategic partial grid patterns.
- Bold, clean outlines.
- Simple geometric illustrations.
- Generous negative space.
- Balanced editorial layouts.
- Playful but professional appearance.

Grid patterns should be subtle and selective. Do not place decorative grids behind invoice data or checkout information.

Avoid generic crypto imagery, coin illustrations and blockchain symbols.

### Application screens
1. Dashboard.
2. Invoice list.
3. Invoice details.
4. Create invoice.
5. Public invoice page.
6. Payment selection.
7. Payment progress.
8. Payment confirmation.
9. Reconciliation.
10. CSV export.

Responsive desktop and mobile layouts are required.

## 15. Security and reliability

- Keep all secrets server-side.
- Validate all user inputs.
- Enforce invoice ownership.
- Protect public invoice tokens.
- Verify payment status with the provider.
- Validate transaction and settlement references.
- Use idempotency keys where supported.
- Make payment status transitions safe and predictable.
- Rate-limit payment and relayer endpoints.
- Validate all relayer delegate actions.
- Apply strict relayer contract and method allowlists.
- Do not expose internal errors or secrets to users.
- Keep an audit trail of payment status changes.
- Handle API timeouts and provider outages.
- Never use mock payment success in the production payment path.

## 16. Testing plan

### Application tests
- Invoice creation.
- Invoice editing.
- Public invoice access.
- Line-item calculations.
- Invoice status transitions.
- CSV export.

### 1Click integration tests
- Fetch live token list.
- Validate destination asset.
- Request dry quote.
- Request committed quote.
- Validate quote expiry.
- Handle deposit detection.
- Poll payment status.
- Handle success, failure and incomplete deposits.
- Verify destination transaction and amount.

### Relayer tests
- Valid signed delegate action.
- Invalid signature.
- Expired delegate action.
- Replayed nonce.
- Unapproved contract.
- Unapproved method.
- Gas limit enforcement.
- Successful testnet transaction.

### PingPay tests
- Verify supported facilitator capabilities.
- Generate or retrieve valid x402 payment requirements.
- Test a supported Base USDC payment.
- Verify facilitator settlement.
- Verify NEAR recipient and destination asset.
- Match settlement to the correct invoice.
- Reject invalid or incomplete payment responses.

### End-to-end tests
- Create invoice and pay using 1Click.
- Verify settlement and update invoice.
- Export payment record.
- Create invoice and complete the PingPay agent payment path.
- Verify the resulting reconciliation record.

## 17. Environment variables

Use `.env.local` locally and a secure deployment secret manager.

Suggested configuration:

```env
NEXT_PUBLIC_APP_URL=

DATABASE_URL=
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

NEAR_NETWORK=
NEAR_RPC_URL=
NEAR_CONTRACT_ID=
NEAR_RELAYER_ACCOUNT_ID=
NEAR_RELAYER_PRIVATE_KEY=

ONECLICK_BASE_URL=https://1click.chaindefuser.com
ONECLICK_JWT=

PINGPAY_BASE_URL=
PINGPAY_API_KEY=

PAYMENT_ENVIRONMENT=
```

The exact PingPay variables and authentication method must follow its actual integration documentation. Do not assume a generic API key is required.

Never commit `.env.local`. Include `.env.example` with empty values.

## 18. Manual setup required

These steps require the project owner:
1. Create a Supabase project.
2. Configure the database and authentication.
3. Create a NEAR testnet account for contract and relayer testing.
4. Deploy the Vellio contract to testnet.
5. Fund the testnet relayer account.
6. Confirm the official 1Click API token list and destination USDC asset.
7. Obtain 1Click partner credentials if available. The integration can initially use the unauthenticated API, with its documented fee implications.
8. Obtain PingPay integration access and confirm the actual facilitator requirements.
9. Create a funded, controlled wallet for a small mainnet payment test.
10. Configure deployment secrets.
11. Deploy the web application.
12. Complete a real payment and record verifiable transaction references.

The application should be structured so development can continue while credentials are being arranged, without pretending those integrations have already been tested.

## 19. Hackathon submission

The repository must be public.

Include:
- Clear README.
- Product overview.
- Architecture.
- Required environment variables.
- Setup instructions.
- Database setup.
- Contract deployment instructions.
- Relayer instructions.
- 1Click integration details.
- PingPay integration details.
- Test and production environment distinctions.
- Payment verification explanation.
- CSV example.
- Known limitations.
- Demo video or screenshots.
- Real transaction references where available.

The README must accurately describe what works. Do not claim an integration is complete until the application actually calls it and the flow has been tested.

The demo post must:
- Mention NEAR HACKATHON.
- Tag `@NEARProtocol`.
- Include `#NEARHackathon`.
- Include a demo video or screenshots.
- Link to the public repository.

## 20. Scope priorities

With the deadline close, prioritise a working, demonstrable product over optional features.

### Priority 1: Core application
- Application foundation.
- Authentication.
- Dashboard.
- Invoice creation.
- Public invoice link.
- Invoice details.

### Priority 2: Official 1Click payment
- Live token discovery.
- Quote creation.
- Deposit instructions.
- Status monitoring.
- Settlement verification.
- Invoice status update.

### Priority 3: NEP-366 relayer
- Contract deployment.
- Signed delegate actions.
- Restricted relayer.
- Gasless invoice action.

### Priority 4: PingPay
- Confirm actual API and facilitator requirements.
- Implement one working agent payment path.
- Verify Base USDC to NEAR settlement.
- Reconcile payment with invoice.

### Priority 5: Reconciliation and finish
- Payment history.
- CSV export.
- Error handling.
- Responsive design.
- README.
- Demo recording.
- Submission.

Do not spend remaining build time on optional dashboards, advanced analytics, elaborate settings or decorative features while a required payment integration remains unimplemented.

## 21. Definition of done

Vellio is ready when:
- A user can create an invoice.
- A client can open the public invoice link.
- The official 1Click API is called by the application.
- A real supported payment can be initiated.
- Destination USDC settlement is verified.
- The invoice changes to paid only after verified settlement.
- A real NEP-366 relayer sponsors a permitted Vellio contract call.
- PingPay's facilitator is integrated into a working x402 payment path.
- Payment records are reconciled correctly.
- CSV export works.
- The repository contains no secrets.
- The README explains the actual implementation.
- The demo shows real functionality without disguising simulations as payments.

## 22. First Claude Code task

We are building Vellio, a cross-chain invoicing and payment reconciliation platform for freelancers and agencies.

Tagline: One invoice. Payments without borders.

The project is being built for the NEAR Hackathon 2026 Real-World Money track.

The required integrations are:
1. Official NEAR Intents 1Click API.
2. NEP-141 USDC on NEAR as the destination asset.
3. NEP-366 meta transaction relayer for gasless freelancer actions.
4. PingPay Agent Pay facilitator for an x402 payment path using USDC on Base and settlement to a NEAR recipient.

Important network constraint:
Official NEAR Intents 1Click does not have an official testnet deployment. The real 1Click payment integration must use its official mainnet API. NEAR contract and relayer development can use testnet. Tidepool may be considered as an optional independent testnet sandbox, but it must never be presented as official 1Click infrastructure or replace the official 1Click integration.

Do not mock or simulate the production payment path. Do not create fake transaction hashes, fake balances or artificial payment confirmations. A dry-run mode may exist for quote previews and safe exploration, but it must be clearly separated from real payments.

Before implementation:
1. Inspect the project directory.
2. Identify the framework, existing files and dependencies.
3. Report what is already present.
4. Propose a clear folder structure.
5. Identify external credentials and manual setup required.
6. Identify the exact official documentation and integration requirements for 1Click, NEP-366 and PingPay.
7. Flag anything that cannot be verified from available documentation.

Do not invent undocumented API endpoints or request formats. Do not silently substitute an unofficial service for an official integration.

Build in small, testable stages. After each stage, verify that the code runs before proceeding.

The first implementation stage is the application foundation, database schema and invoice creation workflow. Keep the architecture ready for the real payment integrations, but do not claim those integrations are complete until they are implemented and tested.

Follow `AGENTS.md` and this full technical build brief as the project source of truth. Start in Plan mode. Read both files, inspect the project and propose the implementation plan. Do not edit files until the project owner approves the plan.
