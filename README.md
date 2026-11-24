# Cash DNR

Recovered application source. Start with [HOSTING.md](HOSTING.md) for GitHub-backed deployment and environment setup. GitHub Pages cannot execute the server APIs.

## Previous implementation notes

# Cash DNR — milestone 01
Responsive account foundation based on Cash_DNR_MVP_Development_Plan.docx.

## Current implementation
- Landing page, registration and login UI, individual and merchant dashboards.
- Server-side Supabase Auth HTTP integration (passwords never stored in application storage).
- HttpOnly session cookie; same-origin mutation checks; Supabase rate limits apply.
- Unique UUID DNR references created by a database trigger; own-profile read policy.
- No browser profile writes; no self-selected administrator access.
- Explicit synthetic demo mode when Supabase is unconfigured. Demo is memory-only and never calls real registration.
- Wallet and lookup explain their upcoming milestone. No banknotes, transfers or financial balances are fabricated.

## Run locally
Use Node.js 22.13 or newer. Install with `npm install`, then `npm run dev`.
This hosted build uses the Sites Vinext adapter for Next.js/React and Tailwind.

## Connect Supabase
1. Create a separate development Supabase project.
2. Run `supabase/001_accounts.sql`, then `supabase/002_profile_and_audit.sql` in its SQL editor.
3. Enable email/password authentication and retain email confirmation.
4. Copy `.env.example` to `.env.local`; set SUPABASE_URL and SUPABASE_ANON_KEY. Do not use a service-role key.
5. Configure the same variable names for the hosted runtime. Set the Supabase Site URL to the deployment URL and allow the local development URL as needed.
6. Restart locally or deploy. Register a synthetic individual, confirm email, log in and inspect the unique DNR account.
7. Repeat with a merchant. Account roles come from the database, not editable auth metadata.

## Acceptance checks still requiring a connected backend
- Complete email confirmation, login, logout, expired-session handling and merchant registration.
- Assert one account cannot read another profile and neither role can insert/update profiles or promote itself.
- Attempt direct signup with role admin: trigger rejects it.
- Confirm duplicate emails and invalid requests are handled without revealing account existence.
- Review Supabase password policy, authentication rate limits and deployment-origin cookie behavior.

## Limits
Sessions renew using a secure refresh-token cookie. Password recovery remains future work. Supabase is not provisioned by this source repository. No real ID numbers, financial transfers, bank integrations or physical cash authentication. Browser visual QA was unavailable during initial delivery. Banknote workflows are intentionally pending until the first connected account journey and role checks pass.

## Presentation update
Customer screens omit development roadmaps, milestone labels and prototype copy. Synthetic sessions retain a demo label. Added profile editing via an own-account-only database function and immutable account events. Roles and account references cannot be changed through that function. Session responses are not cached; tokens remain HttpOnly. Apply both database migrations before enabling registration.

## Banknote registry
Apply `supabase/003_banknote_registry.sql` once after account setup. Then run `supabase/004_registry_permission_test.sql`: its fixtures, note declarations and temporary administrator role are enclosed in a transaction and rolled back. This test requires an individual and merchant account.

The wallet lists declarations and approved records. Registration accepts only synthetic TEST- references and R10/R20/R50/R100/R200. Database uniqueness prevents duplicates; duplicate attempts generate an administrator flag and never alter the existing holder. Request keys make registration retries idempotent. Pending/rejected declarations stay out of general lookup. Lookup returns status, denomination and a short holder reference; no personal identity or profile data is exposed.

Review is administrator-only, enforced inside the database function. Approval locks the note and atomically assigns the holder, writes history and audit. Rejection requires a reason and assigns no holder. Repeated review is rejected. Ordinary users have no direct write grants on registry, history, flags or audit tables. Transfers, payments and lost-note reporting are separate upcoming workflows.

Admin provisioning is a trusted database-operator action. Create a dedicated account through the app first, then promote that exact profile UUID with SQL after verification. Never use user-editable metadata for privileges. The Admin navigation appears after the next login/refresh.

