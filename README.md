# Container Checklist — Main Gate

A single-page checklist for the security guard at the main gate to log
container inspections: container number, seal number, exterior condition,
and photos (1 of the container number, 1 of the seal, 4 of the container's
sides). Each submission generates a PDF report and downloads it to the
device; a log of today's entries is kept on that device for quick review.

It's a static site — `index.html` is the entire app, no build step and no
backend required.

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
- **Submit** generates a PDF (inspection details + a photo page) and
  downloads it straight to the device.
- **Log tab** shows today's entries, kept in the browser's local storage on
  that device — useful for the guard to review a shift's inspections, but
  it does **not** sync across devices (see Limitations).

## Limitations (by design, since this is a static site)

- **No shared/live log across devices.** The log tab only shows entries
  saved in that browser's local storage, on that device. If you need a
  shared log the back office can watch in real time, that needs a small
  backend (e.g. a database + API route) — happy to help add one.
- **No automatic emailing.** The PDF downloads to the device; the guard
  attaches it manually in an email app. Automatic emailing on submit also
  needs a backend (an API route that calls an email service).
- **Camera capture** uses the device's native "take photo" prompt
  (`<input type="file" capture="environment">`). This opens the camera
  directly on most phones/tablets; on a desktop browser with no camera it
  falls back to a file picker.

## Files

- `index.html` — the whole app (markup, styles, script).
- `vercel.json` — static-site config (clean URLs, a few safe default headers).
