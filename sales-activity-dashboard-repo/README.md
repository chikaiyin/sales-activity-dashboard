# Sales Activity Dashboard

A self-contained Cowork artifact (single `index.html`, no build step) that gives
Landmark's management a live sales-activity view for Amy, Sam, Kyle, Tim, and
the rest of the team, pulled fresh from HubSpot every time it's opened.

This repo is a **mirror** of the live artifact for history/version-control
purposes. The canonical, running copy lives inside the Cowork artifact system
at:

```
/Users/chikaiyin/Documents/Claude/Artifacts/sales-activity-dashboard/index.html
```

That location is managed by Cowork and can only be edited through the
`mcp__cowork__update_artifact` tool — not by writing to the path directly (a
fresh Claude session will get a read-only error if it tries). See "How to
make changes" below.

## What it does

- **Team Comparison** — calls, connect rate, emails, meetings booked, meeting
  completion rate, new leads, enquiries, notes, speed-to-first-contact — one
  row per rep, filterable by rep and by range (Today / Yesterday / This Week
  / Last Week / Last 14 Days / This Month / Last Month).
- **Sales Agent (Agent Focus)** — drills into one rep: period KPIs, a
  meetings-booked activity log, lifetime Meetings Booked-vs-Hosted, lifetime
  **Book Health** (contacts this specific rep has touched vs. never touched),
  and trend mini-charts (daily over 14 days, weekly over 12 weeks, for Calls
  Made and Meetings Booked).
- **Today's Calls** — always "today," independent of the range tabs.
- **Activity Log** — full scrollable raw log across whatever reps/range are
  selected.

## Dependencies

| Dependency | Purpose | Notes |
|---|---|---|
| A HubSpot CRM connector exposing a `query_crm_data`-style MCP tool | All data | In this account the tool is `mcp__fa06d9cc-b3b9-4e63-a33e-89af872b44ba__query_crm_data`. **This exact ID is specific to this HubSpot connection** — on a different machine/account it will be a different ID. Update the `QUERY_TOOL` constant near the top of the `<script>` block, and the `mcpTools` array in the `cowork-artifact-meta` JSON block at the top of the file, to match. |
| Chart.js v4.5.0 (UMD) | All charts | Loaded from `cdnjs`/`jsdelivr` via `<script src>` — no local copy needed, already inlined as a tag in `index.html`. |
| Cowork artifact runtime (`window.cowork.callMcpTool`) | Every HubSpot call | This is why the file only runs correctly *inside* a Cowork artifact view, not as a plain static HTML file opened in a browser. |
| `localStorage` | Caches closed date ranges (Yesterday/Last Week/Last Month) | No server, all client-side. |

No npm install, no bundler — it's one HTML file with an inline `<script>`.

## How to make changes (for a future Claude session)

1. Read the current live copy: call `mcp__cowork__list_artifacts` to confirm
   the artifact id (`sales-activity-dashboard`) and path, then `Read` that
   path to see the current HTML.
2. Write the full, modified HTML to a scratch file in your own outputs
   directory (the live artifact path itself is read-only to a session —
   editing it directly will fail).
3. Sanity-check the JS syntax before pushing — extract the inline
   `<script>...</script>` block and run it through `node -c` (or equivalent)
   to catch a stray brace/quote before it breaks the dashboard for real
   users.
4. Call `mcp__cowork__update_artifact` with `id: "sales-activity-dashboard"`
   and `html_path` pointing at your scratch file.
5. Optionally, copy the same final HTML into this repo as `index.html` and
   commit, so there's history outside the artifact system itself.

## Reconciliation history

