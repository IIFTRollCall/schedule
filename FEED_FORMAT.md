# Roll Call schedule feed, format v1

This is how the official weekly schedules reach the apps. It is fixed: Android APKs stay on phones for months, so anything an old APK can't read must arrive as a new `feedVersion` (which old APKs answer with "please update").

Nobody writes these files by hand. The **publisher page** (`publisher.html`) makes them.

## Where it lives

| | address |
|---|---|
| main | `https://iiftrollcall.github.io/schedule/latest.json` |
| fallback (same repository) | `https://raw.githubusercontent.com/iiftrollcall/schedule/main/latest.json` |

Both apps have these two addresses built in. **Never rename the `iiftrollcall` organization, the `schedule` repository or its `main` branch.**

Every schedule file sits **in the same folder** as `latest.json`, and is named in `latest.json` relative to it.

## latest.json

```json
{
  "kind": "rollcall-schedule-feed",
  "feedVersion": 1,
  "sequence": 29861234,
  "publishedAt": "2026-10-19T21:10:00+05:30",
  "trimester": "V",
  "entries": [
    {
      "id": "T5-W07",
      "file": "T5-W07-r2.json",
      "sha256": "64 lowercase hex characters",
      "bytes": 41234,
      "revision": 2,
      "label": "week 7",
      "from": "2026-10-12",
      "to": "2026-10-18",
      "note": "fri slot moved"
    }
  ],
  "notice": { "id": "n-1792449497882", "text": "no classes on monday (diwali)", "until": "2026-10-20" },
  "android": { "latestVersionCode": 40401, "latestVersionName": "4.4.1", "link": "https://github.com/iiftrollcall/..." },
  "movedTo": null
}
```

