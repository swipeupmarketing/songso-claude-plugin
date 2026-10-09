# song.so MCP — smartlinks & conversion data

Connector: `songso` (endpoint `https://mcp.song.so/mcp`, OAuth via song.so
session). Not connected yet? Guide the user: Claude Settings → Connectors →
Add custom connector → that URL → sign in on the song.so consent page. Any
song.so account works — no paid plan is required for the MCP tools
themselves. Data is scoped to the logged-in song.so user server-side (every
read/write tool filters on the authenticated `user_id`) — never pass or ask
for a user_id.

**song.so is MCP-tools-only — never browser automation.** Don't use
Claude-in-Chrome, screenshots, or manual clicking against app.song.so/
song.so for anything, even as a fallback. That pattern is reserved for Meta
Ads Manager (`references/ads-manager-browser.md`) when no Meta MCP is
available; it does not apply here. If a capability isn't covered by one of
the tools below, tell the user and have them do it in the song.so dashboard
themselves.

Tools: `list_smartlinks`, `get_smartlink`, `get_smartlink_analytics`,
`list_artists`, `list_pixels`, `find_smartlink_for_track`,
`create_smartlink`, `edit_smartlink`, `add_pixel`, `edit_pixel`,
`get_platform_clicks`, `get_platform_clicks_by_utm`, `get_track_metadata`,
`get_track_popularity`, `search_meta_targeting`,
`list_tour_links`, `get_tour_link`, `add_tour_dates`, `update_tour_date`,
`delete_tour_date`, `edit_tour_link`, `list_tour_video_artists`,
`list_tour_base_clips`, `generate_tour_creatives`, `poll_tour_creatives`,
`get_tour_creative_job`, `list_tour_video_renders`, `add_tour_creative_to_adset`.

`search_meta_targeting` is on this same connector but isn't song.so data —
it proxies Meta's ad targeting taxonomy (interests, geo, locales, etc.); see
"Resolving interest/geo/locale IDs" in `references/meta-ads-mcp.md` for how
to use it.

`get_track_metadata {url, country?}` resolves a Spotify/Apple Music/YouTube/
Deezer/Tidal/Amazon Music/Soundcloud URL into cross-platform links, title,
artist, and cover art — not scoped to any account, useful for resolving a
release's canonical links before creating a smartlink.

`get_track_popularity {smartlink_id, start_date?, end_date?, limit?}` returns
the historical Spotify popularity score (0-100) song.so's daily job has
recorded for one of the user's own smartlinks — the latest score plus a
history series. Empty history usually means the smartlink has no Spotify
link, or the daily job hasn't run for it yet (not an error).

Both `create_smartlink` and `edit_smartlink` accept a `cover_image` URL —
pass `get_track_metadata`'s `coverImage` straight through if the user hasn't
supplied their own art.

If calls fail with `Session refresh failed: Invalid Refresh Token`, the
OAuth session is broken (happens after connector updates) — suggest
reconnecting via `suggest_connectors` with the server UUID; don't retry
blindly.

## Creating a smartlink

1. **Always `find_smartlink_for_track` first** (`name` and/or `spotify_url`)
   — it returns matches with `public_url`. If a complete link already exists
   in the user's account, use it instead of duplicating.
2. `create_smartlink {title, artist_id | artist_name, spotify_url,
   platforms?, release_date?, type?, cover_image?}`. Notes: available on any
   plan; there is **no slug parameter** — the slug comes from the title, with
   automatic fallbacks (`title-artist`, then `-2`, `-3`, …) if taken; use
   `type: "presave"` for unreleased tracks (needs `release_date`). If
   `spotify_url` is a real `https://open.spotify.com/...` URL and you don't
   explicitly pass `cover_image` or `platforms`, the tool automatically calls
   `get-track-metadata-musicfetch` to fetch the cover art and streaming links.
   If `artist_id` is provided it must already exist in the user's account.
   If `artist_name` is provided and the artist does not exist yet, the
   `create-smartlink` edge function creates the artist automatically and
   attaches it to the new smartlink.