`dashboard-verification-tests.md` is the real, running log of every "does
this number look right" check that's been done on this dashboard — gotchas
found, per-rep reconciliation results, and the step-by-step recipe for "run
the HubSpot reconciliation for [rep]" (which the dashboard's own caveat text
points users to). `sam-crossref.md` and `samuele-meetings-booked.md` are the
detailed per-rep working files that log references. Read
`dashboard-verification-tests.md` first when a rep's numbers get questioned —
there's a good chance the same issue (or something close to it) has already
been chased down and documented there, and re-deriving it from scratch risks
missing a correction that's already on record (several entries in that file
are corrections of earlier, wrong conclusions — read to the end of a rep's
section, not just the first entry).

Note on this repo's location: it lives on the user's Desktop, which the
Cowork sandbox mounts with delete/rename protection by default (git's normal
commit process needs to rename lockfiles, so a plain `git commit` in this
folder can fail with "Operation not permitted" until file deletion is
explicitly enabled for the session via `allow_cowork_file_delete`). If commits
here start failing with lock errors, that's why — re-request delete
permission for this folder, or do the git work in a scratch directory and
copy the finished `.git` folder + files over as a whole.

## Configuration that's specific to this HubSpot account

All near the top of the `<script>` block:

- **`OWNERS`** — the 9 reps tracked (id + display name). HubSpot owner IDs,
  not names — add/remove reps here.
- **`REP_CREATOR_ALIASES`** — HubSpot has two ID systems (an "owner" ID and a
  separate "user/login" ID used on `hs_created_by_user_id`) that don't always
  match for the same person. Thomas Murray and Diarnah Lynch both needed
  their real login ID aliased in here, or their own self-booked
  meetings/touches silently show as zero. If a rep's numbers look flatly
  wrong, this is the first thing to check.
- **`SETTER_ONLY_IDS`** — reps who only book meetings for others to host and
  never host themselves (Amy Andrade, Samuele Palamara currently). Hides the
  "Hosted" card for them rather than showing a meaningless number.
- **`RECONCILED_BOOKINGS`** — manually verified "Actual" meeting counts per
  rep, layered on top of the live HubSpot numbers where HubSpot's own Meeting
  records are known to be incomplete (see `dashboard-verification-tests.md`).
  Not live — has to be re-run and hand-updated.
- **`CALL_DISPOSITIONS`** — HubSpot's internal disposition GUIDs mapped to
  human labels. If HubSpot's disposition list changes, this needs updating.

## Known data-model gotchas (already handled, documented here so nobody re-breaks them)

- **Meetings are dated by `hs_createdate`** (when the booking action
  happened), not `hs_timestamp` (the meeting's scheduled time) — these can be
  days or weeks apart. "Meetings Booked" always means the former.
- **Meetings are attributed by `hs_created_by_user_id`** (who actually
  booked it), not `hubspot_owner_id` (which HubSpot often leaves stuck on
  whoever created the record even when someone else is the real
  closer/attendee) — see `REP_CREATOR_ALIASES` above.
- **Ranges are plain calendar days, weekends included** — an earlier version
  silently dropped Saturday/Sunday from every multi-day range, which
  undercounted reps who do weekend open-house bookings.
- **Book Health "Untouched" is scoped per rep**, not account-wide — it uses
  HubSpot's cross-object `WHERE` filters (e.g. `CALL.hubspot_owner_id = 'X'`)
  to check whether *that specific rep* logged a Call/Email/Note against the
  contact, unioned client-side across two OR'd query pairs (the query engine
  caps cross-object filters at 2 associated object types per query). An
  earlier version used the account-wide `notes_last_contacted` rollup, which
  flips to "contacted" the moment *anyone* touches the contact — wrong for a
  per-rep number.
- **HubSpot caps `LIMIT` at 500 rows** — everything pages through
  `runQueryAllPages` up to a 10,000-row sanity ceiling.
- **The MCP-level rate limit** is real and can trip even with a low
  concurrency cap — see `MAX_CONCURRENT_QUERIES` / `MIN_CALL_SPACING_MS` near
  the top of the script if load times get slow or start erroring.

<!-- verified second commit works -->
