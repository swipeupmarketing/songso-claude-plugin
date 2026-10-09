# Meta × song.so CVR comparison (daily/all-time snapshot)

A lighter-weight variant of the kill/keep evaluation for when the user just
wants a **conversion-rate snapshot** — "how are campaigns converting today",
"ViewContent vs stream clicks", "daily CVR check" — rather than a full
per-ad kill/keep pass. Use this alongside, not instead of, "The kill/keep
rule" and song.so's built-in hourly optimizer ("Automated optimization" in the
main playbook); this is the quick cross-check, the kill/keep rule is the
actual pause/keep decision.

**Goal:** pull Meta ViewContent events and song.so platform clicks for the
same date range, match them by campaign/artist, and present a comparison
table with CVR per campaign.

**CVR formula:** `song.so platform clicks ÷ Meta ViewContent events`
(song.so uses server-side UTM tracking and is more accurate than the Meta
pixel; CVR > 100% is expected and normal — it means the Meta pixel is
undercounting, typically from iOS privacy prompts or ad blockers, not that
song.so is wrong.)

## Step 1 — Identify the ad account

Ask the user for the Meta ad account ID if not already known (format
`act_XXXXXXXXX`). May already be in context for a recurring session.

## Step 2 — Pull campaigns + ViewContent from Meta

Tool name varies by connector — this may be `ads_get_ad_entities` on some
Meta MCP servers, or `mcp_meta_ads_get_insights`/`get_campaigns` on others
(see "Available tools" in `references/meta-ads-mcp.md`); use whichever the
connected MCP actually exposes. Whichever it is, request:

- `ad_account_id`: the user's account
- `date_preset`: `"today"` for a daily snapshot, or `"maximum"` for all-time
- `fields`: `name`, `spend` (or `amount_spent`), `effective_status`, `id`,
  and the CPA breakdown for ViewContent — see the field-naming caveat below
- No entity-level override needed for campaign-level; the MCP typically
  defaults to campaign level

Only campaigns that actually spent in the date range will have CPA data;
paused/zero-spend campaigns return "Not available" for it — that's expected,
not an error.

**Filtering by name**, if supported: something like
`[{"field": "campaign.name", "operator": "CONTAIN", "value": ["Artist Name 1", "Artist Name 2"]}]`
— `value` must be an array even for a single term on tools that use this
filter shape.

**Deriving the ViewContent count:** some connectors return a raw CPA
sub-field (e.g. `cost_per_action_type:offsite_content_view_add_20_s_calls`)
as a decimal rather than the ViewContent count directly — in that case:
```
ViewContent = amount_spent ÷ that_raw_decimal_CPA
```
A `cost_per_action_type:view_content` field, if present, is often a
formatted currency string, not a count — check the actual value/type before
using it, and prefer a raw numeric field for math. Requesting the bare
`cost_per_action_type` field (no sub-field) commonly returns "Not
available" — request the specific named sub-field instead.

## Step 3 — Find the song.so smartlinks and today's clicks

1. `list_artists` (song.so MCP, no filter) → match artist names against the
   campaign names from Step 2 (campaign name pattern is usually
   `"Artist Name - Song Title"`).
2. `find_smartlink_for_track` per artist/song (preferred — scoped lookup)
   or `list_smartlinks` if you need to browse; pick the smartlink whose
   `title` matches the campaign's song, and the most recently created one
   if an artist has several matching candidates.
3. `get_smartlink_analytics {smartlink_id, start_date, end_date}` (song.so
   MCP) — **for "today", set both `start_date` and `end_date` to today's
   date.** Do not use `get_platform_clicks` for a same-day snapshot: it has
   a multi-hour indexing lag and will read back as 0 for very recent
   traffic. For an all-time comparison, omit `start_date`/`end_date`
   entirely and read `total_views`/the response's totals rather than the
   daily series.
4. From the response's `daily_click_traffic` for the target date:
   `platform_clicks = paid + organic`.

Call all the smartlink lookups in parallel where possible rather than one
at a time.

## Step 4 — Build the comparison table

| Artist | Song | Spend | Meta ViewContent | song.so Clicks | CVR |
|---|---|---|---|---|---|
| Artist | Song Title | €X.XX | N | N | XX% |

Sort by CVR descending. Interpretation notes to include in the report:

- **CVR > 100%:** expected — the Meta pixel undercounts (iOS/ad-blockers
  block it; song.so tracks server-side).
- **CVR < 80%:** worth flagging — people land on the smartlink but don't
  click through to a streaming platform.
- **CVR > ~130%:** the Meta pixel is very likely undercounting significantly
  for that specific campaign.
- If run mid-day, say explicitly that the numbers are a partial-day
  snapshot.

## Common issues

| Problem | Fix |
|---|---|
| A specific CPA sub-field errors as unsupported | Try requesting it as part of the broader `cost_per_action_type` field response instead of standalone, or vice versa — connector-dependent, check the error message. |
| Full-account `date_preset: "maximum"` errors (e.g. 500) | Query per-campaign with an explicit campaign-id filter instead of the whole account at once. |
| `get_platform_clicks` returns 0 for today | Use `get_smartlink_analytics` with today's date instead — see Step 3. |
| A campaign has no ViewContent data | It had no spend in the period — skip it, not an error. |
| An artist has multiple smartlinks | Match by song title against the campaign name; fall back to the most recently created one. |
| A "CONTAIN"-style name filter errors about expecting a list | Wrap the value in an array, e.g. `"value": ["term"]`, not a bare string. |

## All-time variant

Same flow, but: Meta uses `date_preset: "maximum"` (fall back to per-campaign
queries if a full-account query errors); song.so calls
`get_smartlink_analytics` with no date range and reads the response's total
clicks rather than the daily series. All-time ViewContent derived from
total spend ÷ average CPA is less precise than a daily snapshot — prefer
running this daily when possible.

## Hint: CVR cross-validation via song.so UTM clicks

> **Prerequisite:** this only works if the campaign was created via the Meta
> Ads MCP through Claude. In that case Claude knows the campaign ID, all ad
> set IDs, and all ad IDs — and the UTM parameters were embedded at
> creation time — making per-*ad* attribution possible (the CVR snapshot
> above is per-campaign; this is a finer-grained, per-ad secondary check).

Since all IDs are known from the creation session, cross-validate
Meta-reported conversions against song.so's own click data:

- **Meta side:** pull `view_content` events per ad from the insights call
  (`actions` field, `action_type: view_content` — or the connector's
  equivalent, e.g. `ads_get_ad_entities` → `actions`).
- **song.so side:** call `get_platform_clicks_by_utm` on the campaign's
  smartlink, filtering by the UTM parameters that encode the
  campaign/ad-set/ad ID.

Compare the two numbers per ad to detect:
- **Pixel misfires** — Meta reports more ViewContent events than song.so
  recorded clicks.
- **Attribution gaps** — song.so saw clicks Meta didn't attribute.
- **Bot/invalid traffic** — large Meta click counts with near-zero song.so
  platform activity.

Use this as a secondary signal when evaluating CVR: if Meta's ViewContent
count is inflated relative to song.so clicks, the real CVR may be lower
than Meta reports.