3. Attach the tracking pixel by assigning it to the **artist(s)** — pixels
   are never attached to a smartlink. Use `edit_pixel {pixel_id,
   artist_ids: ["..."]}` or `add_pixel {..., artist_ids: ["..."]}`.
   Pixels with no `artist_ids` are **global** and fire for all the user's
   smartlinks. Never change pixel → artist assignment without explicit user
   confirmation. Pixel coverage is by **artist**, not by smartlink. Before
   running ads, call `list_pixels` and check for a pixel whose `artist_ids`
   contains the smartlink's `artist_id` **OR** a pixel with no `artist_ids`
   (global — it fires for every artist). Do **not** look at
   `tracking_pixel_id` on the smartlink; it is not the source of truth. If
   no pixel covers the artist, ask the user to `add_pixel {...,
   artist_ids: ["..."]}` or `edit_pixel {pixel_id, artist_ids: ["..."]}`
   before launching the ad.

## Editing a smartlink

`edit_smartlink {id, ...fields}` updates only the fields you pass — never
send fields you don't intend to change. Supports `title`, `artist_name`,
`artist_id`, `description`, `release_date`, `spotify_url`, `isrc`,
`is_locked`, `custom_url` (globally unique — pass `null` to clear), and
`platforms` (an array that **replaces the entire ordered list** of
streaming/download links — always pass the full desired list, not just the
ones you're changing). Pixels are **not** attached to a smartlink.

## Tour links and tour dates

Tour smartlinks (`type='tour'`) have dedicated tools for managing tour dates.
A tour link is created with `create_smartlink {type: "tour"}`, then tour dates
are added with `add_tour_dates`.

### Listing and reading tour links

- `list_tour_links {search?}` — lists tour smartlinks with date counts
- `get_tour_link {id}` — returns a tour link with all its tour dates ordered by date

### Adding tour dates

`add_tour_dates {smartlink_id, dates: [...]}` inserts one or more tour dates.
Each date needs `city`, `country`, `venue`, and `date` (ISO). Optional:
`ticket_url`, `status` (default "available"), `price`, `currency` (default "EUR").
The `tour_date_url` is generated automatically as `https://song.so/{slug}/{city}`
with the city transliterated (e.g. "München" → "muenchen").

### Updating a tour date

`update_tour_date {tour_date_id, ...fields}` updates individual fields. If
`city` changes, the `tour_date_url` is regenerated from the smartlink slug +
new city.

### Deleting a tour date

`delete_tour_date {tour_date_id}` permanently removes a tour date.

### Editing tour link fields

`edit_tour_link {id, ...fields}` updates top-level smartlink fields specific
to tour links: `title`, `tour_name`, `artist_name`, `artist_id`,
`header_image`, `image_position`, `image_aspect_ratio`, `use_artist_colors`,
`color_schema`, `is_locked`, `waitlist_mode`, `waitlist_description`,
`waitlist_require_first_name`, `waitlist_success_message`. Does NOT touch
tour dates — use the tour date tools for those.

## Tour campaigns on Meta

A tour campaign is one Meta campaign with one ad set per tour date (city), registered
in song.so as a **tour** campaign. Two tools get a campaign into that state; both are
**dry run by default** and only act with the `confirmation_code` the dry run returned
for exactly the same arguments.

### Import an existing tour campaign

`import_tour_campaign {ad_account_id, campaign_id, tour_link_id?, city_overrides?}`
registers a campaign that already exists on Meta (built in Ads Manager or through the
Meta Ads MCP). It pairs each ad set with a tour date of a tour link — by the city in
the ad set name ("Berlin - Broad", "HAMBURG | docs"), the same matching as the
dashboard — reads the Facebook Page from the ads, and stores the campaign. Nothing is
changed on Meta. Without `tour_link_id` the link is guessed from the ad destination
URLs, or the candidates are listed. **Check the pairing with the user before
confirming:** the tour optimizer pauses ad sets of sold-out and past dates, so a wrong
pairing pauses the wrong ad set. Correct it with `city_overrides {adset_id:
tour_date_id | null}`.

### Create a tour campaign

`create_tour_campaign {ad_account_id, pixel_id, page_id, tour_link_id, lifetime_budget?,
end_date?, ...}` builds what the Tour Campaign wizard builds: campaign (OUTCOME_SALES,
named "<Artist> - <Tour> (Tour)"), one ad set per upcoming, not sold-out tour date
(named "City - Venue - dd.mm.yyyy") with a lifetime budget per ad set (default 300, in
the account currency), an end date (default 30 days out), Instagram Reels + Stories on
mobile, optimization for the pixel's AddToCart event (the ticket click), city targeting
within 50 miles, and the artist's warm audiences attached (Instagram followers and
engagers 365 days, site visitors 90 days — created if missing). Everything is created
**PAUSED** unless the user explicitly asked to go live (`activate: true`).

