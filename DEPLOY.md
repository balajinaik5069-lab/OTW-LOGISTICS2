# Deploy: Vercel (website + portals) and Render (backend)

```
OTW-Logistics/
├─ frontend/              ← VERCEL  (Root Directory = frontend)
│  ├─ index.html, services.html, ...     public website (served at /)
│  ├─ dashboard/                         admin panel      (/Admin/)
│  ├─ vendor/                            vendor portal    (/vendor/)
│  ├─ franchise/                         franchise portal (/franchise/)
│  └─ vercel.json                        sends /api/* to Render  ← edit the Render address in it
├─ backend/               ← RENDER  (Root Directory = backend)   API, database, emails
│  ├─ server.js, *.js, data/, shared/
│  └─ .env.production.example            every setting Render needs
├─ render.yaml            Render blueprint (backend + disk + settings list)
├─ deploy/vps/            optional: everything on ONE server instead (Docker + HTTPS)
├─ docs/                  change notes and guides
└─ START-WINDOWS.bat, start-mac-linux.sh   run everything on your own computer
```

## Why `/api` goes through Vercel
Sign-in uses a cookie that browsers only send to the **same address** as the website. If the pages called the Render
address directly, admin / vendor / franchise sign-in would silently fail. So `frontend/vercel.json` makes Vercel pass
every `/api/...` request to Render. Visitors only ever see your Vercel / custom domain. No code change is needed in the pages.

## What you need first
- **Render paid plan (Starter, about $7/month) + a 1 GB disk.** Free Render services block Gmail/SMTP (so **no application
  emails**) and have no disk (the database is erased on every deploy). Do not use the free plan for this app.
- A Vercel account (free Hobby plan is enough), a GitHub repo containing this whole folder.
- A Gmail **App Password** (Google Account > Security > 2-Step Verification on > App passwords). 16 letters.

## Step 1: Render (backend)
1. Render > **New > Blueprint** > pick your repo (it reads `render.yaml`). Or create a **Web Service** by hand:
   Root Directory `backend`, Build `npm ci --omit=dev`, Start `node server.js`, plan **Starter**, add a **Disk** mounted at `/data`.
2. Fill in the settings Render asks for (use placeholders for now, you fix the three address ones in Step 3):
   `GMAIL_USER`, `GMAIL_APP_PASSWORD`, `SUPPORT_EMAIL`, `ADMIN_EMAIL` (optional), and `PUBLIC_URL`, `ALLOWED_ORIGIN`, `ALLOWED_HOSTS`.
3. After it deploys, note the address, e.g. `https://otw-logistics-api.onrender.com`. Opening it shows
   `{"ok":true,"service":"On The Way Logistics API"...}`: that is correct, the API has no web page.
4. **Logs** tab: copy the 4 first admin sign-ins (email + temporary password), shown once. Each admin chooses a new password at first sign-in.
   You should also see `Email: Email is ready`. If it says `Email NOT working`, fix `GMAIL_USER` / `GMAIL_APP_PASSWORD` or check the plan is paid.

## Step 2: Vercel (website + portals)
1. In `frontend/vercel.json` replace `YOUR-RENDER-SERVICE` so the destination reads `https://otw-logistics-api.onrender.com/api/:path*` (your real Render name). Commit and push.
2. Vercel > **Add New > Project** > same repo. **Root Directory: `frontend`**, Framework Preset **Other**, leave Build Command empty. Deploy.
3. (Recommended) Add your own domain in Vercel > Settings > Domains.

## Step 3: tell the backend its public address
In Render > Environment set (then it restarts):
| Setting | Value |
|---|---|
| `PUBLIC_URL` | your Vercel address or domain, `https://www.your-domain.com` (no trailing slash) |
| `ALLOWED_ORIGIN` | same as `PUBLIC_URL` |
| `ALLOWED_HOSTS` | `otw-logistics-api.onrender.com,www.your-domain.com` (no `https://`; also add the `*.vercel.app` address if you use it) |
| `TRUST_PROXY` | `2` (already set by `render.yaml`) |
Wrong values show as "This server is not set up for this web address" (421) or sign-in refusing: re-check these three.

## Step 4: test (before announcing)
1. Open `https://<your site>/` > fill the **Franchise** form with **your own email**, attach an Aadhaar image/PDF, submit.
2. Confirmation page with an ID (`FP-000001`), then the email **"Application received: FP-000001 (Franchise)"** within a minute (check Spam once).
3. `https://<your site>/dashboard/` > sign in > **Client > Franchise > View form** > *Email history* says *sent*.
4. Also try `/vendor/` and `/franchise/` pages load, and upload a **5 MB** PDF as Aadhaar once (see the note below).

**Upload-size note.** Vercel documents a 4.5 MB request limit for its Functions; plain proxy rewrites are generally reported
to pass larger bodies, but I could not test that on a real Vercel account. If step 4's 5 MB upload fails, let only the public
application form talk to Render directly (it needs no sign-in): add
`<script>window.OTW_API_BASE="https://otw-logistics-api.onrender.com";</script>` before the `partner-forms.js` line in
`frontend/index.html` and `frontend/services.html`.

## Backups and limits
- Everything is on the Render disk `/data`: `payments.sqlite`, `uploads/`, and `uploads/.partner-document-key`. **Without that key the
  uploaded Aadhaar / salary-slip files can never be opened again.** Use Render disk snapshots and also keep your own copy of the folder.
  Safer: set `PARTNER_DOCUMENT_ENCRYPTION_KEY` (`openssl rand -base64 32`) in Render and save that value in a password manager.
- One backend instance only (SQLite on one disk). Do not scale it to 2+ instances. A deploy causes a short restart.
- Editing `backend/shared/payment-excel.cjs`? Run `node scripts/sync-shared.mjs` (a test fails if the browser copy differs).

## Updating later
Push to GitHub: Vercel and Render redeploy by themselves. The disk and database are kept.
