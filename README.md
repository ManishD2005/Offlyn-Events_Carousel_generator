# Offlyn — Event Carousel

Turns a CSV of events into branded Offlyn slide carousels, ready to post.

**Live:** _(add your Vercel URL here after the first deploy)_

## What it does

- Paste or upload a CSV. Any column order — a mapper guesses which column is which and shows a live preview of the first card.
- Groups events into one carousel per day, ordered by start time. Days too small
  to fill a slide merge into the next date rather than going out half empty.
- Renders 4:5 slides (1080×1350) with up to 9 event cards each, matching the Offlyn brand.
- Upload a title slide as a PNG — it leads every carousel as slide 1.
- Exports per day: PDF, a zip of PNGs, or a single slide as PNG.
- Click any text on a slide to edit it before exporting. A title too long for
  three lines is flagged under the slide, and the flag is the button that puts
  your caret in it.

## How days become carousels

A slide holds nine events. A day with nine or more fills at least one on its own,
so it posts by itself. A day with fewer doesn't, so it rides along with the next
date and the carousel is headed by the span it covers:

| Events per day | Result |
|---|---|
| 4 on the 28th, 5 on the 29th | one post, `AUG 28 - 29` |
| 12 on the 28th, 3 on the 29th | two posts — the 3 has nothing after it to merge into |
| 3 + 3 + 3 across the 26th–28th | one post, `AUG 26 - 28` |
| 2 on the 28th, 20 on the 29th | one post, `AUG 28 - 29` |

Days accumulate until they fill a slide, then the group closes — so a run of
small days collects into one post rather than all piling onto the next big one.
A short tail at the end of the sheet has nothing left to merge into and posts as
it is. Each slide is still headed by the dates actually on it, so slide 1 of a
merged carousel reads `AUG 28 - 29` while a later all-29th slide reads `AUG 29`.

Pick **One carousel — fill every slide** in the mapper if you'd rather ignore
day boundaries entirely.

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
| Event title | `Title` | Clipped to three lines; over-long ones are flagged under the slide |
| Start time | `Time` | `9:30 AM – 11:45 AM` → `9:30 AM`; `19:00` → `7:00 PM` |
| Host | `Description` | Extracts the name from `Hosted by X — …` |
| City | `City` | `Bengaluru` → `BLR`. Falls back to scanning the address |
| Date | `Date` | Accepts `2026-08-28`, `28/08/2026`, `28 Aug 2026`, `Aug 28, 2026` |

Known city codes live in one object near the top of the app script (`CITIES`),
with the spellings that resolve to them in `CITY_ALIASES` just below. Mumbai
resolves to `MUM`; change that entry if you want `BOM` on the cards instead.

## The slide design

Every measurement in the `.slide` rules is taken from the reference artwork,
which is drawn at 1080×1350, so the numbers in the stylesheet are literal rather
than scaled — the frame really does start at x 49, y 356.

- **Ground.** The purple paper texture is embedded as a base64 JPEG on `.slide`,
  resampled to exactly 1080×1350 so the export is 1:1 with no resampling.
- **Header.** `Events happening on` and the date are both TAN Harmoni, at 60px
  and 157px, positioned absolutely against measured ink positions rather than
  flowed — which is why they carry pixel offsets like `top:69px`.
- **Stamp frame.** The pink border is one SVG path built by `stampPath()`:
  semicircular bites along each straight edge, and a larger quarter-circle
  scooped out of each corner. Walked clockwise every arc is concave, so all the
  sweep flags are 0. Bite spacing is ~68px and the count follows from the edge
  length, so the rhythm holds when the frame shrinks.
- **Short slides.** A full slide is three rows of three. The last slide of a
  carousel usually isn't, so the frame wraps only the rows that hold cards
  instead of leaving a wall of pink. Set `frameHeight()` to return a constant
  957 if you would rather every slide framed the same.
- **Colour.** The frame cycles pink → orange → green → yellow → sky blue across
  a carousel's slides and wraps round if there are more than five, so a post
  reads as a sequence. Each carousel restarts at pink. The order lives in
  `TINTS`; the title slide is skipped, since it has no frame to colour.
- **Cards.** Fixed 293×285 with a 5px radius. The time and host chips are
  positioned absolutely rather than flowed, so a one-line title never lifts them
  out of the rhythm the grid reads by.

## Deploying

Single self-contained file — no build step, no dependencies to install.

**Vercel:** import the repo, framework preset **Other**, leave Build Command
empty, Output Directory `.` (root). Every push to `main` redeploys.

## Editing

Everything lives in `index.html`: styles in the `<style>` block, app logic in the
last `<script>` block. The three earlier `<script>` blocks are vendored
libraries (jsPDF, JSZip, html2canvas) — leave them alone.

TAN Harmoni and Figtree are embedded as base64 woff2, so the app renders
identically offline and exports never fall back to a substitute font. The
background texture is embedded the same way, which is most of the file's size.

`favicon.svg` is kept alongside as the editable source, but the copy the page
actually uses is the data URI in the `<head>` — re-inline it if you change the
file, or the tab icon won't move.
