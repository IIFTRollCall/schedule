# Exam Schedule → JSON — Conversion Prompt v1.0 (Roll Call)

> You will be given **an IIFT exam schedule** (mid-term or end-term) as it arrived —
> an email body, a PDF, or a screenshot — plus **the latest weekly-schedule JSON**
> (the roster; mandatory). Output **only** the JSON object the Roll Call app
> imports — no prose, no markdown fences.
>
> This is the small cousin of the weekly conversion prompt. Same schema, same app,
> three big differences: sessions are `type:"exam"`, there is **no `week` object**,
> and labels carry **no numbers**.

## Rules

1. **Roster first.** Copy the weekly JSON's `courses[]` array **verbatim** —
   codes, names, credits, faculty spellings, elective flags, all of it, including
   courses with no exam. Never re-transcribe names from the mail.
2. **Omit `week` entirely.** The app derives a stable pseudo-week from the exam
   dates; inventing a week number would collide with a real weekly sheet and
   replace its classes.
3. **One session per exam**, `type:"exam"`:
   - `courseCode` — roster code.
   - `date` zero-padded `YYYY-MM-DD`; `start`/`end` 24-hour `HH:MM` (a "2 hrs"
     duration with no end time → keep `start`, set `durationOverrideMins": 120`).
   - `section` — exams are almost always whole-batch: `"section": "*"`. Only emit
     per-section values if the mail explicitly splits sections.
   - `faculty` — the invigilator/instructor **only if** the mail names one AND the
     name is in the roster (copy the roster spelling); otherwise `""`.
   - `room` if stated, else null.
   - **`sessionLabel`: `"<CODE> Mid-Term (A+B+C)"` or `"<CODE> End-Term (A+B+C)"`
     — the audience in parens at the very end, and NO digits anywhere in the
     label.** (Digits would be mis-read as a session number; a date/time keys an
     exam perfectly on its own.)
4. **Skip an exam the weekly sheet already lists.** If a weekly JSON already
   carried this exact exam (same course, same date), leave it out here — a second
   copy under a different label would duplicate it in the app.
5. `trimester` — exact, from the mail/roster (the app uses it to pick the trimester). Keep each exam's `sessionLabel` identical in a revised exam sheet (e.g. `"ITLS End-Term (A+B+C)"`) — the app identifies an exam by its label, so a moved exam replaces the old one instead of appearing twice.
   `schemaVersion:"1.2"`, `kind:"iift-weekly-schedule"` — unchanged (the app
   accepts exam sessions through the same importer).
6. Anything unreadable or ambiguous → transcribe your best guess **and** describe
   it in a top-level `review` array. Never silently guess.

## Output shape

```json
{
  "schemaVersion": "1.2",
  "kind": "iift-weekly-schedule",
  "programme": "MBA (IB) 2025-27",
  "campus": "Kolkata",
  "trimester": "IV",
  "sections": ["A", "B", "C"],
  "courses": [ /* copied verbatim from the weekly JSON */ ],
  "sessions": [
    { "section": "*", "date": "2026-09-14", "start": "10:00", "end": "12:00",
      "courseCode": "ITLS", "faculty": "", "room": "Auditorium",
      "sessionLabel": "ITLS End-Term (A+B+C)", "type": "exam",
      "durationOverrideMins": null, "notes": "" }
  ],
  "review": []
}
```

## Self-check before you output
- [ ] `courses[]` is byte-identical to the weekly JSON's.
- [ ] No `week` object anywhere.
- [ ] Every session: `type:"exam"`, roster `courseCode`, zero-padded date, 24-h times.
- [ ] Labels end with `(A+B+C)` (or the stated sections) and contain **no digits**.
- [ ] Exams already on a weekly sheet are omitted.
- [ ] `trimester` matches the roster's.

*(What the app does with this: each exam lands on the calendar and the subject's
"quizzes & exams" list, gets a night-before notification, and automatically
creates a countdown reminder in the reminders tab that completes itself when the
exam is marked.)*
