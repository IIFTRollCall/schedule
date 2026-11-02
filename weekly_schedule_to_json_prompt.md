# Weekly Schedule → JSON — Conversion Prompt v2.1 (Roll Call)

> You will be given up to **three files**:
> 1. **This week's IIFT class schedule as an Excel file (.xlsx)** — the primary source (the schedule is authored in Excel and exported to PDF, so the .xlsx carries the real grid structure).
> 2. Usually the **PDF** of the same sheet — a secondary cross-check only.
> 3. **Last week's JSON** (the previous output of this prompt) — **the name-and-code roster. When it is attached, it is mandatory input**, not context.
>
> Output **only** the JSON object the Roll Call app imports — no prose, no markdown fences.
>
> **v2.0 changes:** last week's JSON is now a required input and the sole authority for professor-name spellings and course codes (§4); a new §5 explains exactly which fields the app uses as *identity* to carry a student's attendance marks across re-imports and revised sheets — and therefore must stay stable; consolidated rules for revised sheets (§12); expanded self-check.
>
> **v2.1 changes** (matches Roll Call Android 4.3.1 and the iPhone app 1.2): students now **see** `changes[]` (as "notes from the sheet" in the import screen) and each session's `notes` (as the class note and in the calendar event), so both must be written for students (§8, §11, §14). Exams get a fixed label, because the app keys them by name (§5, §11). Feedback labels follow the §9 format. Quizzes must always be numbered (§5). A trimester mismatch no longer blocks the import: the app offers to move the week into the right trimester (§6).

---

## 0. How to read the sheet — Excel first, PDF only as a cursory check
**Read the structure from the .xlsx. It is deterministic — do not visually parse pixels.** The workbook gives you everything directly:
- **A slot-header row** of time ranges (e.g. "9:00 AM to 10:30 AM"), and **a section row** that labels every column `Sec-A / Sec-B / Sec-C` repeated per slot. Read both to map each column → (slot, section). **Don't hardcode** — slot count and times vary week to week.
- **Day rows**, each with a date like `29-06-2026 (Monday)`.
- **The merged-cell range over a class cell *is* the audience.** Collect the section columns the merge spans:
  - one section column → single section (`A`/`B`/`C`);
  - two adjacent (Sec-A+Sec-B or Sec-B+Sec-C) → that split half (`A+B` / `B+C`);
  - all three → `A+B+C` (emit `section:"*"`).
  An explicit in-cell note like `(Sec - B&C)` overrides the column span.
- **Times** come from the slot-header row; a per-cell override like `2 hrs (12:30 - 2:30 PM)` sets that session's `start`/`end`.
- **The footer table** gives each course's name + bracket code, coordinator(s), the CREDIT column, and the CORE/ELECTIVE grouping (usually a merged label spanning the core rows vs the elective rows). Read `elective` from there.

This is a table read, not an investigation — no rasterizing, no zooming, no gridline detection.