City targeting never guesses. The dry run looks every city up on Meta: one clear match
is used; a city Meta knows several times (a "Freiburg" in three regions) comes back
with its candidates and must be settled with `city_keys {tour_date_id: key}` after
asking the user; a city Meta cannot find blocks the build unless
`allow_country_fallback` is set (then the whole country is targeted). The dry run lists
these as `blockers` — resolve them, run the dry run again, show the plan, get an
explicit go, then build.

The ad sets have no ads yet. Next: check the artist template and clips
(`list_tour_video_artists`, `preview_tour_text`, `list_tour_base_clips`), then run
`generate_tour_creatives {campaign_id}` to render the city videos, upload them and
create the ads (tour ads carry no primary text or headline).

## Tour creative generation

Tour campaigns use city-specific video ads: each artist has "blankos" (base
performance clips) that get overlaid with the city, venue, and date text for
each tour date, producing unique creatives per city. The rendering is done by
Rendi (a cloud FFmpeg service), and the finished videos are uploaded to Meta
and turned into ads in each ad set.

### Artist templates and blankos

- `list_tour_video_artists` — lists configured artist templates (artist_key, font_url, logo_url, preset)
- `list_tour_base_clips {artist_key}` — lists the blankos (base clips) for an artist, ordered by sort
- `upsert_tour_video_artist {label, font_url?, logo_url?, preset?, overwrite?}` — create or edit a template. Only the preset keys passed change. The logo and font are downloaded from the URLs given and stored in song.so; a new template needs a font. **Templates are shared by every song.so user**, so an existing template is never changed unless `overwrite: true` — the tool first returns what exists; only overwrite after the user confirmed it is theirs.
- `preview_tour_text {artist_key?, tour_date_id?, format?}` — the exact lines that will be burned into the videos for a tour date, produced by the renderer's own code (long-venue stacking, country style, case and date format included). Pass `format` to try a change before saving it. Run it after creating or changing a template and show the user the result.
- `add_tour_base_clip {artist_key, video_url, name?, duration_seconds?}` — add a base clip from a public URL: 30 MB or less is stored untouched; larger is compressed once to just under 30 MB. Compression is asynchronous: the tool returns `status: "compressing"` and `check_with` values — call `check_tour_base_clip_import` with them every 15-20 seconds until it reports `done` (it retries at a lower bitrate by itself if the result is still above 30 MB, and adds nothing on failure). Pass `duration_seconds` when known, so the first attempt lands near 30 MB instead of guessing.

**What a template (preset) contains.** Logo (PNG) and font (.ttf/.otf, required)
plus a `preset` JSON: `logoWidth` (px on the 1080×1920 canvas), `logoY`,
`dateY`, `venueY` (vertical positions as fractions 0–1; text is always
centred horizontally), `fontSize`, `minFont`, `maxChars` (long lines shrink to
fit), `fontColor`, `fontWeight`, contour (`contourEnabled`, `contourThickness`,
`contourColor`) and shadow (`shadowEnabled`, `shadowColor`, `shadowIntensity`).
Every render is laid out on a 1080×1920 canvas (the clip is scaled and centre-
cropped like the dashboard preview), so a preset looks the same whatever the
clip's resolution. The dashboard preview is rendered by the same code as the
server, so what the user sees in the preview is what gets burned in.

**What text goes on the video — `preset.format`.** An optional object; without
it the template renders the classic text `DATE`, then `VENUE | CITY`, in capitals.
Keys:

- `fields` — ordered list of `{id: "city" | "venue" | "date", on: boolean}`: the
  order on the video and which fields are shown.
- `country` (boolean) — append the country to the city (`City, Country`); the
  country always follows the city and is not a field of its own.
- `countryStyle` — `"name"` (`Germany`) or `"code"` (`DE`). Stored values like
  "Deutschland", "USA", "UK" or "DE" are normalized; values that aren't a
  country are left out instead of printed.
- `caseMode` — `"upper"` (AA), `"title"` (Aa), `"lower"` (aa) or `"asis"` (as
  written on the tour smartlink).
- `dateFormat` — `dd.mm.yyyy`, `dd.mm.yy`, `dd.mm.`, `mm/dd/yyyy`, `mm/dd/yy`,
  `mm/dd`, `month d, yyyy`, `month d`, `d. month`, `d. month yyyy`; with
  `shortMonth: true` month names are abbreviated (Dec).