| field | meaning |
|---|---|
| `kind` | always `"rollcall-schedule-feed"` |
| `feedVersion` | `1`. An app that reads a higher number stops and says "update roll call to keep getting schedules". Only raise it for a change old apps would misread. The apps ignore fields they don't know, but the publisher page rebuilds `latest.json` from the fields above only, so a new field also needs a publisher update. |
| `sequence` | a whole number that **only ever goes up**. Apps ignore any `latest.json` with a lower number than one they have already seen (so an old copy from a cache, or an old file restored from history, can't roll anyone back). They also ignore one more than five years (in minutes) ahead of the phone's clock, so a bogus huge number can't block every later feed. The publisher uses `max(highest seen anywhere + 1, minutes since 1970)`. |
| `publishedAt` | when it was made (display only) |
| `trimester` | the trimester the entries belong to (display only; cut to 40 characters) |
| `entries` | the current trimester's official schedule files, oldest first. **Only the current trimester**: a student who installs mid-trimester gets every week of it. |
| `notice` | optional one-line message for every student, shown on Home until `until` (inclusive) or until dismissed. Its `id` decides "dismissed": the publisher keeps the id while the text stays the same, and gives new text a new id. |
| `android` | optional: the newest Android version and where to get it. An older APK shows "roll call X is out". `link` must be on `https://github.com/iiftrollcall/` or `https://iiftrollcall.github.io/`, in plain characters (letters, digits, `. _ ~ / ? = & + -`: no `%`, no `\`) and without `..`; any other link is ignored. |
| `movedTo` | optional address of a new `latest.json`, **only** on `https://iiftrollcall.github.io/...` or `https://raw.githubusercontent.com/iiftrollcall/...` (anything else is ignored). See "Moving the feed". |

### An entry

| field | meaning |
|---|---|
| `id` | the slot. Stays the same across revisions. `T<trimester>-W<week>` for a weekly sheet (`T5-W07`), `T<trimester>-<name>` for anything else (`T5-midterm`, `T5-endterm`, `T5-extra-2026-10-24`). The file must be what its id says: a `T5-W07` file whose sheet says another week or trimester (or a `T5-midterm` file with a week number) is refused, not applied. |
| `file` | the file name, same folder, ends in `.json`. A revision is a **new file** (`-r2`, `-r3`); a published file is never changed. |
| `sha256` | fingerprint of the file's exact bytes. Apps refuse a file whose fingerprint doesn't match. |
| `bytes` | the file's size |
| `revision` | 1, 2, 3... per `id` |
| `label`, `from`, `to`, `note` | for display: "week 7 (12 to 18 oct), revision 2: fri slot moved" |

The schedule files themselves are the existing `"iift-weekly-schedule"` JSON, unchanged.

### Limits (the apps leave out anything beyond them)

| what | limit |
|---|---|
| `latest.json` | 256 KB |
| a schedule file | 1 MB (an entry claiming more is left out) |
| entries | 60; a repeated `id`, `file` or `sha256` is left out |
| one check | 8 MB downloaded at most; the rest waits for the next check (a file bigger than what's left of that doesn't count as wrong) |
| `id` | 64 characters: letters, digits, `.`, `_`, `-`, starting with a letter or digit |
| `file` | 128 characters, same characters, ends in `.json` |
| `label` / `note` | 80 / 200 characters (cut) |
| `notice` | text 280 characters, id 64 (cut); an `until` that isn't a real date counts as none |
| `android` | version name 20 characters; version code 1 to 2147483647 |
| addresses | https only, at most 500 characters |
| inside a schedule file | 1500 sessions and 80 courses (else the file is refused); text is cut (name 120, faculty 80, label 120, room 60, notes 300 characters) and control characters become spaces |
| nesting | JSON nested more than 64 levels deep is refused |

## What the apps do

1. **When:** on opening, on coming back to the app (at most every 30 minutes), when the phone comes back online (iPhone), after a restore or "clear all", and on **check now** (a "check now" or section change during a check gets its own check right after). Android also downloads in the background every few hours (first after 6 hours), but only the app itself applies anything.
2. Fetch `latest.json`: the main address, then the fallback. Ignore it if `kind` is wrong or `sequence` is out of bounds (above). A higher `feedVersion`: show "update roll call".
3. For each entry not yet applied in its current version **for this trimester and section**: download the file, check `bytes` and `sha256`, and read it with the normal importer for the student's section. A change of section, a restored backup or "clear all" means every file is applied again (an import is idempotent and keeps marks).
4. **Applied automatically** when it belongs in the trimester the student is in (or the student's trimester is still empty). A safety snapshot is taken first (at most one every 12 hours, and at most two of these kept, so they never push out the student's own); if it can't be stored, nothing is applied. The data is saved before the record of what's applied, and a check that a restore, clear or section change overtook records nothing. A "timetable updated" card then shows what changed (not for the very first fill; the toast says it), and a toast only for checks the student started.
   Re-applying a file keeps every mark where it was: two sessions numbered alike each keep their own, a cancelled class's history copy is left alone, and a class the student added by hand at the same date and time becomes the official one (keeping its mark and note).
5. **Asks first** when it would switch to or start a newer trimester: a card opens the normal import screen. A file for an older trimester than the student's is ignored.
6. Elective and sitting questions don't block: they show on the update card ("· 2 questions") and then as "answer 2 questions about your timetable", exactly like a manual import. They go once the file that asked is applied again without asking them.
   Home shows **one** timetable card at a time, in this order: roll call needs an update; a new trimester waiting; timetable updated; the maintainer's notice; questions; (Android) a newer APK.
7. **Downloads that go wrong:** a file that can't be reached (no internet, not uploaded yet) is simply tried again at the next check. A file that arrives wrong (another size or fingerprint, or too big) is tried 3 times, then skipped until the next `latest.json`. A file that downloads fine but can't be read, or isn't what its id says, is skipped until a new revision replaces it.
8. Nothing about the student leaves the phone. The requests carry no name, account or device id; GitHub sees the phone's internet address, as any website does. Android sends the plain user agent "RollCall".
9. Manual import (WhatsApp / Files) still works. When a file matches an official one, the import screen says so; when it's a different version of an official week, it says that too.

## Moving the feed

The apps always ask the built-in address first. When its `latest.json` has a valid `movedTo`, they read the feed there **for that check only**, and they remember it to use only if the built-in addresses can't be reached at all. So:

- to move, keep the old repository up with a `latest.json` carrying `movedTo`, for at least a year;
- to undo a move (or a wrong `movedTo`), take `movedTo` out of the real `latest.json`: every phone is back at the next check.

The publisher page never writes `movedTo` and refuses to publish while the live feed has one: a move is a deliberate hand edit.

## Fixing a mistake

Publish forward: run the publisher again with the corrected file. It becomes the next revision with a higher `sequence`. Don't restore an old `latest.json` from GitHub's history: phones that saw a newer one ignore it, and the next publish would build on the restored list.

A wrong file can also be taken off the list (publisher page, "other feed settings"). That only stops new installs getting it; students who have it keep its classes until a corrected sheet replaces them.

## How the publisher page makes the files

1. It reads `latest.json` from the site, from raw.githubusercontent.com and from GitHub's API (the committed copy), and builds on the one with the highest believable sequence. It never offers to start a new feed unless both addresses answer "not found". When GitHub's API copy can't be read (it allows 60 reads an hour), the other two can be minutes old, so the maintainer has to tick that nothing was published elsewhere in the last few minutes. The repeated "check it's live" never reads the API.
2. The new `sequence` is above every believable copy, and above any sequence still visible in a live copy the apps can't read (after a takeover, phones may have taken it), unless it is one no phone would take.
3. It remembers (in that browser) the last files it made, and won't make more until all of them are on the site (`latest.json` **and** the week's file) or the maintainer says they were never uploaded. So a second publish in a row can't be built on an old list. A newer `latest.json` made elsewhere that doesn't list the file made here is reported, not taken as done.
4. It checks the sheet with the apps' own importer (including the trimester as the apps read it: cleaned and cut to 40 characters) and stops on: a file the apps would refuse or read differently, a week whose dates don't overlap its own published week or overlap another published week, the same file already published, a moved or poisoned feed. A newer trimester, classes dated in another published week, a revision that takes exams or quizzes away, and other doubts need a tick; an older trimester needs its number typed. A sheet with no week number is never put in an already published exam slot by guesswork.
5. It reads the new `latest.json` back with the apps' own parser before offering it.
6. A file name is never reused: before naming `T5-W08-rN.json` it asks the site and raw.githubusercontent.com whether that name exists, and takes the next free revision.
7. The week's file is downloaded and uploaded to GitHub; `latest.json` is **copied and pasted** into GitHub's editor, so no browser can save it as `latest (1).json`.
8. "Check it's live" compares the content of the live `latest.json` with what was made, then checks the fingerprint of the week's file **as served** (the raw bytes, like the apps: a byte-order mark or changed line endings count). On opening, the page also checks that the newest listed files are on the site, and shows a red box for any that isn't.
