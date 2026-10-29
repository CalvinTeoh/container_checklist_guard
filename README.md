# Container Checklist — Main Gate

A single-page checklist for the security guard at the main gate to log
container inspections: container number, seal number, exterior condition,
and photos (1 of the container number, 1 of the seal, 4 of the container's
sides). Each submission generates a PDF report and downloads it to the
device, and logs a live entry to a shared **Supabase** table — so the
office sees it immediately, on any device, no refresh needed.

It's a static site — `index.html` is the entire app, no build step and no
custom backend. Supabase (Postgres + Storage) is the data layer, called
directly from the browser with a public, read/write-restricted-by-policy
key.

## Deploy to Vercel

1. Push this repo to GitHub (already done if you're reading this from the repo).
2. In Vercel: **Add New → Project**, import `CalvinTeoh/container_checklist_guard`.
3. Framework preset: **Other** (plain static site). Leave build command and
   output directory blank — Vercel will serve `index.html` from the repo root.
4. Deploy. That's it — no environment variables needed.

Vercel also redeploys automatically on every push to the default branch.

## What it does

- **Container number & seal number** — enter the number, confirm it's
  correct (Yes/No), with a correction field if No, plus a photo of each.
- **Condition of container** — flag visible damage (Yes/No) with a
  description field if Yes, plus 4 photos (front, rear, left, right).
  Photos upload to Supabase Storage as soon as they're taken.
- **Submit** generates a PDF (inspection details + a photo page), downloads
  it to the device, and inserts a row into the `inspections` table.
- **Log tab** shows today's entries live from Supabase, with realtime
  updates — any device with the app open sees a new entry the moment
  another guard logs it, including thumbnail photos.

## Supabase setup (already done for this repo)

Project: `Container_checklist_guard` (`ztbsnoxeryekmkplyhlo`, `eu-west-3`).

- **Table** `public.inspections` — one row per inspection (container/seal
  numbers as entered and as corrected, damage flag + description, guard
  name, timestamp, and a public photo URL per angle).
- **Storage bucket** `inspection-photos` — public bucket, one object per
  photo at `<report-id>/<photo-key>.jpg`.
- **Row Level Security** — enabled on both the table and
  `storage.objects`. Per your choice, policies allow the `anon` role
  (i.e. anyone with the app link, no login) to **insert and select**
  freely. There's no PIN/auth gate — don't share the deployed URL more
  widely than you want write access to the log.
- **Realtime** — `inspections` is added to the `supabase_realtime`
  publication so the Log tab updates live via `postgres_changes`.
- **Retention** — none configured; rows are kept indefinitely. Prune
  manually in the Supabase dashboard if needed later.

The project URL and publishable (anon) key are inlined in `index.html`
(`SUPABASE_URL` / `SUPABASE_ANON_KEY` near the top of the script) — this
is normal for a Supabase project used from a public client; it's safe
specifically because the access rules above are enforced server-side by
RLS, not by hiding the key.

## Limitations (by design, since this is a static site)

- **No automatic emailing.** The PDF downloads to the device; the guard
  attaches it manually in an email app. Automatic emailing on submit
  needs a small backend (e.g. a Supabase Edge Function that calls an
  email API) — happy to help add one.
- **No write authentication.** Anything with the Vercel URL can submit an
  inspection (by design, per the access-control choice above). If that
  becomes a problem, add a shared passcode gate or move to real
  Supabase Auth.
- **Camera capture** uses the device's native "take photo" prompt
  (`<input type="file" capture="environment">`). This opens the camera
  directly on most phones/tablets; on a desktop browser with no camera it
  falls back to a file picker.

## Files

- `index.html` — the whole app (markup, styles, script, Supabase client).
- `vercel.json` — static-site config (clean URLs, a few safe default headers).