- `stackLongVenue` + `longVenueChars` (default 22) — when the venue is longer
  than that, the text becomes three lines: `City, Country` / `Date` / `Venue`.
  Consecutive place fields otherwise share a line joined with ` | `; the date
  is always its own line. New templates start with this rule switched on.

**Do not confuse with `tour_video_renders.date_text`.** That column always stays
the long `Month D, YYYY` form — the poll worker compares it against the tour
date to verify a render belongs to its date, and a different string would make
valid renders get skipped. The display format only affects the pixels.

Templates can be created and edited with `upsert_tour_video_artist`, or in the
dashboard: Tour Campaigns → Create tour campaign → Creatives → **+ Create Artist
Template** (or **Artist settings** in the Tour Creatives modal) — same data.

**Base clips (blankos).** Added with `add_tour_base_clip` or in the dashboard
(file picker, drag and drop, Dropbox or YouTube import). A clip of **30 MB or less is stored untouched**. A
larger clip is compressed **once, at import**, to just under 30 MB (bitrate
calculated from its length; resolution, aspect ratio and frame rate unchanged)
and the compressed file replaces the original for all future renders — the big
original is not kept, so users should keep their own copy. Existing clips are
never re-compressed.

### Ad copy: no primary text, no headline

Tour ads are created **without a primary text and without a headline** (or link
description). The creation step drops them even when it copies an existing ad as
its template, so old "Get tickets for …" text does not spread. The ad carries
the video, its poster image, the call to action `ORDER_NOW` and the city's tour
date URL. Never add primary text or a headline to a tour ad, and don't offer to.
Ads that were created earlier keep whatever text they already have — Meta
creatives can't be edited in place.

### Bulk generation flow

1. `generate_tour_creatives {campaign_id}` — starts the job: for each ad set with a matched tour date (regardless of active/paused status), finds unused blankos and submits Rendi render jobs. Returns a `job_id`.
2. `poll_tour_creatives {job_id}` — call every 10-30 seconds. Each call advances the job one step (check render status → upload finished renders to Meta → create ads in ad sets). Includes automatic de-duplication (queries Meta for existing ad video_ids, deletes duplicates, skips already-created ads).
3. `get_tour_creative_job {job_id}` — check final status and per-ad-set results.

The job goes through these phases: `rendering` → `uploading` → `adding` → `done` (or `failed`).

In the dashboard the same job shows in the "Processing" bar at the top of Tour
Campaigns: a render bar with a thumbnail strip (queued clips blurred, finished
videos sharp and playable on hover), then live Upload-to-Meta and Create-ads
bars with a card per video and a time estimate. Finished renders appear in the
campaign's media gallery (icon left of the rocket) as soon as they are stored,
grouped by tour date. Rendi has no cancel API: aborting a job only stops
song.so from collecting and using the results.

### Single and selected clips (Tour Creatives modal)

Generating one clip, all clips for a city, or a selection from the modal is a
**server-side render job** with its own id (it shows in the same "Processing"
bar, labelled "Rendering creatives"). The server collects the finished videos
and links them to the tour date, so the modal or the browser tab can be closed.
A per-minute background job finishes renders when no dashboard is open. These
renders only create video creatives for the tour date; uploading to Meta and
creating ads is a separate step (`add_tour_creative_to_adset`, or the bulk flow).

### Render clips for one city (server-side job)

`generate_tour_clips {tour_date_id, artist_key, base_clip_ids?}` renders the
artist's clips for ONE tour date as a server-side render job — the same as
generating from the Tour Creatives modal. Default: every clip that has no finished
render for that date yet. It only creates the videos (no Meta upload, no ads; use
`generate_tour_creatives` for the full flow or `add_tour_creative_to_adset` for a
single ad). The job finishes on the server even if nobody watches; follow it with
`get_tour_render_job {job_id}` (each call also nudges the server to collect
finished renders). Jobs also show in the dashboard's "Processing" bar.

### Media gallery

`list_tour_media_gallery {tour_link_id, tour_date_id?}` — every rendered video of a
tour link grouped by tour date (newest render per clip): playable URL, thumbnail,
whether it is on Meta, plus how many are still rendering and any failures. The same
content as the gallery icon next to the rocket in the dashboard.

