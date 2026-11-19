# Ha Noi Asian Food Website

Restaurant ordering website for Ha Noi Asian Food / Bar Ha Noi in Warsaw.

## Company Defaults

- Legal name: Duna Anh Thao Dao
- NIP: 1132932468
- Email: daoanhthao82@gmail.com
- Website: https://barhanoi.pl
- Lokal 2: ul. Człuchowska 37, 01-100 Warszawa, +48 579 192 223
- Lokal 3: ul. gen. Sylwestra Kaliskiego 15, 01-476 Warszawa, +48 729 556 224

## Backend Contract

- Convex owns menu data, site settings, orders, reservations, users, delivery validation cache, P24 payment state, and Dotykačka OAuth captures.
- Clerk owns identity. Admin access is allowlisted in `convex/accessControl.ts` and mirrored in `src/App.tsx` / `src/pages/AdminPage.tsx`.
- Resend is in dry-run mode by default with `RESEND_TEST_MODE=true`; no API key is required until real emails should send.
- Przelewy24 and Dotykačka are preserved as integrations. New board-priced menu items must be mapped to Dotykačka product IDs before paid online ordering can safely go live.

## Cloudflare Workers

1. `wrangler.jsonc` is configured for project `ha-noi-bar` with account `CLOUDFLARE_ACCOUNT_ID`.
2. Local deploy: `npx wrangler login`, then `npm run deploy`.
3. Live preview URL: `https://ha-noi-bar.kamihi.workers.dev`
4. Production frontend URL is set in `.env.production` as `VITE_APP_URL=https://barhanoi.pl`.
5. Both `barhanoi.pl` and `www.barhanoi.pl` are attached to the existing `barhanoi` Worker as custom domains.

## Required Keys Per New Website

1. Clerk: `VITE_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `CLERK_ISSUER_URL`, `CLERK_JWT_ISSUER_DOMAIN`.
2. Convex: `VITE_CONVEX_URL`, `VITE_CONVEX_SITE_URL`, `CONVEX_SITE_URL`, `CONVEX_DEPLOYMENT`.
3. Resend: set `RESEND_TEST_MODE=false`, then add `RESEND_API_KEY`, `RESEND_FROM_EMAIL`, `RESERVATION_ADMIN_EMAIL`.
4. Przelewy24: `P24_MERCHANT_ID`, `P24_POS_ID`, `P24_CRC_KEY`, `P24_API_KEY`, `P24_API_URL`.
5. Dotykačka: `DOTYKACKA_CLIENT_ID`, `DOTYKACKA_CLIENT_SECRET`, `DOTYKACKA_LICENSE_KEY`, `DOTYKACKA_REFRESH_TOKEN`, `DOTYKACKA_CLOUD_ID`, `DOTYKACKA_BRANCH_ID`, `DOTYKACKA_WEBHOOK_SECRET`.

## Setup Flow

1. Copy `.env.example` to `.env.local` and `convex/.env.local`.
2. Add Clerk and Convex keys first. Do not touch UI copy until auth and data reads work.
3. Run `npm install`, then `npx convex dev` to generate/sync Convex functions.
4. Run `npm run build`.
5. Run `npx convex run importMenu:upsertBarHaNoiMenu` after admin auth is configured to upsert the board-priced menu into Convex.
6. Add P24 and Dotykačka only after mapped menu parity is verified.

## Blindspots

- Admin authorization still has hardcoded email mirrors. For launch, update both frontend visibility and `convex/accessControl.ts`; backend allowlist is the security boundary.
- Delivery validation currently calculates from Lokal 2. If customers must choose pickup/delivery branch, add branch selection and validate distance against the selected branch.
- The public Google embed shows both locations as a directions map because a true multi-marker Google embed requires either a Maps Embed API key or a prebuilt Google My Maps map.
