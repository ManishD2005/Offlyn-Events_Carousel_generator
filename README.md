# Offlyn — Event Carousel

Turns a CSV of events into branded Offlyn slide carousels, ready to post.

**Live:** _(add your Vercel URL here after the first deploy)_

## What it does

- Paste or upload a CSV. Any column order — a mapper guesses which column is which and shows a live preview of the first card.
- Groups events into one carousel per day, ordered by start time.
- Renders 4:5 slides (1080×1350) with 9 event cards each, matching the Offlyn brand.
- Upload a title slide as a PNG — it leads every carousel as slide 1.
- Exports per day: PDF, a zip of PNGs, or a single slide as PNG.
- Click any text on a slide to edit it before exporting.

## Title slide

Optional. Upload (or drag in) a PNG or JPG on the editor screen and it becomes
slide 1 of every carousel, ahead of the event slides. It's drawn straight onto
the 1080×1350 canvas rather than going through html2canvas, so a 4:5 image
exports pixel-for-pixel; anything else is cropped to fill, and the uploader says
so before you generate.

## Export naming

Files are named for the day they cover, then the slide number, so a folder of
them sorts by date:

```
2026-08-28-1.png   ← title slide
2026-08-28-2.png
2026-08-28-3.png
2026-08-28.pdf
2026-08-28.zip
```

The date comes from the earliest event on that carousel. If the sheet carried no
parseable date, the name falls back to the slide header instead (`28-aug-1.png`)
and the download menu flags it.

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
