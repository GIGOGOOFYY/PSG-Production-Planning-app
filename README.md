# PSG Production Planning

Your production-planning app (Order Book, per-work-center Received/Done/Balance flow,
NCR/rework, capacity, holidays, users, Excel import) as a **single static file** that
runs on **Supabase + Cloudflare** — the same design and behaviour as the Railway
version, with no Node server to keep running.

`public/index.html` is the whole app: your original frontend, plus a browser-side shim
that implements every `/api/*` route (ported from `server.js`) directly against
Supabase. Deploy it as a Cloudflare Worker (static assets) from this GitHub repo.

Work centers: **CT** Cutting · **GR** Grinding · **PE** Polish Edge · **BV** Beveling ·
**HL** Hole/Drilling · **PR** Printing · **TG** Tempering · **BG** Bending ·
**LG** Lamination · **FR** Frosting · **DG** Double Glazing · **QC** Quality Control.

---

## 1. Supabase (once)

> ⚠️ This uses a **different table layout** than the earlier `psg_*` version. Run the
> two SQL files below fresh. Any old `psg_orders`/`psg_queue` tables can be ignored or
> dropped — they aren't used here.

1. In your Supabase project → **SQL Editor → New query**, paste **`schema.sql`**, **Run**.
   Creates: `users`, `orders`, `wc_progress`, `ncr`, `holidays`, `capacity`.
2. **New query** → paste **`sample-data.sql`** → **Run**. Loads your Order Book (340
   orders), the current per-work-center progress, capacities, and holidays.
   *No user accounts are seeded — you create your admin in the app on first open.*
3. **Project Settings → API**: the Project URL + anon key are already wired into
   `public/index.html`. (To rotate them, edit `SB_URL` / `SB_KEY` near the top of the
   `<script>` shim.)

## 2. GitHub

```bash
git init && git add . && git commit -m "PSG Production Planning"
git branch -M main
git remote add origin https://github.com/<you>/PSG-Production-Planning-app.git
git push -u origin main
```

## 3. Cloudflare Worker (connected to GitHub)

Cloudflare dashboard → **Workers & Pages → Create → Workers → Import a repository** →
pick the repo. It reads `wrangler.toml` (serves `public/` as static assets) and deploys.
Every push to `main` redeploys. *(Cloudflare **Pages** also works: Create → Pages →
connect repo → build output directory `public`.)*

## 4. First run

Open the deployed URL. On first visit you'll get a **Create admin account** screen —
pick your own username and password (this replaces the old hard-coded `admin/admin123`).
After that it's your normal login. Add more team members under **Users**.

---

### Security note
The anon key sits in the page and the tables use permissive access policies, so anyone
with the URL can read/write — fine for an internal tool, same as the Railway setup.
Passwords are hashed, but for a hardened multi-tenant setup you'd move auth to Supabase
Auth with per-row policies. Ask and I'll set that up.

### Files
| Path | Purpose |
|------|---------|
| `public/index.html` | The whole app (frontend + Supabase shim). |
| `schema.sql` | Creates the tables. |
| `sample-data.sql` | Loads your data. |
| `wrangler.toml` | Cloudflare Worker (static assets) config. |
