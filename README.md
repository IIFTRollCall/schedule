# Roll Call schedule feed

This repository is where the official IIFT weekly timetable reaches every Roll Call app (Android 4.4.2+, iPhone 1.3.2+). The apps read `latest.json` from

**https://iiftrollcall.github.io/schedule/latest.json**

and import the files it lists. Nobody forwards JSON files on WhatsApp any more.

> **Never rename** the `iiftrollcall` organization, this `schedule` repository, or its `main` branch, and never turn GitHub Pages off. That address is built into every installed app.

---

## Once, when the repository is set up

In the repository click **Add file → Create new file**, name it `.nojekyll` (with the dot), leave it empty and click **Commit changes**. The site then updates faster after each upload, and editing a `.md` file can never break it.

## Every week (about 5 minutes after the Claude step)

**First, make the JSON in Claude, as before.** Paste `weekly_schedule_to_json_prompt.md` with this week's .xlsx, the PDF and **last week's JSON**. To get last week's JSON, open the publisher page and click **download it** next to "newest".

Then open the publisher page: https://iiftrollcall.github.io/schedule/publisher.html (always use this one: a downloaded copy can go out of date). Its steps are numbered, and the numbers below are the same.

1. **The new schedule.** Drop the JSON in and read what it says:
   - the **summary**: week, dates, classes per section;
   - the **checks**: a red box stops you; a yellow box is worth a look ("claude wasn't sure: ..." lines are Claude's own doubts); a yellow box with a checkbox needs you to tick it; a yellow box with a text box needs you to type the number it asks for;
   - for a revised sheet, **what changed from the published version**;
   - the **notes students will see**: every student sees the first 8, so make sure they read well.

   Optionally type a short note for this update ("fri slot moved to 11:00"). Students see it with the update.
2. **Make the files.** Click **make the files**.
3. **Upload the week's file.** Click **download T5-W08-r1.json** (the name changes each week, and the page skips any name already on the site), open the **upload page** link, drag the file in and click **Commit changes**. The name must be exactly what the page shows: if your browser saved it as `T5-W08-r1 (1).json`, rename it first.
4. **Replace latest.json.** Click **copy latest.json**, open the **latest.json on github** link, click in the text, press **ctrl+a** (cmd+a on a Mac) and **ctrl+v**, then click **Commit changes** twice. (The very first time this step is called **create latest.json**, and the link opens **a new file called latest.json**: just paste.)
5. **Check it's live.** Click **check it's live**. It says "saved on github ✓" first, then **live ✓** when the site has both latest.json and the week's file (1 to 5 minutes), and the steps fold away. You're done. Until then the page won't make new files.

Optional: post one line in the class group, for example "week 8 is out, open roll call".

If you close the publisher page halfway, open it again: it remembers the files it made and picks up from there. It won't make new files until all of them are on the site; if you never uploaded them, click **forget these files**.

Each time it opens, the page also checks that the newest files latest.json lists are really on the site. A red box at the top means one isn't: see the table below.

**A revised sheet in the middle of the week:** the same steps. The publisher names it `-r2`, `-r3` by itself, and students see what changed.

**Exam schedules** (sent by mail, no week number): convert them with `exam_schedule_to_json_prompt.md` and publish them the same way. The publisher asks what the sheet is: **mid-term exams**, **end-term exams** or **something else** (a one-off extra sheet). It never picks an exam schedule that is already published for you: to revise one, click **mid-term exams** or **end-term exams** yourself. If the revision takes exams or quizzes away, it asks you to tick that this is right. A sheet with ordinary classes in it starts on **something else**: it's probably a week whose number Claude left out, so ask Claude to add it.

**A new trimester:** publish its first sheet as usual. The publisher asks you to tick "this starts trimester N". Each student's app then asks before switching trimester.

**A sheet for an older trimester** than the feed's is usually a mistake in its "trimester": ask Claude to fix it. It is right only in two cases: a new batch's first trimester after the old batch's last (trimester 4 after 6), or undoing a mistaken "starts trimester 6". Then type the trimester's number in the box to publish it. The newer trimester's files leave the list. Students whose app already moved to the newer trimester don't get the sheet by themselves: send it in the class group and ask them to import it by hand.

## If the publisher stops you

| it says | do this |
|---|---|
| "don't overlap the published week N" | Claude kept last week's number. Ask Claude to fix the week number. |
| "overlap the published week N" | This is last week's sheet (or last week's dates) with a new week number. Ask Claude again with this week's .xlsx and PDF. |
| "classes are dated in the published week N" | Some classes in this sheet are dated in another week. Tick only if that's right; otherwise ask Claude to fix the dates. |
| "the sheet's trimester ... is too long" | The apps read only the first 40 characters. Ask Claude to write just the trimester, like "V". |
| "older than the feed's trimester" | See "A sheet for an older trimester" above. Type the number only for a new batch or to undo a mistaken switch. |
| "takes away an exam or quiz" | Tick only if those exams or quizzes really were cancelled. Otherwise ask Claude for the full exam sheet. |
| "github's own copy of latest.json couldn't be read" | GitHub lets the page read it 60 times an hour, or your network blocks it. Click **try again** in a few minutes. Tick the box only if you're sure nobody published from another computer or browser in the last 10 minutes. |
| "isn't on the site as latest.json lists it" (red box at the top) | That file was never uploaded, or went up under another name. Upload the copy you downloaded with exactly the name shown. No copy? Drop that sheet's JSON in step 1 and click **download ... again**. |
| "doesn't list ... was never published" | Someone published from another computer or browser on top of an older list, so your files never went out. Drop your JSON in step 1 again and make new files. |
| "written as 9.3 without quotes" | Ask Claude to put every time, date and room in quotes. |
| "exactly the file already published" | Nothing to do: it's already out. |
| "the files made here at ... aren't on github yet" or "aren't all on the site yet" | Finish steps 3 to 5 on the page for those files, or click **forget these files** if you never uploaded them. |
| "couldn't reach github" | Check your internet and click **try again**. Never start over: wait until GitHub can be reached. |
| "isn't on the site" | latest.json went up but the week's file didn't, or went up under another name. Upload it with exactly the name shown, then click **check it's live** again. "as made here" means a changed copy went up (opened and saved in another app): upload the downloaded file as it is. |
| "the paste went wrong" | Open latest.json on GitHub again, select everything, paste and click **Commit changes**. |
| "can't be built on" | latest.json on GitHub is broken. Load a good copy (see "Fixing a mistake"). |
| "newer than this copy of the publisher page" | You're using a downloaded copy. Open the page on the site instead. |
| "already published as" | That exact file is already up in another slot. If that slot was wrong, take it off the list under "other feed settings" first. |
| "the live feed says it has moved" or "impossible sequence number" | Someone edited latest.json by hand. Ask the other owner what happened before publishing; see "Fixing a mistake". |

