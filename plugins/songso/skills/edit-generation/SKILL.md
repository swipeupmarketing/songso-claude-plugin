---
name: edit-generation
description: >-
  Generate, edit, animate, and assemble song.so music ad creatives. Use for
  the full Spotify-to-video pipeline: resolve and trim a Spotify preview,
  research Pinterest images, animate images, assemble raw videos with sound,
  generate hooks, add cumulative Typewriter lyrics, control the AI disclosure
  checkbox, and download final videos. Also use for image research,
  cropping/editing, variation generation, cinematic scenes, clip cutting,
  merging, and final downloads. Also covers the text layer: hook and lyrics
  fonts, colours, positions, lyrics sync styles and word timing, video effects,
  new hooks matched to the sound, and downloading the video with or without
  text. Trigger whenever the user mentions ad creative generation, Spotify,
  TikTok or YouTube creatives, image research, animate image, lyrics, sync
  style, Typewriter, fonts, text effects, video effects, hooks, AI badge, cut
  clips, merge clips, download clips, or creative edits. Also covers the bulk
  batch mode: turn one finished batch into many clip-order variants, choose
  variants, export them as videos with hook / lyrics / overlay recipes, and
  hand the finished videos to a Meta campaign. Trigger on bulk, variants,
  clip orders, export matrix, batch videos or flexible ads.
---

<!-- skill_package_version: 3.4 -->

# Edit Generation — Loader

This package intentionally stays thin. The full workflow (tool tables, Spotify
batch flow, bulk batch mode, text/lyrics/effects, cinematic scenes, safety and
cost rules) is maintained server-side by song.so and served live through the
song.so MCP connector, so it is always current without re-uploading this skill.

**Mandatory gate — before calling ANY creative tool this session:**

1. Call `get_meta_ads_playbook` (song.so connector, `https://mcp.song.so/mcp`)
   exactly once with `section: "edit-generation"` and
   `installed_version: "3.4"`, then follow the returned
   content for the rest of the conversation.
   - If the response starts with a `[skill package outdated: ...]` notice, tell
     the user once to redownload from `https://app.song.so/mcp` and re-upload it
     in Claude (Settings → Capabilities → Skills → Upload skill); this is only
     needed when this loader itself changes (e.g. its trigger description).
     Continue anyway.
2. If the tool is missing, start a new conversation (it re-fetches the
   connector's tool list); if it is still missing, ask the user to reconnect
   the connector (Claude Settings → Connectors → song.so → reconnect/refresh).
   Do not guess tool names, inputs or outputs — and do not invent song.so
   details you cannot verify.

Respond in English by default, or mirror the user's language.
