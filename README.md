# Experian Agent Pipeline — Readback Site

Client-facing living design document for the Experian Agentic Salesforce Delivery
Pipeline engagement. **Live architecture (corrects the previous version of this README,
which described a GitHub Pages + Cloudflare Zero Trust Access setup that was never actually
what's deployed):**

This repo holds only the static asset files. The actual production site is a
**Cloudflare Worker** named `experian-agent-pipeline-readback`, connected to this repo via
Cloudflare's Git integration — every push to `main` redeploys the assets automatically.
The Worker's own script (a custom PIN + signed-cookie gate, plus the `/api/feedback/*`
routes backing the UAT Feedback tab) is **not** in this repo; it's deployed separately via
`wrangler deploy`. Its source of truth is Gavin's vault at
`00 INBOX/Agent Task Outputs/experian-feedback-worker/` (`index.js` + `wrangler.toml` + a
README with redeploy steps) — worth moving to a proper repo with CI if this pipeline stays
long-lived.

**Access control:** gated by a shared PIN, checked server-side by the Worker (not Cloudflare
Access/Zero Trust as this README previously claimed — that was never built; a simpler
custom gate was used instead). A correct PIN sets an HMAC-signed session cookie, valid 24
hours. The PIN and cookie-signing secret are Cloudflare Worker secrets — never in this repo
or any client-visible source.

## Contents

- `index.html` — the readback: overview, discovery findings, target architecture,
  user stories, demo walkthrough, what we need from Experian, and a UAT Feedback tab.
  Updated at each readback session, not a one-time deck.
- `Experian - PoC Preview - CPQ Product.html` / `Experian - PoC Preview - Account LWC.html`
  — standalone Salesforce Lightning-style PoC mockups linked from the Demo Walkthrough tab.
- `Experian - UAT Feedback Admin.html` — internal triage view for UAT feedback (passphrase-gated,
  soft deterrent only — see note in that file). Not linked from the main nav; reachable only
  by its direct URL. Sets each submission's status (open / resolved-working / N/A-out of scope).

## UAT Feedback

The readback page's "UAT Feedback" tab lets testers submit issues (with optional screenshots/docs)
and see prior submissions and their status. Testers can add feedback but not change status —
that's done via the admin page above, itself gated by a separate passphrase (also a Worker
secret, checked server-side via `/api/feedback/verify-pass` and `/api/feedback/status`).

The front-end never talks to Supabase directly and holds no Supabase credentials — it calls
same-origin `/api/feedback/list`, `/api/feedback/submit`, and `/api/feedback/status`, which
the Worker implements server-side using a Supabase anon key stored as a Worker secret
(`SUPABASE_ANON_KEY`). Backend: Supabase project `zdonuvafqcviapusqwsv`. Setup SQL for the
table/bucket/RLS policies lives in Gavin's vault at
`00 INBOX/Agent Task Outputs/Experian - UAT Feedback - Supabase Setup.sql`.

## Updating

**Page content** (this repo): the source files live in Gavin's vault at
`00 INBOX/Agent Task Outputs/Experian - BA Process Design Readback - 2026-08-10.html`
and `00 INBOX/Agent Task Outputs/Experian - UAT Feedback Admin.html`. Copy the current
version of those files (and the two PoC preview files, if changed) over the matching files
here and push to `main` — the Worker's Git integration redeploys the assets automatically.
This does **not** touch the Worker's script or secrets.

**Worker script/API logic** (not in this repo): edit the script in the local
`experian-worker-deploy/` workspace and run `wrangler deploy` from there. See that
workspace's `wrangler.toml` — it must keep `run_worker_first = true` under `[assets]`, or
the PIN gate silently stops applying to static pages (Cloudflare serves matching static
files directly, bypassing the Worker, unless this is set — this broke once during setup
and was caught by testing before going live).

**Worker secrets** (`SITE_PIN`, `COOKIE_SECRET`, `SUPABASE_ANON_KEY`, `ADMIN_PASS`): set via
`wrangler secret put <NAME> --name experian-agent-pipeline-readback`. These persist across
script deploys automatically — no need to re-set them unless rotating a value.