## Fixing a mistake

Publish forward: correct the JSON and publish it again; it becomes the next revision. **Don't** restore an old `latest.json` from the history: the apps ignore any `latest.json` older than one they've already seen, and the next publish would build on the old list. If something looks badly wrong, ask in the class group for students to hold on, then publish the fix.

A file published in the wrong slot can be taken off the list under "other feed settings". That stops new installs getting it; students who already have it keep its classes until a corrected sheet replaces them.

If the account was ever taken over: after getting it back (and changing passwords), open the publisher. Check the message for everyone, the android version and the files listed under "other feed settings", and clear or take off anything you didn't publish, then make latest.json. If the feed is broken, the publisher asks you to load a good copy instead: open latest.json in the repository, click **History**, open the last good version, download it, and load it in the publisher. The new latest.json is then numbered above the broken one (when its number can still be read), so phones that already took the bad one accept the fix.

## Other things the publisher page can do (under "other feed settings")

- **Message for everyone:** one line on every student's Home until a date you pick (a week ahead unless you change it), for example "no classes on monday 20 oct (diwali)". Click **make latest.json with only these changes**, then do the page's steps 3 and 4 (replace latest.json, check it's live). Changing only the date doesn't show it again to students who dismissed it.
- **Android app version:** when a new APK is out, fill in its version, code and a link on the club's GitHub. Whoever built the app gives you the version and code. Older APKs then show "roll call x.y.z is out". The page refuses a link anywhere else, or one with "%" or ".." in it (the apps would ignore it).
- **Take a file off the list:** see "Fixing a mistake".

## Files here

| file | what it is |
|---|---|
| `latest.json` | the list of current files (made by the publisher; never edit by hand) |
| `T5-W07-r1.json`… | the published weekly sheets (never edited; a revision is a new file) |
| `publisher.html` | the publisher page (use it on the site; a downloaded copy works but can go out of date) |
| `.nojekyll` | an empty file that makes the site update faster (see "Once") |
| `weekly_schedule_to_json_prompt.md` | the Claude prompt for weekly sheets |
| `exam_schedule_to_json_prompt.md` | the Claude prompt for exam schedules |
| `FEED_FORMAT.md` | the exact format the apps read (for whoever changes the apps) |

Old weeks can stay. Each file is about 50 KB.

## Who owns this, and the yearly handover

- The **organization** owns the repository, not a person. Keep **at least two owners**, each signing in with their own GitHub account and two-factor login switched on: the maintainer, the successor, and ideally a faculty adviser or alum as a long-term owner.
- Turn on two-factor login for everyone: Organization **Settings → Authentication security → Require two-factor authentication**.
- **Handover, in the last trimester:**
  1. Invite the successor as an **Owner** (Organization → People → Invite).
  2. The successor publishes two real weeks while you watch, including one revision.
  3. Walk them through this README and the publisher page.
  4. Before you leave, remove yourself (or stay on as an alum owner if you're happy to).
- **If the maintainer disappears:** any other owner adds a new maintainer the same day. Until then, the apps keep the timetable they have, and students can still import files by hand.
