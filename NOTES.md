# LSCD Calendar — WordPress + Google Sheet version

**Built:** 2026-09-01 · **Deliverable:** `wordpress-embed.html`, a single block
pasted into a WordPress Custom HTML / Elementor HTML widget, reading its data
live from a Google Sheet.

## Brief

Javier: the studio owner needs to update the schedule herself, the site is
WordPress on Hostinger, and the GitHub Pages links were only ever for review.
He floated uploading a **picture** of the schedule and having it auto-update.
Asked for "the best way to do this so it never mess up."

**Recommendation given, and accepted:** picture/OCR is the one part that
genuinely cannot be made reliable — a misread `6:15` vs `6:45` publishes a
wrong class time silently and nobody notices until a parent shows up at the
wrong hour. Steered to a **Google Sheet as the source of truth** (she already
works in Excel, so it's the same motion), with the picture idea available later
as a *drafting* step into the Sheet behind a human confirm, never as the live
source. Javier chose Sheet-only, delivered via a WordPress code block.

Scope note: the schedule changes ~2–3 times a year. An OCR pipeline would be a
lot of machinery for that cadence — worth revisiting only if the edit frequency
changes.

## What was built

Same calendar as `builds/lscd-calendar-simple` (studio × day table, browse by
day, month calendar) with the data layer swapped from hardcoded arrays to a
live Google Sheet fetch, plus everything needed to survive a real WordPress
theme.

**Reliability design — the whole point of the build:**

1. **Built-in fallback schedule** ships inside the code. The page renders it
   *immediately* on load, then quietly swaps in sheet data once it arrives. The
   calendar is therefore never blank, never mid-load-empty, and survives the
   Sheet being deleted, renamed, un-shared, or Google being unreachable.
2. **Empty sheet is ignored** — if a tab is cleared by accident, the built-in
   data stays rather than blanking the page.
3. **Per-row validation.** A row missing day/name/valid times, or with an
   invalid studio, or end ≤ start, is skipped with a console warning; the other
   rows still render. One typo can't take the calendar down.
4. **Loose header matching** — columns can be reordered and headers can be
   spelled several ways (`Teacher`/`Instructor`/`With`).
5. **Forgiving time parsing** — `5:15 PM`, `5:15pm`, `17:15` all work. With no
   AM/PM given, 9–11 read as morning and 1–8 as evening, which matches the
   studio's real 9am–10pm operating window.
6. **Cache-busting** on the fetch (`nocache` param + `cache:"no-store"`),
   because WordPress and Hostinger both cache aggressively and would otherwise
   make her think her edit didn't save.
7. **Full CSS/JS isolation.** Everything scoped to `#lscd-calendar`: no global
   reset, no `body` styles, CSS variables namespaced `--lscd-*`, all JS inside
   an IIFE querying within the root element only, and hooks are `lscd-`-prefixed
   classes rather than generic ids that could collide with the theme.

## Verification

`runtests.js` (headless Chromium) drives seven scenarios against test pages that
embed the snippet inside a **deliberately hostile fake WordPress theme** — loud
serif type, dashed-red tables, 18px-radius buttons, and a `[hidden]{display:block
!important}` rule designed to break the tab panels. **40/40 checks pass:**

- no sheet configured → built-in 66 classes render
- live sheet → sheet data replaces built-in, quoted CSV fields with commas intact
- broken rows → 4 good rows survive, 4 bad rows skipped, times parsed correctly
- empty sheet → falls back, does not blank
- 404 sheet → falls back
- isolation → theme untouched by embed **and** embed untouched by theme
  (this caught a real leak: the theme's `td{color}` was reaching the table cells;
  fixed by restating text properties on the table rules)
- interactions → tabs, month nav and day switching all work inside the theme page

Screenshots of the embed inside the hostile theme are in `shots/`.

## Files

- `wordpress-embed.html` — the paste-in block. One line to edit (`SHEET_ID`).
- `SETUP.md` — plain-English setup + "how to update" for the client.
- `starter-sheet/Classes.csv`, `starter-sheet/Events.csv` — the real, verified
  schedule, generated programmatically from the shipped build so the Sheet
  starts correct rather than blank. Times written human-readable (`5:15 PM`).

## Known limits, stated honestly

- **Google's published-sheet cache** means edits appear in a few minutes, not
  instantly. Documented in SETUP.md so it doesn't read as a bug.
- **The built-in fallback goes stale** unless refreshed once a season. Harmless
  (it only appears when the Sheet is unreachable) but noted in SETUP.md.
- **Requires the Sheet to be link-viewable.** If she ever tightens sharing, the
  calendar silently falls back — the troubleshooting section names this as the
  first thing to check.

## Lesson for `system/` (folded in)

Anything pasted into a client's WordPress via a code block must be treated as
hostile-environment code: scope every selector, namespace the variables, and
prove it with a test page that wraps the snippet in a deliberately aggressive
fake theme. The fake-theme harness found a real style leak that reading the CSS
would not have.

## Studio review — 2026-09-02 (Loom, ~27:00–30:05)

Jess on the calendar: *"Yeah, I love it. I think it's very clean."* Two changes, nothing else:

1. **Script header matches the site.** *"Where it says Laredo school, if we can just keep the font consistent, and it's a brush script."* Great Vibes → **Hurricane** (the live site's script face), sizes bumped ~25% because Hurricane runs smaller.
2. **Class names shouldn't wrap like they're cut off.** *"Where it says grade three to four and pointe 3/4, like the 3/4 is not on Monday… it doesn't look like it has a second line, like it's incomplete. Will we be able to expand the columns?"* The very long Thursday class is fine to wrap: *"at least it reads like two classes."*
   - Container 1180 → 1400px, side gutter 40 → 24px at ≤1440px.
   - Each day column is **measured from its longest class name** (table-layout fixed + generated `<colgroup>`), not split evenly — Friday doesn't need Wednesday's width. A name over 230px is left out of the measurement and allowed to wrap, so "Preparatory Grade 2 / Grade 1 & Pointe 1" doesn't make Thursday huge and squeeze every other day.
   - Measured live, so it keeps working when class names change in the Sheet.
   - Result: at 1366px and wider only the Thursday class wraps. At 1280 some still wrap — there isn't enough physical width at that size.
   - On phones (≤860px) the table gets its full natural width, since it already scrolls sideways there.

**Scope flag, not built:** in the call Josh described the update flow as *"you export the PDF… I give you a little form, you upload it, hit submit, and it will auto update."* That's a PDF upload, not the Google Sheet Javier chose. Worth reconciling with Josh before the studio gets instructions.

**Heads-up for WordPress:** Elementor's default content width is 1140px. The embed needs a full-width section, or the wider columns get squeezed back.

## Font follow-up — 2026-09-17

After the review, Javier sent the studio's own MyFonts webfont kit: **Brush Script Std Italic** (Monotype, MyFonts build 3867246). That's the "brush script" Jess meant. The first pass had used Hurricane (the live site's script), which is thin and elegant; Brush Script Std is much heavier and matches the lettering in their logo. Shown side by side, Javier chose Brush Script.

- Font files are in `fonts/brush-script-std/`, loaded with `@font-face`. The fallback is `'Brush Script MT'`, the system version on Mac and Windows.
- All script sizes are about 16% smaller, because Brush Script Std sets wider than Hurricane at the same px. The line widths and layout stay the same.
- Licensing: the kit is licensed to the website owner (the studio). It's hosted in the public preview repos for now; long-term it belongs on their own server only.
- WordPress embed: the `@font-face` tries `/wp-content/uploads/lscd-fonts/` first, then the GitHub preview copy. Browsers fall through the `src` list on a 404; this was verified in the test run. SETUP.md now has an upload-the-font step.

## Second Loom review — 2026-09-17

Content changes mirrored from the standalone calendar: season line removed (and the `SEASON` constant with it), the "schedule subject to change · questions? call" line added under *This Week At A Glance* and centred above the closing rule, and "Dance with Us. Grow with Us." removed.

**Deliberately not mirrored: the black logo bar and the site footer.** This file is the *embed* — it sits inside the WordPress page, which already renders the site's real header and footer. Adding its own would double them up. The standalone page at `builds/lscd-calendar-simple` is the one that carries the chrome.
