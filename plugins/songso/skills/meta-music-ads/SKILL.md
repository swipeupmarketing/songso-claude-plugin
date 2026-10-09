---
name: meta-music-ads
description: >-
  Plan, launch, and optimize Meta (Instagram/Facebook) ad campaigns for music
  releases using the song.so performance framework — campaign structure, market
  benchmarks (CPC/CPA/CTR/play rate/conversion rate), kill/keep decisions per
  ad, and the Friday-to-Monday optimization workflow. Can also operate Meta Ads
  Manager directly in the browser to read metrics and pause underperforming
  ads, create full campaigns via the Meta Ads MCP/Graph API, and explain the
  hourly optimization that song.so runs on its own. Use this skill whenever the user mentions
  music ads, song/release
  promotion on Meta or Instagram, Spotify growth campaigns, ad benchmarks for
  artists, evaluating or optimizing ads for a release, "which ads should I turn
  off", release-day ad checks, or asks to check/optimize campaigns in Ads
  Manager for an artist — even if they don't say "Meta ads" explicitly.
---

<!-- skill_package_version: 3.4 -->

# Meta Music Ads — Loader

This package intentionally stays thin. The full playbook (strategy, market
benchmarks, campaign setup, and the Friday-to-Monday optimization workflow) is
maintained server-side by song.so and served live through the MCP connector,
so it can be updated at any time without requiring a new package download.

**Mandatory gate — before calling ANY other song.so or Meta ads tool this
session:**

1. Look for a `get_meta_ads_playbook` tool (from the `songso` / song.so MCP
   connector — `https://mcp.song.so/mcp`). If present, call it **first**,
   exactly once per session, with
   `installed_version: "3.4"` (the `skill_package_version`
   noted above) — this single call both loads the current main playbook
   (always fresh, independent of the installed package) and checks whether
   the `.skill` package itself is outdated.
   - If the response starts with a `[skill package outdated: ...]` notice,
     tell the user once to redownload it from `https://app.song.so/mcp` and
     re-upload it in Claude (Settings → Capabilities → Skills → Upload
     skill) — this is only needed when the package structure itself changes
     (new bundled files, changed activation rules), not for routine
     playbook/content updates. Then continue anyway; don't block on it.
   - Follow the (rest of the) returned playbook content for the rest of the
     conversation.
2. If you need one of the deeper reference docs bundled below, you can either
   read the local file directly (`references/*.md` — still bundled here for
   offline/degraded use) or call `get_meta_ads_playbook` again with the
   matching `section` argument (`songso-mcp`, `campaign-creation-api`,
   `ads-manager-browser`, `meta-ads-mcp`, `songso-cvr-comparison`, or
   `meta-music-genre-interests`) to get the latest version — no need to
   pass `installed_version` again after the first call.

**If a tool the playbook mentions isn't available** (e.g. `get_meta_ads_playbook`
itself, or a song.so/Meta tool referenced in the loaded content): don't guess
or fabricate its output. MCP tool lists are typically only refreshed when a
connector session (re)connects, so first try starting a **new conversation**
(it re-fetches the connector's tool list). If the tool is still missing, tell
the user to refresh the connection: Claude **Settings → Connectors →
song.so → reconnect/refresh** (disconnect and add it again if there's no
refresh action). There is currently no reliable way to force this from
inside a conversation — don't claim you can refresh it yourself.

**If the song.so MCP connector isn't connected** (no `get_meta_ads_playbook`
tool available at all): tell the user this skill works best with it connected
(Claude Settings → Connectors → Add custom connector →
`https://mcp.song.so/mcp`), then continue in a degraded mode using only the
bundled reference docs in `references/` and whatever the user tells you
directly — do not invent song.so-specific details (smartlinks, pixels,
benchmarks) that you can't verify.

**Language: respond in English by default**, unless the user writes in
another language — then mirror theirs.

# Multi-Pixel Handler

The song.so pixel tracking system uses a global multi-pixel handler that works at the account level, not per-smartlink:

- **No direct assignment needed**: Pixels are not assigned to individual smartlinks. Instead, they work globally for the user's account.
- **Account-level coverage**: If no artists are assigned to a pixel, it fires for all smartlinks in the account.
- **Artist-level targeting**: If one or more artists are assigned to a pixel, it fires for all smartlinks belonging to those artists.
- **Simplified management**: This eliminates the need to manually configure pixels for each smartlink and ensures consistent tracking across an artist's entire presence on song.so.