### Inspecting renders

`list_tour_video_renders {tour_date_id}` — shows all renders for a specific tour date (city), including status (processing/success/failed), output_url, and meta_video_id (if uploaded to Meta).

### Manually adding a creative to an ad set

`add_tour_creative_to_adset {campaign_id, adset_id, tour_video_render_id}` — creates a Meta ad creative from a specific rendered video and adds it as a new ad in the specified ad set. The render must have `status: "success"` and a `meta_video_id`. Built-in de-duplication: if an ad with the same video_id already exists in the ad set, it's skipped.

The ad is built like the bulk flow builds it: no primary text or headline, call to action `ORDER_NOW` linking to the render's own tour date page (`https://song.so/<slug>?city=<city>`), and UTM tags ending in `utm_content={{ad.id}}`. If the destination cannot be determined the tool refuses instead of creating an ad that could not be attributed.

### Tour campaign optimization (inside song.so)

Tour campaigns (`campaign_type = tour`) are handled by their own hourly
optimizer, `meta-ads-tour-optimize`, never by the release optimizer. Always on:
an ad set whose tour date is **sold out** or **in the past** has all its active
ads and the ad set itself paused. With automation enabled (off by default for
tour campaigns): within a future, non-sold-out date, a paused ad with both a
better RCPR (spend per ticket click) **and** a lower frequency than an active
ad is swapped in; an ad set is never left with zero active ads. Ads whose name
contains "force" are exempt from everything. No Claude scheduled task is needed
or wanted for tour campaigns either.

## Release campaigns: dashboard changes of 2026-10-08/09

- **Dashboard click columns count TOTAL clicks.** In the /campaigns table the "song.so
  Clicks" column (and the RCPR and Conv. Rate derived from it) now counts every platform
  click-out attributed through `utm_content`, consented and non-consented. Before it counted
  only consented visitors, which made Meta's link clicks look better than song.so's numbers
  (RPC `meta_ads_song_clicks(..., p_consent_only := false)`; the default stays consent-only).
  The hourly optimizer keeps its own click count — compare with `get_optimization_log` metrics.
- **No default warm-audience exclusion** when creating a campaign (dashboard and
  `create_release_campaign`); see `auto_exclude_audiences`.
- **Flexible ads from batch videos**, **server-side upload queue** (table `meta_upload_jobs`,
  worker `meta-upload-worker`) and **server-side campaign jobs** (`meta_campaign_jobs` /
  `meta_campaign_steps`, worker `meta-campaign-worker`, used by the ad-set duplication modal) —
  100+ video campaigns never run only in the browser. `create_flexible_ad` must be routed to
  `meta-ads-create` (fixed for the MCP tools).
- **Ad set duplication** ("+" next to an ad set): copies an ad set with new interests and the
  same ads; use `meta_api` for the equivalent manual call.
- **Optimize button** per ad account and the stricter optimizer (hard RCPR cap, forced
  exploration of untested ads): tool `run_campaign_optimization`.

## Creative bulk batches (the /creatives bulk mode)

A finished batch (all clips animated) can be turned into many variants and exported as many
finished videos. Everything is queued server-side (worker `creatives-bulk-process`, cron every
minute), so nothing needs a browser.

1. `start_bulk_job {project_id, mode, item_ids?, skip_original_order?}` — **multi**: the same
   clips in N different orders (skip_original_order leaves out the order of the regular edit, so
   10 clips queue the 9 others); **single**: one variant per clip. `item_ids` = only the clips
   shown on the Clips step (archived/replaced clips stay out). Variants are lazy: only the clip
   orders/edits are built, text is rendered at export only.
2. `get_bulk_job {job_id}` / `list_bulk_jobs {project_id?, stage?}` — poll until completed.
   `stage: assemble` = variants, `stage: finalize` = exports (output_url = finished video).
3. `select_bulk_variants {job_id, selected, item_ids?}` — choose the variants to export.
4. `export_bulk_job {project_id, style_project_id, hooks?, lyrics?, overlay?, combined?}` —
   every selected variant × every caption recipe: one video per hook (hook only), lyrics only,
   overlay only, optionally one combined video (several captions; they can overlap). The look
   comes from the Text step settings of `style_project_id` (saved on the batch; duplicating or
   editing a batch loads them back). Each variant is merged once, then one text render per
   recipe — state "N variants × M recipes = X videos" before calling. If variants are still
   rendering, the export starts by itself afterwards.
