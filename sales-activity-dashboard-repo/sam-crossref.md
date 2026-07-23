# Sam's Spreadsheet vs. HubSpot — Cross-Reference

**Sam's manual spreadsheet:** 161 rows.
**HubSpot (as creator, `hs_created_by_user_id`):** 128 meetings.
**Gap: 33.**

Matched by name + appointment date (allowing for the day/month mix-up below): **108 confident matches**. That leaves **53 spreadsheet rows** I couldn't tie to a specific HubSpot record. A chunk of those are typos on Sam's side (e.g. "Pashant Nair" vs. HubSpot's "Prashant Nair," "Steven Grayham" vs. "Steven Graham," "Mohammad Shaheen" vs. HubSpot's "Md Shaheen") rather than real gaps — but not all of them.

## Found: a real date bug in Sam's spreadsheet

A bunch of rows show dates like **2026-11-04**, **2026-12-05**, **2026-08-05**, **2026-09-05** — reading as November/December/August/September. Side-by-side with HubSpot, several of these are clearly **day and month swapped** (Australian DD/MM typed into a sheet that read it back as MM/DD): e.g. Sam's "Varun Handa, booked 11/04, appt 11/04" lines up with HubSpot's Varun Handa meeting booked **April 10**, held **April 11** — not November at all. Worth telling Sam to double check his Excel's date format/locale, since it's silently scrambling any date where both day and month are ≤ 12.

## Rows I couldn't match to a HubSpot meeting (53)

