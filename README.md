# Offlyn — Event Carousel

Turns a CSV of events into branded Offlyn slide carousels, ready to post.

**Live:** _(add your Vercel URL here after the first deploy)_

## What it does

- Paste or upload a CSV. Any column order — a mapper guesses which column is which and shows a live preview of the first card.
- Groups events into one carousel per day, ordered by start time.
- Renders 4:5 slides (1080×1350) with 9 event cards each, matching the Offlyn brand.
- Exports per day: PDF, a zip of PNGs, or a single slide as PNG.
- Click any text on a slide to edit it before exporting.

## Column handling

The mapper reads four fields. It recognises them by header name and by content:

| Field | Source | Notes |
|---|---|---|
| Event title | `Title` | Clipped to two lines; the toolbar flags how many are too long |
| Start time | `Time` | `9:30 AM – 11:45 AM` → `9:30 AM`; `19:00` → `7:00 PM` |
| Host | `Description` | Extracts the name from `Hosted by X — …` |
| City | `City` | `Bengaluru` → `BLR`. Falls back to scanning the address |
| Date | `Date` | Accepts `2026-08-28`, `28/08/2026`, `28 Aug 2026`, `Aug 28, 2026` |

City codes and their band colours are defined in one object near the top of the
app script (`CITIES`). DEL, BLR and MUM are the approved brand colours; the rest
are placeholders.

## Deploying

Single self-contained file — no build step, no dependencies to install.

**Vercel:** import the repo, framework preset **Other**, leave Build Command
empty, Output Directory `.` (root). Every push to `main` redeploys.

## Editing

Everything lives in `index.html`: styles in the `<style>` block, app logic in the
last `<script>` block. The three earlier `<script>` blocks are vendored
libraries (jsPDF, JSZip, html2canvas) — leave them alone.

TAN Harmoni and Figtree are embedded as base64 woff2, so the app renders
identically offline and exports never fall back to a substitute font.