**Then do ONE cursory pass over the PDF — and stop.** Glance at it only to catch a gross discrepancy (a day/course clearly on the PDF but missing from your Excel-derived output, or a merge that plainly looks different). Hard limits:
- **Do NOT** re-parse the PDF, re-derive audiences from the image, zoom cells, crop, or detect gridlines. The Excel is authoritative.
- Budget **one or two glances total**. If something looks off, note it in the `review` array and keep the Excel value (it's almost certainly right). Then emit the JSON.

*(Fallback: if only a PDF/image is provided and no Excel, render the page once at ~200 DPI, read it in a single pass, and zoom at most ~6 ambiguous cells — never more.)*

---

You convert **one weekly IIFT MBA (IB) class schedule** into a single JSON object. The schedule is a batch-level grid; you transcribe it faithfully and completely. You are **not** filtering to any one student — emit every class on the sheet for every section; the app projects each student's own timetable from your output.

## 1. Hard rules
1. **Copy, don't compose.** Professor names and course codes come **verbatim from last week's JSON** (§4). The grid structure comes from the Excel. The only thing you ever author fresh is this week's dates, times, rooms, and session numbers as printed.
2. **Take the structure from the Excel.** Column position (which section) and merged-cell ranges (the audience) come straight from the workbook — don't re-derive them from the PDF image.
3. **Never hardcode slot times.** Read the slot-header row each time; slot count and times vary every week (5 or 6 slots, sometimes an evening slot, a LUNCH gap).
4. **The footer is the authority for the course *set*** — which courses run, credits, CORE/ELECTIVE status. But spellings of names/codes defer to last week's JSON (§4). Course codes are reused across trimesters for *different* courses — use **this trimester's** roster only.
5. **Every `sessions[]` entry must carry a `courseCode` that exists in `courses[]`.** The importer silently drops sessions without a matching course. (Consequence: see §11 on course-less events.)
6. **When you genuinely cannot tell** something (is a digit the session number? does an annotation override the span?), do **not** guess — record it in the top-level `review` array and make your best transcription.

## 2. The grid model
- Rows = **dated days** (e.g. `02-02-2026 (Monday)`); weeks can include Sat/Sun.
- Columns = **time slots** (read the header).
- Each time slot is split into **3 section sub-columns: Sec-A | Sec-B | Sec-C**.
- A **Classroom header row** (just under the section row) may give each section's default room (A→Room-8, etc.); cells inherit it unless they name their own room.
- A non-empty cell reads like `IFM (JKS) 9` → course `IFM`, faculty initials `(JKS)`, **session number `9`** (the number may sit mid-label, e.g. `MM 14 (SB) 7` → number is **7**).

## 3. Output schema (emit exactly these keys)
```json
{
  "schemaVersion": "1.2",
  "kind": "iift-weekly-schedule",
  "programme": "MBA (IB) 2025-27",
  "campus": "Kolkata",
  "trimester": "IV",
  "week": { "number": 1, "revision": 1, "startDate": "2026-06-29", "endDate": "2026-07-05", "issuedLabel": "WEEK - 1" },
  "sections": ["A", "B", "C"],
  "courses": [
    { "code": "ITLS", "name": "International Trade Logistics & Sourcing", "credits": 3, "faculty": ["Deepankar Sinha", "Raghuveer Negi"], "elective": false }
  ],
  "sessions": [
    { "section": "B", "date": "2026-06-30", "start": "16:30", "end": "18:00", "courseCode": "ITLS", "faculty": "Raghuveer Negi", "room": "Room-6", "sessionLabel": "ITLS 1 (B)", "type": "lecture", "durationOverrideMins": null, "notes": "" }
  ],
  "review": [],
  "changes": []
}
```
`kind` must be exactly `"iift-weekly-schedule"` and `schemaVersion` must stay `"1.2"` — the app rejects the file otherwise.

## 4. Names & codes — the roster rule (the #1 source of past data corruption)
Professor names respelled week to week ("Deepankar Sinha" → "Dr. Deepankar Sinha" → "D. Sinha") used to create duplicate professors in the app and detach students' attendance marks. The fix is mechanical:

- **When last week's JSON is attached, its `courses[]` array is the roster.** Copy each course's `code`, `name`, and `faculty` array **character-for-character**. Do not re-transcribe them from the sheet, do not add titles (Dr./Prof./Mr.), do not abbreviate, do not "correct" spelling.
- **Every session's `faculty` value must be an exact, character-for-character copy of one roster entry.** Resolve the cell's initials (e.g. `(RN)`) to the roster name, not to your own rendering of the footer.
- The footer may only **add**: a genuinely new course or a genuinely new professor gets added exactly as the footer prints it (full name, `*` guest marker stripped, no titles). Note the addition in `changes`.
- **Course codes must be identical week to week.** The app matches subjects by code; a drifted code (`ITLS` → `ITL&S`) creates a duplicate subject. If the sheet itself prints a changed code, keep last week's code and flag it in `review`.
- If no previous JSON is attached (week 1), build the roster from the footer: full names exactly as printed, `*` stripped, no titles — and keep that file as the roster for every following week.

## 5. What the app keys marks on — stability requirements
The app identifies a session across imports by **(type, audience, session number, professor)** — taken from your `type`, the label's `(audience)`, the label's number, and `faculty`. A student's P/A marks ride on that identity. Therefore, across revisions of the same week:

- **Same session ⇒ same `sessionLabel` number, same `faculty` spelling, same audience, same `type`.** A reschedule moves date/time only (§11) — the identity stays put and the mark follows automatically.
- **Never renumber sessions yourself.** Transcribe the number the sheet prints, even if the sequence looks odd (multi-professor courses number per professor; gaps happen when classes are cancelled). "Fixing" the numbering detaches marks.
- **Quizzes are their own number line.** `ITLS Quiz 1 (B)` and lecture `ITLS 1 (B)` are different things — always keep `type:"quiz"` and the word `Quiz` in the label so they can't collide.
- **Exams (mid-term, end-term) carry no number, so the app keys them by their label's name** (the label minus the `(audience)`). Always use exactly `<CODE> Mid-Term (<AUDIENCE>)` / `<CODE> End-Term (<AUDIENCE>)` and keep it identical across revisions, so a moved exam keeps its mark and its reminder.
- **Always number quizzes** (`<CODE> Quiz 1 (B)`, never `<CODE> Quiz (B)`). A session with no number (an unnumbered quiz, an event, a feedback marker) is keyed by date + time + professor instead, so if it moves, the student's mark does not follow it.
- A wrong `week.number` makes the app replace the wrong week's classes wholesale — see §12.

## 6. `trimester` — must be exact (the app enforces it)
Emit the trimester this sheet belongs to, as a roman numeral (`"I"`…`"VI"`) or a number. `"IV"`, `"4"` and `"Trimester IV"` all work. The app compares it with the student's current trimester and, if they differ, offers to switch to (or restore, or start) the trimester the sheet names. So a wrong value sends the whole week into the wrong trimester: read it off the sheet header/footer carefully; don't guess.

## 7. `courses[]` — one per footer row (the **full trimester basket**, not just this week's)
List **every course in the footer**, whether or not it has a class this week. **One entry per course code:** if a split-faculty core is printed as two footer rows with the same code, emit a single course whose `faculty` lists both professors. The app needs the full set: cores become trackable subjects even with no class this week, and the complete elective list drives the opt-in prompt. (Sessions in `sessions[]` are still only for what actually meets this week.)
`code` (roster/bracket code; if none, a short uppercase code from the name — then keep it forever), `name` (roster spelling), `credits` (read the CREDIT column — **don't assume; one course is often 3 while the rest are 2**), `faculty` (roster spellings; §4), **`elective`** (true if the footer groups it under ELECTIVE/FUNCTIONAL, else false).

> **`elective` drives an in-app prompt.** A new elective makes the app ask the student "are you taking this?"; a core is added automatically. A core mis-tagged `elective:true` wrongly prompts; an elective mis-tagged `false` is force-added without asking. Get the CORE/ELECTIVE split right from the footer. (Language electives — French/Spanish — are electives.)

## 8. `sessions[]` — one per scheduled class cell
- **`section` OR `sections`** (this drives the audience — get it right):
  | What the cell shows | Emit | Label audience |
  |---|---|---|
  | one sub-column (A only / B only / C only) | `"section": "A"` (or "B"/"C") | `(A)` / `(B)` / `(C)` |
  | two merged sub-columns A+B | `"sections": ["A","B"]` | `(A+B)` |
  | two merged sub-columns B+C | `"sections": ["B","C"]` | `(B+C)` |
  | all three merged | `"section": "*"` | `(A+B+C)` |
- `date` ISO **zero-padded** `YYYY-MM-DD` (`2026-07-06`, never `2026-7-6`), inside `week.startDate..endDate`; null only for an undated special.
- `start`/`end` **24-hour `"HH:MM"`** (convert any AM/PM or dotted times; null if timeless). If a cell states a duration instead of an end time, keep the slot `start`, set `durationOverrideMins`, and leave `end` null — the app computes it.
- `courseCode` (must match a `courses[]` code — roster spelling, §4).
- `faculty` — exact roster spelling (§4); if two faculty teach different sittings, attach the right one per cell.
- `room` (cell override, else the Classroom-row default; else null).
- **`sessionLabel`** — see §9 (the app parses the session number and audience from it; the format must be exact).
- `type` one of `lecture | quiz | exam | feedback | event` (default `lecture`).
- `notes`: **students see this.** It becomes the class's note in the app and goes into the iPhone calendar event. Leave it `""` unless it helps a student (e.g. `"bring laptops"`, `"moved from Mon 9:00"`). Transcription remarks (an in-cell `(Sec - B&C)` annotation, a doubtful digit) go in `review`, never here.

## 9. The self-describing `sessionLabel` (format matters — the app parses it)
Format: **`<CODE> <NUMBER> (<AUDIENCE>)`** — the session **number must sit immediately before the `(audience)`**, the label must **end** with the `(audience)`, and audience ∈ `A | B | C | A+B | B+C | A+B+C`.
Examples: `"ITLS 1 (B)"`, `"MC 1 (A+B)"`, `"MSS 1 (A+B+C)"`, `"SCM 2 (A)"`, quiz: `"ITLS Quiz 1 (B)"`.
- The `(audience)` in the label **must match** the `section`/`sections` you set.
- If the sheet embeds the number mid-text (`MM 14 (SB) 7`), still emit the label as `"MM 7 (A+B)"` etc. — number before the audience paren, nothing after it.
- Never append anything after the `(audience)` — trailing text breaks the app's number parse.

## 10. The audience / grouping decision tree (the crux)
For each course, two merged-column situations look identical but mean different things. Decide from the **footer + the cell faculty initials**:

1. **`A&B / C` split-faculty CORE** (footer says e.g. `OM … [Sec - A & B] and Dr. … [Sec - C]`; the two sittings have **different faculty**). Emit the A&B sitting as `sections:["A","B"]` and the C sitting as `section:"C"`. **There is no B+C sitting.** Different `faculty` per sitting.
2. **`A+B / B+C` low-strength split** (electives, and some cores like MC/SHRM/DM/FinTech/M&A — typically same faculty, the batch is just split in two). The **same course + same session number appears twice**: emit one `sections:["A","B"]` and one `sections:["B","C"]`.
3. **All three merged** → `section:"*"`, label `(A+B+C)`.
4. **Single section** → `section:"A"/"B"/"C"`.
5. An explicit cell annotation (e.g. `(Sec - B&C)`) **overrides** the visual column — set `sections` to match it and record the override in `review`.

> You emit **both** halves of a split (1 and 2 above) exactly as drawn. You never pick a half "for the student" — the app does that per student. If you can't tell which scheme a merge belongs to, transcribe it and add a `review` note.

## 11. Annotation grammar → JSON
| On the sheet | Do this |
|---|---|
| `(cancel)` (e.g. `OM (ST) 7 (cancel)`) | **Omit that session** from `sessions[]` (a cancelled class must vanish from the denominator). Add it to `changes` (e.g. `"OM 7 cancelled"`). |
| `(resche)` (e.g. `IE (RB) 5 (resche)`) | Emit the session at its **new** date/time with the **same courseCode, same audience, same session number, same faculty spelling** (so the student's existing mark survives the move). Don't emit the old slot. Add to `changes`. |
| `(8:30-10)`, `(7:00 PM-11:00 PM)`, `(9-10:30)` | Per-cell time override → set `start`/`end` in 24-hour. A bare duration like `2 hrs` → keep slot `start`, set `durationOverrideMins`. |
| `Room-6`, a standalone room line above the text | Per-cell `room` override. |
| `X`, or a blank cell | Emit **nothing**. |
| `Quiz-N : COURSE (dur)` (often `Auditorium`) | `type:"quiz"`, `courseCode` = that course, `room:"Auditorium"` if shown, `sessionLabel:"<CODE> Quiz N (<AUDIENCE>)"`. Keep the slot's date/time. |
| Mid-term / end-term **exam** | `type:"exam"`, `courseCode` = that course, label exactly `"<CODE> Mid-Term (<AUDIENCE>)"` or `"<CODE> End-Term (<AUDIENCE>)"` (no number; the name is its identity, §5), `room` if shown, `start`/`end` from the cell. |
| Mid-term / end-term **feedback** marker | `type:"feedback"`, attach to the course, label `"<CODE> Mid-Term Feedback (<AUDIENCE>)"` or `"<CODE> End-Term Feedback (<AUDIENCE>)"`. Keep the word `End` for end-term: the app shows end-term feedback as a mandatory to-do. |
| One-time event (`"POSH" Workshop`, `Trimester Registration`) | `type:"event"` **only if** it's tied to a course code; **a course-less event has no place to attach and the importer will drop it**. Instead list it in `changes` as a note students can use, with what, when, where and who (e.g. `"POSH workshop: Fri 10 Jul, 19:00, Auditorium (all sections)"`). Students see these lines in the import screen. |
| `* Guest Faculty` footnote | Strip the `*` from stored names. |

## 12. Revised / re-issued sheets
When the administration re-issues a week (a "Revised" sheet):
- **Keep the SAME `week.number`** as the original sheet and **increment `week.revision`**. The app replaces exactly that week's imported classes on re-import; a wrong number wholesale-replaces a *different* week's data.
- Never invent a week number. If the sheet doesn't print one, omit `week.number` entirely (the app derives a stable one from the dates) — but never guess.
- Keep every unchanged session **byte-identical** to the previous file (same label, number, faculty, audience, type). Only the genuinely changed cells should differ.
- Cancelled sessions are omitted; rescheduled ones move date/time with identity intact (§11); describe every difference in `changes[]`.

## 13. Self-check before you output
- [ ] You took the grid from the **Excel** (merges = audience) and used the PDF only as a quick cross-check — no deep image parsing.
- [ ] **Every `sessions[].faculty` value appears character-for-character in that course's `courses[].faculty`.**
- [ ] **`courses[]` codes, names, and faculty are byte-identical to last week's JSON** (additions from the footer only, noted in `changes`).
- [ ] `trimester` is read off the sheet and correct (the app blocks mismatches).
- [ ] `week.number` matches the sheet header; a re-issued sheet keeps the number and bumps `revision`.
- [ ] Every `sessions[]` entry has a `courseCode` that exists in `courses[]`.
- [ ] Each session's `section`/`sections` **matches** its `sessionLabel` audience, and every label **ends** with the `(audience)`.
- [ ] Session numbers are transcribed as printed — never renumbered; quizzes labelled `<CODE> Quiz N (…)` with `type:"quiz"`.
- [ ] `elective` per course is correct per the footer (it drives the opt-in prompt).
- [ ] Split courses (A+B/B+C) have **both** halves; A&B/C cores have an A+B sitting + a C sitting (no B+C).
- [ ] All dates zero-padded ISO inside the week range; all times 24-hour `HH:MM`; cancelled sessions omitted (and in `changes`); rescheduled ones moved with identity intact.
- [ ] Session count matches the sheet (count the non-empty cells).
- [ ] `credits` read off the footer column per course.
- [ ] Anything uncertain is in `review` — no guesses.
- [ ] Every quiz label carries a number; every exam label is exactly `<CODE> Mid-Term (…)` / `<CODE> End-Term (…)`; feedback labels end with the `(audience)`.
- [ ] `notes` are empty or student-useful (no transcription remarks); every `changes` entry is a short plain sentence a student can act on.

## 14. Output
Return **only** the JSON object. No commentary, no code fences.
- `review` is for you and the person running this prompt; the app ignores it.
- **`changes` is shown to every student** in the import screen as "notes from the sheet" (the first 8 entries). Make each entry a plain string, one change or notice per entry, written for a student: `"OM 7 (C) cancelled"`, `"IE 5 (B) moved to Thu 16 Jul, 11:00"`, `"POSH workshop: Fri 10 Jul, 19:00, Auditorium (all sections)"`. No internal remarks.
