# Campaign creation recipe (Meta Graph API)

**Use `create_release_campaign` (song.so MCP) first** — it applies this recipe,
previews it with a dry run and registers the campaign in song.so so the hourly
optimizer covers it (see "Campaign creation walkthrough" in the main playbook).
The raw recipe below is for what the tool does not cover, or when the song.so
connector is unavailable.

Proven parameters from SwipeUp's production edge function, with fixes applied.
Use these exact specs when creating campaigns via the meta-mcp-wrapper (see
`references/meta-ads-mcp.md` for how to call it and for v23 API quirks:
`is_adset_budget_sharing_enabled`, bid_strategy placement, deprecated
`standard_enhancements`).

## Account values (always dynamic)

Never hardcode IDs. Every account-specific value — ad account ID, pixel ID,
Facebook page ID, Instagram user ID — comes from the first-time setup
(stored in memory) or is asked fresh:

- Ad account + pixel: from setup memory; if missing, run/redo the setup.
- Page/IG: from setup memory as the default identity; resolve the IG account
  via `GET /{page_id}?fields=instagram_business_account`.
  **Re-confirm the page/IG combo with the user before creating each
  campaign**, even when a default is stored.

## Campaign

```
objective:             OUTCOME_SALES
status:                PAUSED            (always — user reviews before launch)
special_ad_categories: []
bid_strategy:          LOWEST_COST_WITHOUT_CAP
name:                  "<Artist> - <Release>"
```

**Budget (fix vs. old function):** the old function put `lifetime_budget` on
the campaign. The framework wants **ABO** — put the budget on each ad set
(`daily_budget`, ~$20/day equivalent in account currency) and leave the
campaign without budget. Schedule: start = release date (or now if already
past), end = start + 30 days.

## Ad set

```
optimization_goal: OFFSITE_CONVERSIONS
billing_event:     IMPRESSIONS
status:            PAUSED
promoted_object:   { "pixel_id": "<pixel>", "custom_event_type": "CONTENT_VIEW" }
```

Targeting spec:

```json
{
  "age_min": 18, "age_max": 65,
  "geo_locations": { "countries": [...], "location_types": ["home", "recent"] },
  "flexible_spec": [
    { "interests": [/* up to 4 main interests — OR logic */] },
    { "interests": [/* up to 2 must-match interests — AND logic, e.g. Spotify */] }
  ],
  "publisher_platforms": ["instagram"],
  "instagram_positions": ["reels", "story"],
  "device_platforms": ["mobile"],
  "targeting_automation": { "advantage_audience": 0 }
}
```

- `advantage_audience: 0` disables Advantage+ Audience — required (AI off).
- `device_platforms`: mobile only (fix — old function included desktop;
  pointless for Reels/Stories).
- **Geo targeting — ask the user to pick one (AskUserQuestion)**, unless they
  already gave an explicit country list:
  - **GSA** = Germany (DE), Austria (AT), Switzerland (CH) only. Always
    German-language (`"locales": [5]`) — never combine GSA with the tier
    lists below; it's a separate, mutually exclusive market strategy, not
    additive. (`geo_preset: gsa`)
  - **Balanced** = Tier 1 + the ten best Tier 2 markets, no Brazil or Mexico.
    Default recommendation whenever the user wants International reach and
    has no strong preference. (`geo_preset: balanced`)
  - **Tier 1** (highest per-stream payout, roughly >$0.004): IS, NO, DK, SE,
    FI, CH, NL, IE, LU, GB, IL, US, DE, AT, FR, BE, AU, NZ, CA, HK.
    (`geo_preset: tier1`)
  - **Tier 2** (good payout, roughly $0.0025–$0.004): IT, ES, PT, EE, LV, LT,
    CZ, SK, GR, CY, SI, HR, HU, PL, JP, KR, SG, AE, QA, KW.
    (`geo_preset: tier2`)
  - **Tier 3** (high volume, lower payout): BR, MX, AR, CO, CL, PE, TR, ZA, IN,
    ID, TH, VN, PH, MY, EG, NG, MA, KE. (`geo_preset: tier3`)
  - **Balanced** list: IS, NO, DK, SE, FI, CH, NL, IE, LU, GB, IL, US, DE, AT,
    FR, BE, AU, NZ, CA, HK, IT, ES, JP, KR, SG, PT, AE, PL, CZ, HU.
  - **EU**: DE, FR, NL, BE, AT, CH, IE, SE, NO, DK, FI, IT, ES, PT, PL, RO, CZ,
    HU, GR, HR, BG, SK, SI, LT, LV, EE, LU, MT, CY. (`geo_preset: eu`)
  These are the same presets as the song.so dashboard wizard (based on
  Spotify per-stream royalty rates), so a campaign built by tool or by hand
  targets the same countries the dashboard would.
  - **Custom** — user-specified country list (ISO 3166-1 alpha-2 codes) in
    `geo_locations.countries`.
