# Hosting Cash DNR from GitHub

This is the recovered Cash DNR application, based on Sites source commit 751f8bcd66faa610264e4b95bbecc4e2e6a0fa28. It includes the existing application and Supabase migrations, with standard Next.js commands for a Node host.

GitHub stores the code. GitHub Pages cannot execute the API routes that protect sessions and transaction approvals. Deploy this repository to a Node-capable host.

## Node deployment

1. Use Node 22.13 or later. Install dependencies with `npm install`.
2. Set server environment variables `SUPABASE_URL` and `SUPABASE_ANON_KEY` using your existing Supabase project. Never commit keys or use a service-role key.
3. Build: `npm run build`. Start: `npm start -- --port $PORT` (or set the host port). HTTPS is required for secure cookies and camera/passkey access.
4. Keep the existing Supabase database. Do not blindly rerun migrations against production. Migrations and rollback tests are supplied for review/new environments.
5. Configure Supabase authentication redirect URLs and the passkey approval function's allowed origins/RP ID for the new hostname; existing passkeys may need re-enrollment for the new origin. See `supabase/functions/passkey-approval/README.md`.
6. Verify login, account separation, camera permission, OCR, scan-to-send, recipient acceptance, merchant payments, reports/recovery and passkey approval on the deployed HTTPS origin before using live records.

OCR binaries are reproduced locally by `scripts/prepare-ocr.mjs` from the installed packages. Real banknote photo fixtures are excluded from this import; the two photo-dependent OCR tests require those original local fixtures. Unit tests on the recovered source passed before import; live database and deployed-origin verification remain outstanding. The reported Invalid Transaction failure is not claimed fixed by this migration.
