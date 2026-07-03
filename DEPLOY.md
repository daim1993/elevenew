# Launch guide — elevé.agency, free hosting

Total cost: **$4.63 first year** (domain only; ~$25.23/yr renewal). Hosting: $0.

What's already prepared in this folder:

- `server/server.js` — now saves all data (accounts, projects, content, portals) to a free Neon Postgres database when `DATABASE_URL` is set. Local dev is unchanged (still uses `server/db.json`).
- `render.yaml` — one-click Render blueprint, free plan.
- `eleve-deploy.zip` — clean copy of the site for upload (no dev data, no node_modules).

Do the steps in order. Where a step says "I can do this with you", ask Claude to walk you through it in the browser.

## 1. GitHub (free) — host the code

1. Go to github.com → Sign up (free) → verify email.
2. Top-right **+** → **New repository** → name: `eleve-site` → **Private** → Create.
3. Get the files in. Easiest reliable way: install **GitHub Desktop** (desktop.github.com) → File → Clone repository → `eleve-site` → copy everything from `eleve-deploy.zip` (unzipped) into the cloned folder → in GitHub Desktop write a summary like "launch" → **Commit to main** → **Push origin**.
   (Alternative without installing anything: on the repo page click "uploading an existing file" and drag the unzipped folders in a few batches — GitHub's web upload caps ~100 files per batch.)

## 2. Neon (free) — the database

1. Go to neon.tech → Sign up (free, no card).
2. Create a project named `eleve` — pick the region closest to your visitors (e.g. Frankfurt for Europe).
3. On the project dashboard, copy the **connection string** (starts `postgresql://...`, ends `?sslmode=require`). Keep it handy for step 3.

## 3. Render (free) — the server

1. Go to render.com → Sign up **with GitHub** (authorizes repo access in one step).
2. **New +** → **Blueprint** → select `eleve-site`. Render reads `render.yaml` automatically.
3. When prompted for **DATABASE_URL**, paste the Neon connection string. (ADMIN_PASSWORD and SECRET are auto-generated.)
4. **Apply/Deploy** → wait for **Live** → your site is now at `https://eleve-XXXX.onrender.com`. Open it and check the pages work. The logs should say `Postgres persistence: on`.
5. **Create the first account immediately** — the first registered account becomes the studio admin, then signups close. In PowerShell:

   ```powershell
   Invoke-RestMethod -Method Post -Uri "https://YOUR-APP.onrender.com/api/auth/register" -ContentType "application/json" -Body '{"email":"creative@sentientbyelysian.com","password":"CHOOSE-A-STRONG-PASSWORD"}'
   ```

6. CMS password: Render → your service → **Environment** → reveal `ADMIN_PASSWORD` (this is the master password `cms.html` asks for).

## 4. Porkbun (~$4.63) — the domain

1. Go to porkbun.com → search **elevé.agency** (it may display as `xn--elev-epa.agency` — that's the same domain, just encoded) → register 1 year (~$4.63; WHOIS privacy is free). Turn OFF auto-renew add-ons you don't want.
2. In Render: your service → **Settings → Custom Domains → Add** `xn--elev-epa.agency` and `www.xn--elev-epa.agency`. Render shows you the exact DNS records to create.
3. In Porkbun: **Domain → DNS records** → delete the parked defaults → add the records Render showed (typically an **ALIAS/A** record for the root and a **CNAME** for `www`, pointing at `YOUR-APP.onrender.com`). Porkbun supports ALIAS on the root.
4. Wait a few minutes — Render verifies and issues free SSL automatically. `https://elevé.agency` is live.

## 5. UptimeRobot (free) — keep it awake

Render's free tier sleeps after 15 min idle (first visitor then waits ~1 min).

1. uptimerobot.com → free account → **New monitor** → type HTTP(s) → URL `https://YOUR-APP.onrender.com/healthz` → interval 5 minutes.
2. That's it — the free 750 h/month covers running 24/7.

## Afterwards

- Publish content: `elevé.agency/cms.html` (master password from step 3.6).
- Local development: unchanged — `cd server && npm start`.
- If traffic grows: Render → upgrade to Starter ($7/mo) removes sleeping entirely; your data is already safe in Neon either way.
- Free-tier limits to know: Neon 0.5 GB storage, Render 100 GB bandwidth/month.

## Caveats

- **eleve.agency (no accent) belongs to someone else** — a live Squarespace site. Always share your URL with the é (or as `xn--elev-epa.agency`). Consider registering a typeable alias later (e.g. eleve.studio / elevedesign.agency) and pointing it at the same site.
- The raw admin event feed (`events.log`) resets on redeploys; the analytics counters and everything else persist in Neon.
- In some apps (email clients, old browsers) the accented domain displays as `xn--elev-epa.agency`. This is normal IDN behavior, not a broken link.
