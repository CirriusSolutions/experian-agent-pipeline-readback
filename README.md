# Experian Agent Pipeline — Readback Site

Client-facing living design document for the Experian Agentic Salesforce Delivery
Pipeline engagement. Deployed via GitHub Pages, gated by Cloudflare Access (see below).

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
directly to a Supabase backend (project `zdonuvafqcviapusqwsv`) and see prior submissions and their
status. Testers can add feedback but not change status — that's done via the admin page above.
Setup SQL for the Supabase table/bucket/policies lives in Gavin's vault at
`00 INBOX/Agent Task Outputs/Experian - UAT Feedback - Supabase Setup.sql`.

## Updating

This repo is a deploy target, not the source of truth. The source files live in Gavin's
vault at `00 INBOX/Agent Task Outputs/Experian - BA Process Design Readback - 2026-08-10.html`
and `00 INBOX/Agent Task Outputs/Experian - UAT Feedback Admin.html`.
To publish an update, copy the current version of those files (and the two PoC preview files,
if changed) over the matching files here and push to `main` — GitHub Pages redeploys
automatically.

## Access control

GitHub Pages has no built-in authentication — anyone with the URL can reach it once Pages
is enabled, private-repo entitlement aside. This repo is intended to sit behind
**Cloudflare Access** in front of the Pages URL, consistent with how other Cirrius
`*-artifacts` client sites are gated. Setup (one-time, in the Cloudflare dashboard):

1. Add/confirm the domain you want to use (e.g. a subdomain of an existing Cirrius-owned
   zone) is on Cloudflare.
2. Cloudflare dashboard → **Zero Trust → Access → Applications → Add an application**
   → **Self-hosted**.
3. Point it at the custom domain/subdomain you'll CNAME to this Pages site
   (`experian-agent-pipeline-readback.pages.dev` if using Cloudflare Pages, or the GitHub
   Pages hostname if fronting GitHub Pages directly with a Cloudflare-proxied CNAME).
4. Add a policy: **PIN / One-time code** (email-based OTP) restricted to the specific
   client email addresses, or a shared **Service Token** if a numeric PIN without email
   verification is preferred.
5. Save — Cloudflare now intercepts every request and enforces the gate before it reaches
   the site; nothing in this repo needs to change for that to work.

Until Access is wired up, treat the raw GitHub Pages/repo URL as unlisted, not secured —
do not send it to Experian yet.