5. `cancel_bulk_job` (stops, keeps outputs) / `discard_bulk_job {confirm: true}` (also deletes
   generated edits; a job with finished exports is only cancelled).
6. `list_batch_videos {smartlink_id}` → feed the finished videos into a campaign
   (`create_release_campaign` with `flexible_groups`).

Other creative changes: clips of removed/replaced images are **archived** (kept, ignored by merge,
animation queue and bulk); merging uses only the clips shown on the Clips step; random image
generation is a job with live progress and refunds for undelivered images; Pinterest video pins
resolve to their mp4; captions can be hidden per hook/lyrics/overlay (hidden = not rendered or
exported); the batch card is "Draft" until a text render exists.

## Resolving the smartlink for a release

1. `list_smartlinks {search: "<release title>"}` — `search` matches the
   **title only**, not the artist name. For artist-based lookup:
   `list_artists {search: "<artist>"}` → take `artist_id` → list smartlinks
   and filter client-side by `artist_id`.
2. No pagination — only `limit` (max 200). On large accounts (labels with
   hundreds of links) always use `search` instead of listing everything.
3. `get_smartlink {id}` (UUIDs from list results) for details: `slug`,
   `custom_url`, platforms `[{name, url, enabled, position}]`.
4. Public URL: usually `https://song.so/<slug>` (domain is song.so, not
   artistl.ink); `custom_url` overrides; user subdomains exist in some
   cases — confirm the final URL with the user before putting it in an ad.
5. Use type `smartlink` for release campaigns (not `bio`/`tour`/`presave`
   unless that's what's being promoted).

`list_pixels` returns `{id, name, meta_pixel_id, tiktok_pixel_id,
google_ads_conversion_id, google_ads_conversion_label, is_active,
artist_ids}` — use `meta_pixel_id` to confirm the campaign's
promoted_object pixel, and `artist_ids` to determine which artists the pixel
fires for (empty/null `artist_ids` = global). Secret API tokens are never
returned by any tool, including `add_pixel`/`edit_pixel` (their responses are
re-read through the same scoped fields even right after you set the token).

Errors come back as `Error: <message>`; "Smartlink not found or not
accessible" / "Tracking pixel not found or not accessible" = wrong ID or
belongs to another user.

## Managing pixels

- `add_pixel {name, meta_pixel_id?, meta_api_token?, meta_test_code?,
  tiktok_pixel_id?, tiktok_api_token?, tiktok_test_code?,
  google_ads_conversion_id?, google_ads_conversion_label?, is_active?,
  artist_ids?}` creates a pixel/connection; if a pixel with the same `name`
  already exists for the user it's updated in place instead of duplicated.
  `artist_ids` assigns it to artists immediately. A pixel with no
  `artist_ids` is global and fires for all of the user's smartlinks.
- `edit_pixel {pixel_id, ...same fields, artist_ids?}` updates only the
  fields you pass. **`artist_ids` REPLACES the full set of assigned
  artists** — always pass the complete desired list, not just additions.
  **Never reassign or remove a pixel's artists without explicit user
  confirmation.**

## Conversion rate from analytics

