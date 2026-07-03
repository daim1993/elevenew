# Security model

## Accounts & sessions
- **Studio-issued accounts only.** Public self-registration is closed; accounts are created in the CMS → Accounts desk. (Sole exception: a fresh, empty server accepts its first sign-up as the admin, so a new install can bootstrap.)
- Passwords are salted + **scrypt-hashed**; never stored or logged in plaintext. Comparisons are **timing-safe** (`crypto.timingSafeEqual`).
- Auth tokens are HMAC-SHA256-signed, expire after 30 days, and carry a **token version**: changing an account's password instantly signs out every existing session for it, everywhere. Deleting an account kills its access immediately.
- Every user's data is isolated: builder projects are scoped to `ownerId` on every query; each client portal lives in its **own database file** (`server/portals/<id>.json`) — no query can cross accounts.

## Attack mitigations
- **Brute force:** both sign-in endpoints (client login + studio master password) lock an IP+identity pair for 10 minutes after 8 failures; failures are written to the audit log with IP. General API traffic is rate-limited per IP (240 req/min).
- **XSS:** all user-supplied text is HTML-escaped before rendering; uploaded "images" are restricted to `data:image` URLs with quote/angle-bracket stripping; **Content-Security-Policy** locks scripts/styles to self + Google Fonts + the two CDNs, with `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'self'`.
- **Headers:** nosniff, X-Frame-Options SAMEORIGIN, Referrer-Policy, Permissions-Policy, Cross-Origin-Opener-Policy, and HSTS when served over HTTPS.
- **CORS:** open for local development; set `ORIGINS` (comma-separated) in production to allow-list exactly which sites may call the API.
- **Path traversal:** portal file names are sanitised to `[A-Za-z0-9_-]` before touching the filesystem.
- **Payload abuse:** JSON bodies capped (30 MB global, 24 MB per portal section), all string fields length-capped server-side.

## Production guard
The server **refuses to start** in production (`NODE_ENV=production` or Render) while `SECRET` or `ADMIN_PASSWORD` are still defaults. Set both, plus `ORIGINS`, before going live. Serve over HTTPS only.

## Data & backups
- `server/db.json` (accounts, content, builder projects), `server/portals/*.json` (per-client portals), `server/events.log` (audit trail) — all git-ignored; rolling backups every 6 h in `server/backups/` (last 14 kept).
- Client portal writes are atomic (tmp file + rename), so a crash can't corrupt a portal.

## Still recommended before real-client scale
Email verification + self-service password reset, uploads in object storage instead of base64, dependency scans (`npm audit`) in CI, and an external penetration test. Report issues to creative@sentientbyelysian.com.
