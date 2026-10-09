# Operating Meta Ads Manager via Claude-in-Chrome

How to read metrics and pause ads in Meta Ads Manager
(https://adsmanager.facebook.com) using the `mcp__Claude_in_Chrome__*` tools.

## Setup

1. Load the Chrome tools in one ToolSearch call (tabs_context_mcp, navigate,
   computer, read_page, get_page_text, find).
2. Call `tabs_context_mcp` first; if Ads Manager is already open, reuse the tab.
3. Navigate to `https://adsmanager.facebook.com/adsmanager/manage/ads`. If a
   login screen appears, ask the user to log in — never enter credentials.
4. If multiple ad accounts exist, confirm with the user which account (and
   which campaign) to work on before reading anything.

## Reading metrics

Work at the **Ads** tab level (not Campaigns/Ad Sets) — kill/keep decisions
are per ad.

Needed columns: Impressions, Link clicks, CPC (cost per link click), CTR (link
click-through rate), 3-second video plays. If a column is missing, use the
"Columns" dropdown → "Customize columns" to add it. Prefer *link* clicks/CTR
over "clicks (all)" — "all" inflates numbers with profile taps and reactions.

Set the date range to "since launch" of the campaign (usually last 3–7 days,
matching the Friday-release timeline).

Derive:
- 3s play rate = 3-second video plays ÷ impressions
- CPA (cost per ViewContent) from the Results/Cost-per-result columns when the
  campaign optimizes for ViewContent

The smartlink → DSP conversion rate is NOT in Ads Manager — it comes from the
song.so dashboard. If the user can't provide it, evaluate on the metrics you
have and note the gap.

Use `get_page_text` / `read_page` to extract the table. Ads Manager tables
render lazily — scroll horizontally/vertically if values are missing and
re-read. Double-check that numbers line up with the right ad name; when
uncertain, zoom into a row rather than guessing.

## Pausing ads

Only after the user has confirmed your kill list:

1. Find the ad's row and click its toggle switch (left of the ad name).
   Toggle = pause. Never use Delete.
2. Verify the toggle state changed / status shows "Off" before moving on.
3. Pause one ad at a time and confirm each state change — Ads Manager UI can
   lag, and a double-click re-enables the ad.

Never: delete ads/campaigns, edit budgets, change targeting, duplicate
campaigns, or touch billing — unless the user explicitly asks for that
specific action.

## After acting

Report back: which ads were paused and why (which metrics failed), which stay
live, and what the next scheduled check is per the weekly workflow.