Initial verification: TypeScript and input-validation checks pass. Live SQL tests must be run after applying the migration; browser visual QA remains unavailable. Lists are bounded (500 notes/history, 200 audit records, 100 flags); pagination will be needed for larger demonstrations.

## Banknote transfers
Apply `supabase/005_transfers.sql` after the registry migration, then run `supabase/006_transfer_permission_test.sql` (rollback-only). Wallet includes incoming/outgoing requests and request, accept, decline, cancel actions. Recipient DNR IDs must belong to individual or merchant profiles. Only active notes held by the sender qualify. A single pending request per note preserves the sender as holder until acceptance. Acceptance atomically updates the holder, transfer status, ownership history and audit. Repeated matching requests/actions are idempotent. These are recorded physical handovers, not electronic payments or currency authentication.

Validation: TypeScript and PostgreSQL-compatible PGlite rollback tests cover unauthorized requests/actions, direct-write denial, participant privacy, self-transfer, duplicate pending requests, repeat acceptance history/audit, cancellation, decline and restricted-note rejection. Production database migration and two-account browser handover remain manual verification steps.

## Transaction identities and email notifications
Apply `supabase/007_transaction_notifications.sql` after transfers. Transaction lists and participant ownership history include display names and complete DNR IDs, scoped to the participants (or admin history). Public reference lookup retains minimal holder information. Wallet opens on currently held notes; past declarations are labelled as belonging to another holder. The sidebar retains the single log-out control.

Future transfer requests and accepted/declined/cancelled outcomes queue private, separate sender/recipient emails atomically with the transaction. There is no backfill of historical notifications. Configure and deploy `supabase/functions/transaction-mail` using its README, then enable the Vault-based schedule in `008_schedule_transaction_mail.sql`. A Resend API key, verified sender address and worker dispatch secret are required; delivery is not enabled by the app deployment alone. Keep service credentials within Supabase Edge Functions. Queue state says sent only after provider acknowledgement, not inbox delivery.

Validation: TypeScript app and worker checks; `node --experimental-strip-types --test tests/transaction-mail.test.mjs` verifies separate messages, provider keys, error/timeout handling and worker authorization. PGlite rollback tests `006` and `009` passed. Browser/inbox verification remains manual after migration and email-service configuration. Merchant payments, lost/stolen reporting, transfer expiry, broader admin management and final mobile/concurrency checks remain delivery work.

## Merchant payments and reconciliation
Apply `010_merchant_payments.sql` after `007`, then run rollback test `011_payment_permission_test.sql`. Merchants request a cash purchase using a customer DNR ID, sale reference, whole-rand amount and 1–100 unique registered TEST references. Their active notes must sum to the exact amount. Shared note reservations prevent overlap with other payments and P2P transfers. The customer approves or declines; the requesting merchant can cancel. Acceptance atomically changes every note's holder and writes payment-linked ownership/audit events. Repeat requests and matching decisions are idempotent.

Payments shows requests and receipts to their participants. Merchant Reconciliation includes accepted purchase totals, pending requests, declined/cancelled counts and an unsaved physical-count comparison. Accepted totals use completion time; pending totals use request time. Date windows use the visitor's local day boundaries. Lists show the latest 200 matching records, while summary totals cover the entire selected range. These totals cover recorded Cash DNR sales, not the shop's full cash drawer, banking or electronic settlement.

Coins, change, refunds, offline handover/disputes, inventory/POS integration and expiry are not implemented. Email delivery remains paused at the user's request; the transfer queue is retained but no email provider/worker is enabled. Browser payment testing requires migration and test notes held by the customer. TypeScript, validator tests and PGlite multi-note permission/atomicity/reservation/reconciliation tests passed. Live two-account verification remains pending.