`get_smartlink_analytics {smartlink_id, start_date?, end_date?, timezone?}`
returns `total_views`, `total_clicks`, `click_through_rate_pct`,
`platform_breakdown [{platform_name, clicks}]`,
`country_breakdown [{country, visits}]`,
`daily_views [{date, views, paid_views, paid_clicks}]`, and
`daily_click_traffic [{date, paid, organic}]` (paid vs. organic platform
clicks per day — same RPC the song.so dashboard's own analytics view uses).

**Date-filter quirks — critical for correct reports:**

- `total_views` / `total_clicks` / `click_through_rate_pct` are **lifetime**
  values and IGNORE `start_date`/`end_date`. Never present
  `click_through_rate_pct` as the CVR "since launch" — it isn't.
- `platform_breakdown` and `country_breakdown` ARE filtered by
  `start_date`/`end_date`.
- `daily_views` and `daily_click_traffic` are timezone-aware but NOT
  date-filtered — they return all days; slice the range yourself.
- Always pass the user's IANA `timezone` (e.g. `Europe/Berlin`) so days
  line up with Meta's reporting; default is UTC.

**Computing the benchmark CVR (smartlink → DSP, target ≥60–70%) for the
campaign window:**

1. Preferred: sum `paid_clicks` ÷ sum `paid_views` from `daily_views` over
   the campaign dates — isolates ad traffic from organic.
2. If paid fields are 0/absent for the window: clicks = sum of
   `platform_breakdown` clicks (with `start_date`/`end_date` set to the
   window) ÷ views = sum of `daily_views.views` sliced to the same dates.
   Note that this mixes in organic traffic — say so.
3. Lifetime `click_through_rate_pct` is a last resort only, labeled as
   lifetime.

**Granularity caveat:** CVR is per smartlink, not per ad. All ads pointing
at the same link share one CVR — apply it as the 4th metric to every ad of
that campaign, and rely on CPC/CTR/play-rate to differentiate between ads.
For true per-ad CVR, use `get_platform_clicks` (see below).

Also useful: `platform_breakdown` shows where fans convert (Spotify vs.
others); `country_breakdown` sanity-checks that ad targeting matches actual
traffic. `list_artists` includes Spotify stats (`spotify_followers_total`,
`spotify_popularity`, `spotify_genres`) — handy context when picking
benchmarks or interest targeting.

## Comparing clicks against Meta campaign/ad set/ad IDs

`get_platform_clicks {start_date?, end_date?, timezone?, campaign_ids?,
adset_ids?, ad_ids?}` returns real landing-page click data aggregated by
Meta campaign/ad set/ad id — the same data the song.so **DSP Tracker**
Chrome extension shows next to Meta Ads Manager's own numbers. It's scoped
to Meta traffic only (UTM sources `ig`/`fb`/`meta`/`instagram`/`facebook`)
and returns `total_clicks`, `total_unique_visitors` (deduplicated —
don't try to derive this by summing per-row values), and
`by_campaign_id`/`by_adset_id`/`by_ad_id` maps of `{clicks,
unique_visitors}` keyed by the Meta id (this relies on the ad's URL being
tagged with `utm_campaign`/`utm_term`/`utm_content` = campaign/ad
set/ad id — confirm the user's ads are tagged that way before relying on
this). Use this to sanity-check Meta's reported link clicks against actual
song.so landing-page traffic for the same ids — large gaps can indicate
click fraud, bot traffic, or broken tracking.

**Matching workflow — three granularity levels:**

Run all levels in parallel (one `get_platform_clicks` call + one Meta MCP call
per level). Always use the same date range on both sides.

**Date-range alignment:** pass `date_preset: maximum` to the Meta MCP and omit
`start_date` on `get_platform_clicks` for new campaigns (launched recently) —
`maximum` resolves to the ad set creation date, and song.so all-time will cover
the same window since there can be no earlier UTM-tagged clicks. For older or
long-running campaigns, explicitly pass `start_date: "<launch date>"` to
`get_platform_clicks` to avoid inflating song.so numbers with pre-existing
organic traffic.

**Level 1 — Campaign:**
1. `get_platform_clicks {campaign_ids: ["<id>"]}` → `by_campaign_id` clicks.
2. Meta MCP `ads_get_ad_entities` at `level: campaign`, field `results` →
   total Results for the campaign.
3. Match: song.so campaign clicks ÷ Meta Results. 60–100% is normal.

**Level 2 — Ad Set:**
1. Same `get_platform_clicks` call → `by_adset_id` clicks.
2. Meta MCP at `level: adset`, field `results` → Results per ad set, filtered
   by campaign.
3. Match per ad set ID. Note: these frameworks typically use one ad set per
   campaign, so ad-set numbers = campaign totals — confirm before drawing
   conclusions from differences.

**Level 3 — Ad (per-ad CVR):**
1. Same `get_platform_clicks` call → `by_ad_id` clicks.
2. Meta MCP at `level: ad`, field `results` → Results per ad, filtered by
   campaign.
3. Match by ad ID: one Meta Result = one song.so platform click. Compare per
   ad — a match rate of 60–100% is normal (attribution window + iOS/cookie
   gaps). Significantly more song.so clicks than Meta Results on a single ad
   can indicate organic spillover or misattribution; significantly fewer can
   indicate pixel drop-off.

**Low match rate on a large, old campaign** (e.g. <30% overall despite healthy
per-ad rates on active ads) usually means UTM tags were added or fixed after
launch — the bulk of historical Results predate the tagging. In that case,
restrict both sides to the date range where tags were confirmed correct.