Some of these are almost certainly the swapped-date issue above; others may be walk-ins/referrals that never got their own HubSpot record (e.g. Jyoti Lakhani and Rahul Mishra both show up same day as Varun Handa's appointment, and Sam's notes on Varun say "referred 2 friends, they bought as well" — likely they came in with Varun's appointment rather than getting a separate booking).

| Booked | Appt Date | Name | Agent | Outcome |
|---|---|---|---|---|
| 2026-11-04 | 2026-11-04 | Jyoti Lakhani | Mark | Complete |
| 2026-11-04 | 2026-11-04 | Rahul Mishra | Mark | Complete |
| 2026-04-29 | 2026-01-05 | Rakesh Vasudeva | | Incomplete |
| 2026-04-29 | 2026-01-05 | Mala Sharma | | Incomplete |
| 2026-04-21 | 2026-02-05 | Jono N | | Incomplete |
| 2026-04-16 | 2026-02-05 | Rachel Ward | Mark | Incomplete |
| 2026-02-13 | 2026-02-21 | Esther Hanna | Mark | Incomplete |
| 2026-02-26 | 2026-02-28 | Ivan Ang | | |
| 2026-02-20 | 2026-03-13 | Esther Hanna | | |
| 2026-02-03 | 2026-03-13 | Kereen Rodrigo | | |
| 2026-03-19 | 2026-03-25 | Sinchai Apilukpuvadol | | |
| 2026-03-30 | 2026-04-17 | Sue Anato | | |
| 2026-03-20 | 2026-04-18 | Ruby Chan | | |
| 2026-03-20 | 2026-04-21 | Glenn Macabenta | | |
| 2026-08-05 | 2026-05-13 | Supriya Amgai | | |
| 2026-05-13 | 2026-05-15 | Alex Nanda | | |
| 2026-05-18 | 2026-06-18 | Wisitta Gray | | |
| 2026-04-05 | 2026-09-05 | Michael Garcia | | |
| 2026-09-05 | 2026-06-27 | Priyam Sarkar | | |
| 2026-05-25 | 2026-05-30 | Glenda Penecitos | | |
| 2026-12-05 | 2026-05-22 | Brandon Clark | | |
| 2026-06-25 | 2026-07-04 | Che Tan | | |
| 2026-03-13 | 2026-07-11 | Ian Roberts | | |
| 2026-06-18 | 2026-06-19 | Li Lien Chua | | |
| 2026-07-09 | 2026-06-11 | Siddharth Sridhar | | |
| 2026-05-20 | 2026-05-25 | Kunal Shanthappa | | |
| 2026-05-27 | 2026-05-31 | Aida Khoshvaght | | |
| 2026-02-23 | 2026-03-03 | Roginee Govender | | No Show |
| 2026-02-16 | 2026-02-22 | Chris Chen | | Show |
| 2026-02-28 | 2026-03-15 | Jovanka Mancevski | | |
| 2026-04-20 | 2026-04-25 | Maylaine Rucat | | |
| 2026-04-20 | 2026-04-25 | Mike Kano-Mccallum | | |
| 2026-04-23 | 2026-01-05 | Mohammad Mahmudul Haque | | |
| 2026-03-20 | 2026-03-14 | Pashant Nair | | (likely = HubSpot's "Prashant Nair," just a typo) |
| 2026-09-03 | 2026-04-27 | Praveen Chananna | | |
| 2026-02-18 | 2026-05-03 | Marcia Corbett | | |
| 2026-04-30 | 2026-05-15 | Murielle Moya | | |
| 2026-04-15 | 2026-05-15 | Sargon Aorahim | | |
| 2026-09-05 | 2026-05-16 | Rabee Shreet | | |
| 2026-05-21 | 2026-05-25 | William Li | | |
| 2026-10-04 | 2026-10-04 | Maria Ellis | | |
| 2026-02-24 | 2026-06-20 | Reza Babaei | | |
| 2026-06-25 | 2026-07-04 | Rob Coburn | | |
| 2026-02-18 | 2026-04-04 | Rohith Gujja | | |
| 2026-04-14 | 2026-04-21 | Nish Bhansali | | |
| 2026-04-16 | 2026-05-20 | Agnes Yee | | |
| 2026-03-31 | 2026-10-04 | Nilou Jamshidi | | |
| 2026-06-26 | 2026-06-27 | Izzy Gilmore | | |
| 2026-07-01 | 2026-07-03 | Sanaz Hashemi | | |
| 2026-06-12 | 2026-07-04 | Hazmal Zamri | | |
| 2026-06-22 | 2026-07-04 | Tony Le | | |
| 2026-07-08 | 2026-07-25 | Henry Caldwell | | |
| 2026-07-17 | 2026-07-21 | Susan Chan | | |

Note: a handful of these (Nish Bhansali, Agnes Yee, William Li, Maria Ellis, Rob Coburn, Rohith Gujja...) DO have same-named HubSpot meetings, just far enough off in date (more than 2 days after accounting for a possible day/month swap) that the automated match didn't count them — worth a manual glance rather than treating this whole list as "missing from HubSpot."

**Bottom line:** this is a rough automated pass, not a guaranteed-accurate audit — matched on name text + date proximity, no human eyes on each row. Good enough to hand to Sam as a starting checklist of "these might not be logged, or might just be a typo/reschedule," not good enough to conclude 53 meetings are truly missing from HubSpot.

## Follow-up: checked each name directly in HubSpot (not just Sam's 128)

Kyle asked to cross-reference the unmatched names against HubSpot itself — not just Sam's created-by-him meetings — to see whether this is really a "never logged" problem. Checked 35 of the 51 names as CONTACT records, then looked for ANY associated MEETING (any creator, any owner):

**Confirmed real gaps — contact exists, zero meetings logged anywhere:** Rachel Ward, Esther Hanna, Ivan Ang, Kereen Rodrigo, Sue Anato, Ruby Chan, Glenn Macabenta, Supriya Amgai, Alex Nanda, Wisitta Gray, Michael Garcia, Mala Sharma, Priyam Sarkar, Glenda Penecitos, Brandon Clark, Che Tan, Kunal Shanthappa, Aida Khoshvaght, Jovanka Mancevski, Maylaine Rucat, Mike Kano-McCallum, Izzy Gilmore, Sanaz Hashemi, Hazmal Zamri, Tony Le, Henry Caldwell, Praveen Chananna — **27 contacts, real appointments per Sam's log, nothing logged as a HubSpot Meeting at all.**

**Jono N specifically (the one Kyle flagged):** No MEETING record exists for this contact — confirmed by checking directly. But he does have 9 logged Calls and 6 Tasks with Sam, including "confirm app and see if he can adjust time to 12:00pm" and "Reschedule app." So the appointment was clearly real and being actively managed — it just never got created as a HubSpot Meeting engagement, only as calls/tasks. This is the clearest single proof that the gap is a **logging problem** (appointments happening and being worked, but not entered as Meetings), not just Sam's spreadsheet being messy.

**Contact + meeting both exist, but logged by someone else entirely:** Ian Roberts, Li Lien Chua, Chris Chen, Marcia Corbett, Rabee Shreet, Reza Babaei, Jyoti Lakhani, Rahul Mishra. These meetings exist but were created by user IDs `60605153` and `50258835` — neither of which is Sam or any of the 9 reps on the dashboard, most likely a reception/back-office account entering these after the fact. So the appointment IS in HubSpot, just not attributed to Sam as the booker — a related but different issue from "never logged."

**False alarms — actually fine, just missed by the automated date-matching:** Sargon Aorahim and Susan Chan both DO have a meeting created by Sam that matches; my earlier ±2-day matching tolerance was too tight to catch them (Susan Chan's appointment shifted from July 21 to July 27, for example).

**Still to check:** Rakesh Vasudeva (only meeting found is from 2024, unrelated to the May 2026 spreadsheet entry — likely also a real gap), and 15 remaining unresolved names not yet checked directly: Roginee Govender, Chris Chen (dup), Ian Roberts, Mohammad Mahmudul Haque, Pashant Nair, William Li, Maria Ellis, Rob Coburn, Rohith Gujja, Nish Bhansali, Agnes Yee, Nilou Jamshidi, Siddharth Sridhar.

**So: yes — this looks like it's mostly a HubSpot logging gap, not a Sam-attribution problem.** Roughly 27 of the ~35 names checked have zero Meeting record anywhere in the system despite being real, worked appointments. A smaller group are logged, just under an unrelated account instead of Sam. Worth raising with whoever owns the HubSpot process on this team — appointments are happening and being worked (calls, tasks, notes) but not consistently making it into a Meeting record.
