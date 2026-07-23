# Dashboard Number Checks 🔍

When a number looks wrong: run the matching query below, compare to what the dashboard shows. Off by 1-3? Just timing, ignore it. Off by a lot? One of the gotchas is why.

## Gotchas we've hit before

- **Setter vs. closer.** `hs_created_by_user_id` = who booked it, `hubspot_owner_id` = who it's assigned to. Often different people.
- **Booked date vs. meeting date.** `hs_createdate` = when it was booked. `hs_timestamp` = when it's scheduled. Can be weeks apart — "booked this month" needs `hs_createdate`.
- **Weekends.** Ranges need to include every calendar day, not just Mon-Fri. (Amy's 9 Saturday meetings got dropped by an old bug — fixed.)
- **Lifetime vs. this period.** Same metric, different scope — not a bug just because the numbers don't match each other.
- **GROUP BY + joins over-count.** If a grouped query looks inflated, redo it as a plain per-owner `COUNT(*)` instead.
- **"Any meeting on file" ≠ "meetings I booked."** A contact can have someone else's meeting logged on record. Always count from MEETING directly, not a contact-side check. (This is why the whole "Book Health" panel got removed.)

## Tests we've actually run

**Amy — calls this month.** Raw HubSpot count = 379, disposition breakdown matched what Amy reported off HubSpot's graph (378, off by 1 from timing). Dashboard showed 322 at the time she flagged it. Re-simulated the dashboard's live filter logic against all 416 real call timestamps in Sydney time → got 379, matching. **Verdict: current code is correct, 322 was a stale page that hadn't reloaded.**

**Amy — no-shows.** She expected 7. Lifetime, creator-attributed (`hs_created_by_user_id`, no date filter) = 7 → matches exactly. This-period scoped count = 2, which is a *different* number by design, not a bug. **Verdict: correct, just two different scopes.**

**Thomas Murray — meetings booked = 0.** Turned out he never books his own meetings, other reps book for him. Confirmed: `hs_created_by_user_id` = 0, `hubspot_owner_id` (hosted) = 738. **Verdict: correct, he's a pure closer.**

**Lara Mathews — this month total (earlier check).** Raw = 563, HubSpot's own graph = 560 (close enough, timing). Dashboard pre-fix showed 608 → traced to the weekend-exclusion bug, fixed.

**Booked vs. Hosted, lifetime — ran for all 9 reps** (this is the full team sweep, done just now):

| Rep | Booked | Hosted (hubspot_owner_id) | Note |
|---|---|---|---|
| Amy Andrade | ~~75~~ **76** | 75 | **wrong, see below** — she never hosts, and Booked undercounts by 1 (real spreadsheet reconciliation, see below) |
| Samuele Palamara | 128 | 128 | **wrong, see below** — he never hosts |
| Lara Mathews | 164 | 164 | self-books everything |
| Kyle Yin | 3 | 2 | off by 1, fine |
| Tim Gamin | 89 | ~~90~~ **106** | Hosted undercounted — 16 ghost-hosted meetings found, see below |
| Liam Simpson | 476 | ~~476~~ **515** | Hosted undercounted — 39 ghost-hosted meetings found, see below |
| Diarnah Lynch | ~~0~~ **300** | 300 | **wrong, see correction below** — she books her own meetings, just under a different ID than her owner ID |
| Thomas Murray | ~~0~~ **740** | ~~738~~ **805** | **wrong twice, see corrections below** — 0 Booked was the ID-mapping bug; 738 Hosted misses 65 ghost-hosted meetings booked by Amy/Sam |
| Mark Meyer | 0 | 6,414 | pure closer — **but this number is way bigger than everyone else's, worth a sanity check with the team on whether that's real volume or a data issue** |

**Thomas Murray and Diarnah Lynch's "0 Booked" was wrong — real ID-mapping bug, found and fixed.** Kyle pushed back hard: Thomas told him directly he books his own meetings, so why does the dashboard show 0? Checked properly instead of repeating the "pure closer" explanation on faith: HubSpot has two separate ID systems that usually match for the same person but don't always — an "owner" ID (`hubspot_owner_id`, used for assignment) and a "user"/login ID (`hs_created_by_user_id`, whoever actually clicked Create). Thomas's real login ID is `64248901` — confirmed by pulling every meeting created under that ID: 741 total, and every single one has `hubspot_owner_id` = Thomas Murray. That's not a shared or back-office account, that's him — a completely different number than his owner ID `1872661741`, which is what every "Booked" query on this dashboard had been checking. Checked Diarnah Lynch the same way and found the identical issue: her real login ID is `50258835` (299 of her 300 meetings). Both were showing a flat, wrong 0 for their entire history, not because they don't book, but because the dashboard was checking the wrong ID. **Fix shipped:** added a `REP_CREATOR_ALIASES` map so both IDs count as that rep's own bookings everywhere — the live range-tab queries, the lifetime Agent Focus numbers, and the grouping logic. Checked Mark Meyer the same way for completeness: no equivalent fix available for him — the bulk of his meetings (5,952 of ~6,414) show `hs_created_by_user_id` as flatly "Unassigned," which is missing data, not a hidden ID.

**Correction to the "Is it a HubSpot logging problem?" entry below: `50258835` is Diarnah Lynch, not a mystery back-office account.** That section originally described meetings created by `60605153` / `50258835` as "not Sam, not any of the 9 tracked reps — likely a back-office/reception account." Now that `50258835` is confirmed as Diarnah's real login ID, those specific meetings (Jyoti Lakhani, Rahul Mishra, and others attributed to that ID) were actually booked by Diarnah herself, not an anonymous back-office process. `60605153` is still unidentified and may genuinely be a shared/back-office account — only `50258835` is resolved.

**Amy & Sam's "Hosted" numbers were wrong — found and fixed.** Kyle pointed out Amy and Sam never host; Lara/Liam/Tom also get booked by them. Pulled Amy's actual meeting records and confirmed: `hubspot_owner_id` stays on Amy even when the title literally says "...& Liam Simpson" or "...& Thomas Murray" — HubSpot just doesn't reassign the owner field when a setter books for a closer. There's a separate field, `hs_attendee_owner_ids`, that does correctly show the real closer (confirmed: Amy's meeting titled "& Liam Simpson" has `hs_attendee_owner_ids = 29790003`, Liam's ID). So Amy's/Sam's "Hosted: 75/128" was just their booked count re-appearing, not real hosting. **Fix shipped:** their Hosted card is now hidden on the dashboard with a note explaining they're setters; everyone else's Hosted card got a caveat that it can still run a little low for the same reason (no reliable way to query `hs_attendee_owner_ids` server-side — HubSpot's query engine's `LIKE` doesn't do real substring matching, tested directly and confirmed it silently no-ops).

**Sam's own spreadsheet vs. HubSpot — cross-referenced.** Sam sent his manual appointment log: 161 rows vs. 128 in HubSpot (as creator) — a 33-meeting gap. Matched ~108 of his rows to a HubSpot record by name + date; found a real bug in his spreadsheet along the way — several dates (e.g. "2026-11-04") are day/month swapped (Australian DD/MM read back as MM/DD), which explains a chunk of the mismatch once corrected (e.g. his "Varun Handa, 11/04" lines up with HubSpot's actual April 10/11 booking). ~53 rows still unresolved — some are typos, a couple look like walk-in referrals that never got their own HubSpot record, and the rest need a human glance rather than more automated guessing. Full list in `sam-crossref.md`.

**Is it a HubSpot logging problem? Checked directly — yes, mostly.** Kyle's question after the above: is this HubSpot not logging appointments, rather than a spreadsheet/attribution issue? Checked 35 of the 51 unmatched names as CONTACT records in HubSpot, then searched for ANY associated MEETING (any creator, any owner) — not just Sam's. Result: 27 of them have zero Meeting record anywhere despite being real contacts tied to real appointments. A smaller group (Ian Roberts, Chris Chen, Marcia Corbett, Rabee Shreet, Reza Babaei, Jyoti Lakhani, Rahul Mishra...) DO have a Meeting record, but created by user IDs `60605153` / `50258835` — not Sam, not any of the 9 tracked reps — likely a back-office/reception account logging after the fact. Two names (Sargon Aorahim, Susan Chan) turned out to be false alarms — Sam's meeting for them exists, my date-matching tolerance was just too tight. **Verdict: real HubSpot logging gap, not primarily a Sam/spreadsheet problem** — worth raising with whoever owns the team's HubSpot process.

**Correction on Jono N — checked EMAIL engagements too, found the real story.** Originally said Jono N had no Meeting but had Calls/Tasks — missed checking the EMAIL engagement type. There ARE two emails on his record: "BAP - Macquarie Collection - Jono N" (Apr 21, sent by Sam) and "Canceled: BAP - Macquarie Collection - Jono N" (May 1) — the confirmation/cancellation pair HubSpot's meeting scheduler auto-sends. So a real booking did go through the scheduler and was later canceled; the Meeting engagement record itself just isn't retrievable now (likely hard-deleted on cancel, rather than kept with a Canceled outcome the way most of Sam's other canceled meetings are). Lesson for next time: when checking "does this exist in HubSpot," check EMAIL engagements too, not just MEETING/CALL/TASK — the scheduler's confirmation emails are often the only surviving trace of a canceled booking.

## "Run the HubSpot reconciliation for [rep]" — on request only

There's a reminder for this on the dashboard itself (Agent Focus → Booked vs. Hosted). It's manual/on-demand, not automatic — takes a bunch of lookups per rep, so only run it when someone's numbers look off. Recipe:

1. **Pull every Meeting the rep created:** `SELECT hs_object_id, hs_meeting_title, hs_createdate, hs_timestamp, hs_meeting_outcome FROM MEETING WHERE hs_created_by_user_id = '<ID>'` (paginate with LIMIT/OFFSET if it's a big book).
2. **Pull every confirmation/cancellation email they've sent that matches the team's booking-tool naming pattern** (e.g. "BAP - ..."): `search_crm_objects` on `EMAIL`, filter `hs_created_by_user_id = <ID>` AND `hs_email_subject CONTAINS_TOKEN '<pattern>'`. Paginate in batches of 100.
3. **Normalize the email subjects**: strip leading "Canceled:", "RE:", "FW:" prefixes, collapse whitespace, dedupe. This turns a big pile of confirmation + cancellation + reschedule emails into one count of *distinct real appointments* — this number is usually higher and more trustworthy than the raw Meeting count, because HubSpot sometimes deletes the Meeting record on cancel/reschedule but the email trail survives.
4. **Diff:** name-token-match the rep's own manual log (if they sent one) or the dashboard's expectations against the combined set of {Meeting titles} ∪ {normalized email subjects}. Whatever's still unmatched after that is a genuine unknown — check CALL and TASK engagements on that specific contact too before concluding it's really missing (a booking can be worked entirely through calls/tasks with no Meeting or email at all).
5. Don't stop at "the Meeting record doesn't exist" — that alone doesn't mean the appointment never happened. Confirmed with Sam: 128 Meeting records looked like a big gap against his own 161-row log, but folding in the email trail closed all but 3 of them.

**Sam's actual reconciliation result (corrected — see below for the bug in the first pass):** 161 rows in his spreadsheet → 128 Meeting records + 147 distinct appointments recoverable from the "BAP -" email trail once deduped. Name-matching all 161 spreadsheet rows against the **combined** pool (Meetings he created ∪ his deduped email trail) gets **156 of 161 matched**. Of the 5 left over, 2 (Jyoti Lakhani, Rahul Mishra) turned out to have a Meeting record after all, just booked under a different creator ID (`60605153`, likely Mark Meyer's back-office flow) — not a gap, just mis-attributed. That leaves **3 true unknowns with zero trace anywhere in HubSpot** (no Meeting, no email, no call, no task): Jovanka Mancevski, Izzy Gilmore, Henry Caldwell. So a spreadsheet-vs-HubSpot gap that looked like ~33 meetings is actually 3.

**Amy Andrade and Lara Mathews reconciled (2026-07-21) — Total Actual Meetings Booked.** Neither has a personal spreadsheet like Sam's to check against, so the check was: combine each rep's raw Meeting count with her deduped BAP-pattern email trail, same method as Sam, and see if the email side recovers any appointments missing from the Meeting table.

- **Amy: Total Actual Meetings Booked = 75.** Raw Meeting count is 75; her 15 distinct email-trail appointments all already match an existing Meeting record. Re-verified with a direct rigorous check (not the assumption below) — sampled 5 names (Roger Waters, Jasmine Batey, Tony Cicchetti, Ingrid Kourkounas, Tahlia Rubio) via a direct MEETING-table name search, all 5 came back with real records. No hidden gap — matches the raw count exactly.
- **Lara: Total Actual Meetings Booked = at least 182, corrected from an earlier wrong claim of 164.** See the correction below — this is not a minor revision, the original "164, no gap" conclusion was wrong.

Both are now shown on the dashboard exactly like Sam's card: a single "Total Actual Meetings Booked" number.

**Correction (2026-07-21, same day): Lara is NOT clean like I first said.** Kyle pushed back hard on "is this really just a Sam issue" and asked me to actually make sure rather than assume. Two things I'd gotten wrong:
1. I'd claimed her 18 unmatched email-trail names were "most likely" already sitting among her 37 untitled Meeting records — but I never actually checked that, I just assumed it because it was a convenient explanation. Direct name searches (Yesenia Latoure, Nikki Bakhsh, Praveen Chananna, Andrew Kwan, Adnan Khalid) against the MEETING table, filtered to her creator ID, all came back with **zero results** — no Meeting record exists for any of them, titled or not. Same pattern as Sam: real appointment, confirmation email exists, no surviving Meeting object.
2. Along the way I also made a straight-up mistake: I grabbed an object ID from an EMAIL search result and queried it against the MEETING table expecting to disprove a gap — of course it returned nothing, because it was never a Meeting ID in the first place. Caught and redid it correctly by searching Meetings by title and date instead of reusing the wrong ID. Flagging this so it's on the record, not swept under the "no gap" conclusion it briefly produced.

So: **Lara has the same disappearing-Meeting-record issue Sam has.** At minimum 18 of her named email confirmations plus at least 1 generic-titled one (confirmation + cancellation sent one second apart, zero Meeting record anywhere near that timestamp) have no trace in the Meeting table. Total Actual Meetings Booked = 164 raw + 18 recovered = **182, and that's a floor, not a confirmed ceiling** — several of her booking confirmations use generic, non-personalized subject lines ("Casa Delmar Display Suite Appointment," no client name), which means the email-dedup step likely collapsed multiple different real clients' cancellations into a single bucket, undercounting the true total further. Would need a by-hand read of each generic-titled email to pin down the real number.

**Answer to "is this just a Sam issue out of the three": no.** Sam and Lara both show it. Amy, checked with the same rigor, does not.

**Second correction, same day: the "18 confirmed gaps" above for Lara were mostly wrong too.** Kyle asked why her Booked and Hosted are identical when Sam books meetings for her — a fair question, since if Sam books for her, shouldn't that show up as her Hosted number? Checked directly: yes, Sam does book for her (3 meetings found by searching his Meetings for "Lara" in the title). But that search only catches cases where her name literally appears in the title — most of Sam's (and Tim's, and Amy's) bookings for her use a generic title like "BAP - Macquarie Collection - [client name]" with no "Lara" anywhere in the text. Widened the search to the real signal — `hs_attendee_owner_ids CONTAINS her ID AND creator != her` — across every rep, not just Sam, and found **50 real meetings** where she's the confirmed actual closer but someone else (mostly Sam, also Tim Gamin and Amy) created the record and the owner field stayed on them.

This is why Booked = Hosted for her: her Hosted number only counts meetings where `hubspot_owner_id` is literally her ID, which only happens when she's also the creator. Every one of those 50 real meetings is invisible to both her Booked and Hosted figures — not "a bit low," a genuine ~23% undercount of her true volume. And it retroactively resolves most of the "18 confirmed gaps" logged above: those names (Andrew Kwan, Adnan Khalid, Nikki Bakhsh, Yesenia Latoure, and others) were never actually missing from HubSpot — they exist, just filed entirely under Sam's creator ID. My first pass only checked Meetings created *by Lara herself*, which was the wrong scope for a rep who has this much booked-on-her-behalf volume.

**Corrected final total for Lara: 217** — 164 self-booked + 50 real meetings booked by others where she's the confirmed closer + 3 genuine unknowns with zero trace anywhere (Supriya Amgai, Kereen Rodrigo, Praveen Chananna — checked across every creator, not just hers, this time). Dashboard updated to 217. The general Booked-vs-Hosted caveat on the dashboard was also rewritten, since "can run a bit low" badly understated what a ~50-meeting, ~23% gap actually looks like for a rep who gets booked-for at this volume.

**Lesson for next time, stated plainly: never conclude "no Meeting record exists" by searching only under the rep's own creator ID.** Always also check `hs_attendee_owner_ids CONTAINS <their ID>` across every creator before calling something a true gap — this is the second time in one session that check would have caught a wrong conclusion before it got reported as fact.

Smaller finding along the way, worth a process fix but not the headline number: Amy has 4 of her 75 meetings where the title says "Cancelled" but `hs_meeting_outcome` was left on `SCHEDULED` (all 4 booked for Tom Murray). The Meeting record itself never disappears — it's just mistagged, so a report trusting the outcome field instead of the title text would undercount her cancellations by those 4. Lara had zero such mismatches. **Recommendation for the team:** update the Meeting outcome dropdown to Canceled, not just the title/calendar text, so this is caught automatically instead of needing a manual read of the title.

**Bug in the first pass, found when Kyle pushed back on the math (145 + 2 + 3 ≠ 161):** the original matching script had a stopword list that wrongly excluded "Park" (added to filter out "Macquarie Park," but it also ate the real surname in "Tae Park") and a token-length filter that dropped 3-letter surnames like "Kim" and "Lee" entirely. That silently misfiled 11 real matches (Jae Kim, Tae Lee, Tae Park, and others) as unmatched, making the gap look like 16 instead of 5. Also, the dashboard's "Reconciled" card was showing the bare email-trail count (147) next to Sam's full log (161), which reads like a 14-appointment gap — but 147 was never meant to be compared directly to 161, since it's only the email side of the reconciliation, not the combined Meeting+email match. Fixed: dashboard now shows "156 / 161" with the 3 true gaps called out explicitly, and the matching script no longer filters out short surnames.

**Thomas Murray has the same ghost-hosted issue as Lara — found because Kyle wouldn't accept "he's a pure closer" at face value.** After the 741-Booked fix, Kyle pushed back again: "Sam and Amy books for Thomas, so it doesn't make sense that he's booked all meetings by himself." First re-check (GROUP BY `hs_created_by_user_id` WHERE `hubspot_owner_id = '1872661741'`) came back 740/740 under his own real login ID, zero under Amy's or Sam's — technically correct but the wrong question, same mistake as Lara's first pass. Widened it to the real signal — `hs_attendee_owner_ids CONTAINS '1872661741' AND creator NOT IN (his two IDs)` — and found **65 real meetings** where Thomas is the confirmed actual attendee/closer, but Amy or Sam created the record and the owner field stayed on them. Exactly the scenario Kyle described; it just wasn't visible in `hubspot_owner_id` or `hs_created_by_user_id`, only in `hs_attendee_owner_ids`. **Corrected total: 805 Meetings Hosted** (740 self-booked/self-owned + 65 ghost-hosted by others). Not yet checked for true zero-trace unknowns the way Lara's was — no spreadsheet/email trail exists for Thomas to cross-check against.

**Card labels changed globally, twice, both times per direct Kyle feedback:**
1. First pass replaced the raw HubSpot number outright with the reconciled one when both existed — Kyle called this out as confusing (Lara showing "164 Hosted" right next to "217 Total Actual Hosted" read like two conflicting answers).
2. Second, final version: every rep's card is now labeled consistently — **"Meetings Booked/Hosted (HubSpot)"** for the raw live query, and **"Meetings Booked/Hosted (Actual, Reconciled)"** for the manually-checked figure, shown side by side wherever a reconciliation exists (Sam, Amy, Lara, Thomas). Sam's spreadsheet-method card now also reports a single confirmed number (156 matched + 2 misattributed = 158) rather than a bare fraction, with the personal-log total and true-gap count moved to the sub-text — consistent with how Lara's and Thomas's cards already worked.

**Tim Gamin and Liam Simpson reconciled (2026-07-21) — same ghost-hosted check as Lara/Thomas.** Kyle noticed the new "(Actual)" card wasn't showing for Tim or Liam and asked why the global label fix "wasn't showing up" for them — it wasn't a bug, they simply didn't have a `RECONCILED_BOOKINGS` entry yet (the label only appears where one exists). Ran the same `hs_attendee_owner_ids CONTAINS <their ID> AND creator != them` check used on Lara and Thomas:

- **Tim Gamin: true Meetings Hosted = 106** (90 raw, owner=Tim + 16 real ghost-hosted, mostly booked by Sam). Raw search returned 18 candidates; screened out 2 that aren't real client meetings — a duplicate record (two "BAP - Sky - Andre Lahoiud" Meetings created 22 seconds apart, clearly a double-submit, counted once) and the team-wide "END OF QUARTER EVENT" RSVP.
- **Liam Simpson: true Meetings Hosted = 515** (476 raw, owner=Liam + 39 real ghost-hosted, mostly booked by Amy). Raw search returned 43 candidates; screened out 4 — 3 team-wide events (End of Quarter, Melbourne Cup, Christmas Party) and 1 logistics-only meeting ("Provide Access to B103 Greens," not a client appointment). Cancelled-but-recorded meetings were kept, same as everywhere else on this dashboard.

Neither has a personal spreadsheet or email trail checked yet for true zero-trace gaps the way Sam's and Lara's did — this reconciliation only covers the ghost-hosted side, same caveat as Thomas's entry.

**Amy fully re-verified (2026-07-21) — Kyle didn't believe the earlier 5-name spot check.** Redid it properly: pulled all 21 of her raw BAP-pattern emails (not a sample), deduped to 15 distinct real appointments, checked every one against her 75 self-created Meetings via a normalized-token matching script. Result: 15/15 matched, zero unmatched. The "no gap, 75 = 75" conclusion holds up under full scrutiny — this wasn't a case of an under-checked claim turning out wrong, it was already right, just under-verified.

**Tim Gamin's Meetings Booked (Actual) = 92, not 89 — first rep to need both a Booked AND a Hosted reconciliation.** Kyle: "I also need Meetings actually booked Reconciled... He's a booker." Ran the same email-trail method as Sam: deduped 125 raw BAP-pattern emails to 60 distinct appointments, matched 45 directly against his 89 self-created Meetings. Of the 15 leftover:

- 5 (Monya Robb, Lily Ho, Scott Li, Cristian Sylvestre, Andre Lahoiud) are meetings **Sam** actually booked with Tim as the real attendee — already counted in Tim's Hosted/ghost-hosted total, not his own booking. Correctly excluded from Booked.
- 3 (Salah Kahlifa, Dan Park, "Diarnah Lynch & Chloe") are other reps' clients (Mark Meyer's, Diarnah's) that Tim was just copied on — not his bookings, excluded.
- 3 (Gigi Gouw, Shahin Oftadeh, Steven Cirson) have a real Meeting record in HubSpot, just filed under a different creator (Kyle Yin, Sam respectively) — misattributed, not missing. Added to his 89 raw.
- 4 (Pooja Shevade, Li Lien Chua, Hazmal Zamri, Aida Khoshvaght) have zero Meeting record anywhere under any creator, checked directly — true gaps.

**Total Actual Meetings Booked for Tim: 89 + 3 misattributed = 92**, with 4 true gaps flagged separately (not included in the 92, same treatment as everyone else's true gaps on this dashboard).

**Data structure change to support this:** `RECONCILED_BOOKINGS` used to hold one reconciliation per rep (a single method/metric). Tim needed both a Booked (92) and a Hosted (106) entry at the same time — restructured to `{ booked: {...} | null, hosted: {...} | null }` per rep, with the card renderer checking each side independently. Every other rep's entry was migrated to the new shape with no change in the numbers shown.

**Amy's real appointment log (2026-07-21) — upgraded from an email-trail estimate to a real spreadsheet reconciliation, same method as Sam's, and it overturned the "no gap" conclusion.** Kyle uploaded her actual personal log, "Amy Andrade Appointment Outcomes.xlsx" (a setter's log — she books for Liam, Tom Murray, and occasionally Lara as the closer). 75 real rows in it, matching her raw HubSpot Meeting count of 75 — which looked like confirmation of "no gap" at first glance, but checking name-by-name in both directions told a different story:

- 74 names are common to both her spreadsheet and her 75 raw Meetings.
- 1 spreadsheet row, **Jennie Hain** (booked for Tom Murray), has zero Meeting record anywhere in HubSpot — but is fully confirmed real via 5 emails (booked May 26, cancelled June 3). Same disappearing-Meeting-record bug as Sam's, just recovered here instead of logged as an unknown. This is exactly what the earlier "75, no gap" check missed, and the reason is instructive: that check only searched for emails with "BAP" in the subject line, and Jennie Hain's booking was titled "Live Caringbah" — no "BAP" anywhere. A full 21-email re-check (done a few messages earlier, also with 0 unmatched) still couldn't catch this because it drew from the same narrow BAP-only pool. Only cross-referencing her real spreadsheet — which has no such filter — surfaced it.
- 1 raw Meeting, **Rachael Surr & Thomas Murray**, has no row in her spreadsheet at all — a real booking she just never personally logged. The mirror-image gap.

**Corrected total: 76** (74 common + 1 log-only, recovered via email + 1 HubSpot-only, unlogged). Dashboard now shows Amy's Booked reconciliation using the same `spreadsheet` method as Sam's (reconciledCount + recoveredViaEmail + extraUnlogged), not the older simpler `total` method.

**Lesson repeated here, worth stating plainly since it's now happened three times this session (Sam's stopword bug, Lara's creator-only search, Amy's BAP-only filter): any reconciliation check is only as complete as its search filter.** A narrow filter (subject-line keyword, single creator ID, sample of 5 names) can produce a clean-looking "no gap" result that isn't actually clean — it just means the filter didn't happen to catch the gap. The fix each time was the same: widen the search to something that doesn't depend on a naming convention or a single ID (here, the rep's own independently-kept spreadsheet).

## Quick queries for next time

**Calls, by rep/range:**
```sql
SELECT COUNT(*) FROM CALL WHERE hubspot_owner_id = '<ID>' AND hs_timestamp BETWEEN '<start>' AND '<end>'
```

**Meetings booked, by rep/range (booking date):**
```sql
SELECT COUNT(*) FROM MEETING WHERE hs_created_by_user_id = '<ID>' AND hs_createdate BETWEEN '<start>' AND '<end>'
```

**Booked vs. Hosted, lifetime:**
```sql
SELECT COUNT(*) FROM MEETING WHERE hs_created_by_user_id = '<ID>'   -- booked
SELECT COUNT(*) FROM MEETING WHERE hubspot_owner_id = '<ID>'        -- hosted
```

**No-shows, lifetime or this period:**
```sql
SELECT COUNT(*) FROM MEETING WHERE hs_created_by_user_id = '<ID>' AND hs_meeting_outcome = 'NO_SHOW'
-- add "AND hs_createdate BETWEEN '<start>' AND '<end>'" for this-period only
```

## Owner IDs

Amy Andrade `33175505` · Samuele Palamara `32133440` · Kyle Yin `33760446` · Tim Gamin `33001011` · Diarnah Lynch `790812864` · Mark Meyer `1337198788` · Thomas Murray `1872661741` · Lara Mathews `32677368` · Liam Simpson `29790003`

**Real login/creator IDs that differ from owner ID (use these for any `hs_created_by_user_id` query, not the owner ID above):** Thomas Murray `64248901` · Diarnah Lynch `50258835`. Confirmed by pulling every meeting created under each ID and checking `hubspot_owner_id` matches — 741/741 and 299/300 respectively.