- **Language targeting — ask separately (AskUserQuestion)**, applies as
  Meta's `locales` targeting field (skip this question for GSA — it's
  always German, see above):
  - **No language filter** (default unless the user has a reason to
    restrict) — omit `locales` entirely.
  - **English (All)** — resolve the exact Meta locale id with
    `search_meta_targeting {params: {type: "adlocale", q: "English"}}` (see
    "Resolving interest/geo/locale IDs" in `references/meta-ads-mcp.md`;
    verified live as key `1001` at the time of writing, but re-check rather
    than hardcoding it).
  - **Other** — ask which language, then resolve its locale id the same way.
- Interest IDs: Spotify `6002969794329` is known-good. For a music genre,
  check `references/meta-music-genre-interests.md` first (~120 pre-resolved
  genre/subgenre IDs, e.g. Pop music `6003341579196`); for anything else,
  resolve via `search_meta_targeting` (`type=adinterest`, then validate the
  id with `type=adinterestvalid`) rather than guessing — see
  `references/meta-ads-mcp.md`.
- Ad set structure (min. 2): **Broad** = only Spotify as interest
  (`flexible_spec: [{interests:[Spotify]}]`, no must-match layer).
  **Genre + Spotify** = genre interests as OR in the first `flexible_spec`
  entry, Spotify as AND in the second — ask the user for the genre first.
  Electronic example: `[{interests:[Electronic dance music, House music,
  Dance music, Electronic music]}, {interests:[Spotify]}]`. Resolve interest
  IDs via the targeting search endpoint or Ads Manager suggestions.

## Creative & ad

```json
object_story_spec: {
  "page_id": "<page>",
  "instagram_user_id": "<ig_user>",
  "link_data": {
    "link": "<song.so smartlink from user>",
    "message": "\"<Release>\" out now!",
    "call_to_action": { "type": "LISTEN_NOW", "value": { "link": "<smartlink>" } }
  }
}
```

**AI enhancements off:** add to the creative. Verified working on v23.0 for
link AND video creatives (`standard_enhancements` is deprecated there):

```json
"degrees_of_freedom_spec": {
  "creative_features_spec": {
    "ig_video_native_subtitle":      { "enroll_status": "OPT_OUT" },
    "image_animation":               { "enroll_status": "OPT_OUT" },
    "product_browsing":              { "enroll_status": "OPT_OUT" },
    "product_metadata_automation":   { "enroll_status": "OPT_OUT" },
    "profile_card":                  { "enroll_status": "OPT_OUT" },
    "standard_enhancements_catalog": { "enroll_status": "OPT_OUT" },
    "text_overlay_translation":      { "enroll_status": "OPT_OUT" }
  }
}
```

(Feature names shift between API versions and creative types — if Meta
rejects a key, its error lists the exact valid set for that creative; opt out
of all of those. If the field can't be set, keep the campaign paused and tell
the user to switch enhancements off in Ads Manager before activating.)

**No Multi-Advertiser Ads (important):** additionally opt the creative out of
multi-advertiser ads: `"contextual_multi_ads": {"enroll_status": "OPT_OUT"}`
(top-level creative field; wrapper v12+ forwards it). If the API rejects the
placement of the field, follow the error message — and always verify the
"Multi-advertiser ads" checkbox is OFF in Ads Manager before activating.

For **video ads** (the standard case: ~20 video creatives), replace
`link_data` with `video_data: { "video_id": <uploaded id>, "image_url":
"<thumbnail — release cover if no mapping>", "message", "call_to_action" }` —
upload each video file directly to Meta first (multipart, never an
intermediate URL host) via `mcp_meta_ads_upload_video`; see
"Uploading creatives" in `references/meta-ads-mcp.md`. `instagram_user_id`
can be omitted — Meta uses the page-connected IG account.

Ad: `status: PAUSED`, `creative: { "creative_id": <id> }`. At least 1 ad per
campaign; framework ideal is the 3-ad-set structure (Broad / Artist
Interests / Spotify+Genre) with ~20 creatives total.

## After creation

Return/report: campaign ID, ad set ID(s), creative/ad IDs, and the Ads
Manager deep link
`https://business.facebook.com/adsmanager/manage/campaigns?act=<account>&selected_campaign_ids=<campaign_id>`.
Everything stays PAUSED until the user reviews and activates.

song.so's hourly optimizer only covers campaigns that are registered in song.so
(created in the dashboard or imported there). A campaign built directly through
the Meta API / Meta Ads MCP is **not** registered, so tell the user it is not
auto-optimized yet and that importing it in the dashboard ("Campaigns → Add
campaign") puts it under the automation. Do not set up a Claude scheduled task
for monitoring — see "Automated optimization (runs inside song.so)" in the main
playbook.