## Phone camera scanning
Scan navigation provides rear-camera capture, photo selection (JPG/PNG/WebP, max 12 MB), rotation, vertical crop controls, local OCR, editable reference/denomination review and a synthetic practice image. The pinned Tesseract.js worker/core/model files are served from public/ocr; no captured image is uploaded to the server or persisted to an account. Camera tracks stop after capture, when the page becomes hidden, and when the scanner unmounts. OCR has a 60-second timeout and cleans up the worker on exit/failure.

Review confirmation is required before copying or using a reading. TEST references can prefill lookup, the registration form, or a merchant payment request. Those actions still require their normal submit/approval steps; OCR never changes ownership. Real-note serial candidates can be reviewed/copied but cannot enter the current TEST-only registry or create funds. Candidate matching is conservative and is not a complete ZAR-series serial parser. R/ZAR text is used for denomination suggestions; when absent or ambiguous, the user must select it. The OCR engine confidence is not an authenticity score.

`node --experimental-strip-types --test tests/scan.test.mjs` passed, including actual OCR of a synthetic fixture using the bundled language model. TypeScript passed. Browser camera permissions, private-site worker loading, iOS/Android device performance and real-note OCR accuracy need live device testing; no real-note accuracy claim is made. Passkey approval is the next development step.

## Passkey transaction approval
Apply `012_passkey_approval.sql` after merchant payments, then deploy `supabase/functions/passkey-approval/index.ts` in Supabase using its README. The browser does not receive a service key. The worker validates the account token, then verifies WebAuthn signatures against the pinned Cash DNR origin/RP ID and requires device user verification. Private database tables contain public credentials, counters, one-use challenges and transaction-bound approvals. SQL wrappers block direct-RPC bypass for enrolled accounts and atomically consume approval receipts before transaction completion.

Account > Transaction approval offers first-passkey enrollment. One credential per account is currently supported. Existing session login is retained; unenrolled accounts continue session-authorized approvals. Enrolled accounts need a fresh passkey for payment acceptance and incoming transfer acceptance. Decline/cancel and outgoing request initiation remain session-authorized. Passkey sign-in, additional keys, removal and recovery require further implementation before a public rollout. No weak automatic fallback disables approval security.

Validation: valid synthetic registration/assertion signatures passed; wrong origin/challenge, missing user verification, stale counters and modified signatures were rejected. PostgreSQL-compatible rollback tests cover private access, trusted issuance, blocked internal RPC, transaction binding, expiry and replay protection, along with prior payment/transfer regression tests. TypeScript app and worker checks passed. Actual Supabase worker deployment and phone passkey enrollment/approval still require live verification.

## Lost/stolen reports and recovery
Apply `014_lost_stolen.sql` after passkeys, then run rollback-only test `015_incident_permission_test.sql`. Current holders can report an active note as lost/stolen with a reason. Reporting keeps ownership unchanged and immediately blocks new requests and acceptance of existing requests through the transaction functions' active-note checks. Pending requests stay recorded and reserved until declined/cancelled; they are not silently deleted or automatically completed after a report. Transfer and payment screens expose a restricted note status and disable acceptance while blocked.

Note details include reporting, incident history and holder recovery requests. Admin has a report queue and can restore or reject a requested recovery with a reason. Restoration reactivates the note with the same holder; rejection keeps it restricted. Each report/review writes private incident events and banknote audit entries. Direct browser edits are denied, report reasons are limited to reporter/admin, and public lookup returns only existing minimal holder data plus the restricted status.

Restrictions apply within Cash DNR and do not disable physical currency or prove authenticity. Reporting/recovery request/admin review use authenticated session and database role checks; passkeys remain required for enrolled-account transfer/payment acceptance. Local PostgreSQL-compatible rollback tests passed for holder access, privacy, report replay, restricted lookup, pending-transfer/payment blocks, atomic failure with no proof consumption, recovery rejection/restoration and ownership invariance. Existing transaction/passkey regression tests and TypeScript passed. Browser verification and production database migration remain pending.
