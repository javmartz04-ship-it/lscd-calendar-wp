# LSCD Calendar — Setup & How To Update

Two parts: a **Google Sheet** the studio edits, and **one block of code** pasted
into the WordPress page. Set it up once; after that, updating the schedule is
just editing a spreadsheet.

---

## Part 1 — Make the Google Sheet (one time, ~5 minutes)

1. Go to **sheets.google.com** and create a new blank spreadsheet.
   Name it something like `LSCD Class Schedule`.

2. You need **two tabs, named exactly**:
   - `Classes`
   - `Events`

   (The tab name is the little label at the bottom. Rename the default `Sheet1`
   to `Classes`, then add a second tab and name it `Events`. Spelling and
   capitalisation matter.)

3. Fill them from the starter files in `starter-sheet/`, which already contain
   the complete, correct 2026–2027 schedule:
   - Open the `Classes` tab → **File → Import → Upload → `Classes.csv`** →
     choose **"Replace current sheet"** → Import.
   - Click the `Events` tab → **File → Import → Upload → `Events.csv`** →
     **"Replace current sheet"** → Import.

   She now starts from the real schedule instead of a blank page.

4. Click **Share** (top right) → under *General access* choose
   **"Anyone with the link"** → role **Viewer** → Done.
   The calendar can only read a sheet that is link-viewable. It stays
   read-only to the public; only people you invite can edit it.

5. Copy the **Sheet ID** out of the address bar. It's the long code in the middle:

   ```
   https://docs.google.com/spreadsheets/d/1AbCdEfGh...XyZ/edit#gid=0
                                          ^^^^^^^^^^^^^^^
                                          this is the Sheet ID
   ```

---

## Part 2 — Put it on the WordPress page (one time)

1. Open `wordpress-embed.html` in any text editor. Near the bottom, find:

   ```js
   var SHEET_ID    = "PASTE_SHEET_ID_HERE";
   ```

   Replace `PASTE_SHEET_ID_HERE` with the Sheet ID you copied. Keep the quotes:

   ```js
   var SHEET_ID    = "1AbCdEfGh...XyZ";
   ```

2. Copy the **entire** contents of that file.

3. In WordPress, edit the page where the calendar should live:
   - **Block editor:** add a **Custom HTML** block and paste it in.
   - **Elementor:** drag in an **HTML** widget and paste it in.

4. Publish. Done — you never have to touch this code again.

---

## How the studio updates the schedule from now on

Open the Google Sheet, edit it, done. The website picks up the change on its
own — no login to WordPress, no calling anyone.

**Note on timing:** Google serves the sheet through a short cache, so a change
usually shows up on the site within a few minutes, not instantly. If you want
to confirm it went through, do a hard refresh (Cmd/Ctrl + Shift + R).

### The `Classes` tab

| Column | What goes in it | Required |
|---|---|---|
| Day | Monday, Tuesday, Wednesday, Thursday, Friday, Saturday | **yes** |
| Studio | A, B, C, or D | **yes** |
| Class | The class name, e.g. `Pre-Ballet 3` | **yes** |
| Ages | e.g. `ages 9–10` — leave blank if not age-specific | no |
| Teacher | First name, e.g. `Jess` | no |
| Start | e.g. `5:15 PM` | **yes** |
| End | e.g. `6:15 PM` | **yes** |
| Style | Ballet, Jazz, Hip Hop, Contemporary, Tumbling, Turns & Leaps, Fitness, Company | no |
| New | Put `Yes` to show the blue **NEW** badge. Leave blank otherwise. | no |
| Note | Small italic note, e.g. `by admission only` | no |

- **To add a class:** type a new row at the bottom. Order doesn't matter —
  the calendar sorts by time automatically.
- **To remove a class:** delete the whole row.
- **To change a time:** edit the Start/End cells.
- Always include `AM`/`PM`. (24-hour like `17:15` also works.)

### The `Events` tab

| Column | What goes in it | Required |
|---|---|---|
| Title | e.g. `Thanksgiving Break` | **yes** |
| Start Date | e.g. `2026-11-23` (or `11/23/2026`) | yes* |
| End Date | Only for multi-day events. Blank for single days. | no |
| Date Label | What parents see, e.g. `Nov 23–28`. Leave blank and it's written for you. | no |
| Month | Only needed for a **TBA** event with no date, e.g. `May 2027` | yes* |
| Location | e.g. `Laredo Country Club` | no |
| Note | Small italic fine print | no |
| No Classes | Put `Yes` for a closure — shows the red shading and a NO CLASSES tag | no |

\* Every event needs **either** a Start Date **or** a Month.

---

## Why this can't take the calendar down

The code is built so a spreadsheet mistake can't break the live page:

- **A built-in copy of the schedule ships inside the code.** If the Sheet is
  ever deleted, renamed, made private, or Google is down, the calendar quietly
  shows that instead. It never goes blank.
- **An empty sheet is ignored.** If someone clears the tab by accident, the
  calendar keeps showing the last good built-in schedule rather than nothing.
- **One bad row doesn't sink the rest.** A row missing a day, a class name, or
  with an unreadable time is skipped; every other class still shows.
- **Column order doesn't matter**, and headers are matched loosely — `Teacher`,
  `Instructor` and `With` all work.
- **The code is sealed off from the website's theme.** Everything is scoped to
  `#lscd-calendar`, so it cannot restyle the rest of the site, and the theme
  cannot restyle it. This was tested against a deliberately hostile theme.

**Worth doing once a year:** after the new season's schedule is finalised in the
Sheet, ask Javier to refresh the built-in copy inside the code so the safety net
stays current. Everything keeps working if you skip it — the safety net just
gets older.

---

## Troubleshooting

**The site still shows the old schedule.**
Wait a few minutes and hard-refresh. If it persists, clear the WordPress/
Hostinger page cache.

**A class I added isn't showing.**
Check that row has a Day, a Class name, and both times with AM/PM. Open the page,
press F12 → Console — skipped rows are reported there by name.

**Everything reverted to the original schedule.**
That's the safety net. It means the Sheet couldn't be read — most likely sharing
got switched off, a tab was renamed, or the Sheet ID changed. Re-check step 4
and step 1 above.
