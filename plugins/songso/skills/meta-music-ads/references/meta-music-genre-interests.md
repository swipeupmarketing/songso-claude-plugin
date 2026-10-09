# Meta music genre interests (pre-resolved IDs)

A pre-resolved lookup table of Meta ad-targeting interest IDs for music
genres/subgenres, so a common request ("target metal fans", "add K-pop as
an interest") doesn't need a live `search_meta_targeting` round-trip. Use
this table first; only fall back to a live search (see "Finding more" below)
for a genre/subgenre not listed here, or if you need to confirm an ID is
still valid/deliverable (`type: "adinterestvalid"`) before using it in a
real campaign — Meta interest IDs occasionally get renamed/retired, so for
anything high-stakes (about to spend real budget), validate rather than
trust this table blindly if it's been a while since the table was refreshed.

Audience sizes are Meta's own broad estimates (worldwide, all ages/genders)
at the time this table was built — treat them as rough scale indicators for
picking a niche-vs-broad interest, not exact current numbers.

## Table

### Pop

| Name | Interest ID | Audience size |
|---|---|---|
| Pop music | `6003341579196` | 994.6M–1.17B |
| Popular music | `6003199871665` | 550.0M–646.8M |
| K-pop | `6003339140579` | 257.2M–302.5M |
| Latin pop | `6003091232616` | 162.4M–191.0M |
| Pop rock | `6003697803783` | 116.1M–136.6M |
| Dance-pop | `6003336556256` | 54.5M–64.1M |
| J-pop | `6003110209472` | 48.0M–56.5M |
| V-pop | `6005722532204` | 39.7M–46.7M |
| Pop punk | `6003358555604` | 22.9M–27.0M |
| Mexican pop music | `6002950184974` | 21.1M–24.8M |
| Indian pop | `6003073724597` | 19.6M–23.1M |
| Traditional pop music | `6002999936173` | 17.7M–20.8M |
| Mandopop | `6003386919512` | 16.7M–19.7M |
| Cantopop | `6003190630726` | 15.9M–18.7M |
| Electropop | `853985594662078` | 14.7M–17.3M |
| Arabic pop music | `6003107527835` | 13.8M–16.3M |
| Thai pop music | `6017801666711` | 10.8M–12.7M |
| Pinoy pop | `6003361706460` | 6.7M–7.9M |
| Country pop | `6003629273583` | 1.4M–1.7M |

### Rock

| Name | Interest ID | Audience size |
|---|---|---|
| Rock music | `6003582732907` | 959.5M–1.13B |
| Rock and roll | `6003104445629` | 316.6M–372.4M |
| Alternative rock | `6003314955799` | 314.6M–370.0M |
| Hard rock | `6003673812662` | 158.4M–186.2M |
| Progressive rock | `6002933925173` | 141.0M–165.8M |
| Indie rock | `6003225780498` | 128.3M–150.9M |
| Soft rock | `6003377208805` | 116.1M–136.6M |
| Thai rock | `6003278226799` | 61.2M–71.9M |
| Grunge | `6003352237730` | 60.5M–71.1M |
| Blues rock | `6003371145740` | 59.1M–69.6M |
| Britpop | `6004176353295` | 56.8M–66.8M |
| Garage rock | `6002970980594` | 50.9M–59.9M |
| Psychedelic rock | `6003126178149` | 36.3M–42.7M |
| Post-punk | `6003328637897` | 24.8M–29.2M |
| Experimental rock | `6002976495459` | 14.9M–17.5M |
| Classic rock | `6003139217861` | 13.0M–15.3M |
| Post-grunge | `6002879533372` | 6.5M–7.7M |
| Country rock | `6003460127825` | 4.8M–5.6M |
| Dream pop | `6003467417547` | 2.4M–2.9M |

### Hip-Hop / Rap

| Name | Interest ID | Audience size |
|---|---|---|
| Hip hop music | `6003225556345` | 824.4M–969.5M |
| West Coast hip hop | `6003184558161` | 131.6M–154.8M |
| East Coast hip hop | `6003092152617` | 126.6M–148.9M |
| Southern hip hop | `6003195522298` | 123.6M–145.4M |
| Alternative hip hop | `6003224474661` | 121.9M–143.3M |
| French hip hop | `6003339411221` | 116.4M–136.9M |
| Korean hip hop | `6012875345920` | 112.8M–132.6M |
| Japanese hip hop | `6002927517562` | 109.9M–129.3M |
| Gangsta rap | `6003498332480` | 40.2M–47.2M |
| Trap (music) | `6010119006911` | 37.1M–43.6M |
| Freestyle rap | `6003259015102` | 32.4M–38.1M |
| Drill | `6002998649892` | 21.1M–24.8M |
| Underground hip hop | `6003128658328` | 16.9M–19.9M |
| Hot R&B/Hip-Hop Songs | `6002972712194` | 1.4M–1.7M |

### R&B / Soul / Funk

| Name | Interest ID | Audience size |
|---|---|---|
| Rhythm and blues music | `6003195554098` | 669.2M–787.0M |
| Soul music | `6003107699532` | 461.2M–542.4M |
| Contemporary R&B | `6003180715102` | 334.0M–392.8M |
| Funk | `6003902444066` | 222.6M–261.7M |
| Funk carioca | `6003432194483` | 9.8M–11.5M |
| Funk ostentação | `6016917656327` | 5.4M–6.3M |
| R&B/Soul | `6003968219348` | 0.3M–0.4M |

### Electronic / Dance

| Name | Interest ID | Audience size |
|---|---|---|
| Electronic music | `6003902397066` | 763.3M–897.6M |
| House music | `6003479860669` | 232.5M–273.4M |
| Electro (music) | `6003395797326` | 160.8M–189.0M |
| Electronic dance music | `6003155409305` | 127.9M–150.4M |
| Electronica | `6002920671555` | 116.1M–136.5M |
| Techno | `6002911345572` | 97.9M–115.1M |
| Ambient music | `6003305648021` | 48.9M–57.5M |
| Trance music | `6003805457383` | 47.2M–55.6M |
| Electro house | `6003288039651` | 26.0M–30.6M |
| Lo-fi music | `6003121974734` | 22.8M–26.8M |
| Drum and bass | `6003537481663` | 18.5M–21.8M |
| Dubstep | `6003022971356` | 11.0M–13.0M |
| Deep house | `6003596378473` | 8.1M–9.5M |

### Metal

| Name | Interest ID | Audience size |
|---|---|---|
| Heavy metal music | `6003633122583` | 603.6M–709.9M |
| Metal | `6003276721810` | 134.3M–158.0M |
| Extreme metal | `6003453238242` | 31.7M–37.3M |
| Glam metal | `6003430691469` | 19.3M–22.7M |
| Thrash metal | `6003475887071` | 18.1M–21.3M |

### Punk

| Name | Interest ID | Audience size |
|---|---|---|
| Punk rock | `6003280772823` | 196.6M–231.3M |
| Hardcore punk | `6003378873765` | 95.5M–112.3M |
| Skate punk | `6002985722723` | 3.0M–3.6M |

### Country / Americana

| Name | Interest ID | Audience size |
|---|---|---|
| Country music | `6003493980595` | 470.6M–553.4M |
| musica ranchera | `6003300315938` | 57.8M–67.9M |
| Texas country music | `6003305455369` | 8.9M–10.5M |

### Jazz / Blues / Classical

| Name | Interest ID | Audience size |
|---|---|---|
| Jazz music | `6003146442552` | 462.4M–543.7M |
| Blues music | `6003257757682` | 453.9M–533.8M |
| Classical music | `6002951587955` | 324.3M–381.3M |
| Opera | `6003628789383` | 49.0M–57.6M |
| Latin jazz | `6004111763189` | 14.7M–17.3M |
| Contemporary classical music | `6003274787171` | 6.3M–7.4M |
| Afro-Cuban jazz | `6003254226816` | 0.3M–0.4M |

### Reggae / Caribbean

| Name | Interest ID | Audience size |
|---|---|---|
| Reggae | `6003309820509` | 230.4M–270.9M |
| Reggaeton | `6003434636451` | 139.6M–164.1M |
| Dancehalls | `6003247890613` | 103.4M–121.6M |
| Reggae en Español | `6003364047823` | 26.3M–30.9M |

### Latin (Mexico / South America)

| Name | Interest ID | Audience size |
|---|---|---|
| Latin music (genre) | `853932481331810` | 186.6M–219.4M |
| Cumbia | `6003359723621` | 67.5M–79.3M |
| Samba | `6003302709853` | 62.9M–74.0M |
| Banda (music) | `6003288085327` | 58.6M–69.0M |
| Salsa music | `6003668231507` | 49.6M–58.3M |
| Música popular brasileira | `6013004584342` | 46.9M–55.2M |
| Bachata (music) | `6003143764440` | 46.6M–54.7M |
| Merengue music | `6003332917632` | 37.5M–44.1M |
| Mexican cumbia | `6003497930280` | 28.3M–33.2M |
| Argentine cumbia | `6003985876266` | 25.5M–30.0M |
| Peruvian cumbia | `6003686938107` | 24.2M–28.5M |
| Salsa romántica | `6003309367514` | 5.7M–6.7M |
| Cumbia villera | `6002910067372` | 2.3M–2.7M |
| Latin Music | `6003452434671` | 1.4M–1.7M |

### World / Folk

| Name | Interest ID | Audience size |
|---|---|---|
| World music | `6003226755338` | 160.7M–189.0M |
| Folk music | `6003290182925` | 45.9M–54.0M |
| Afrobeat | `6003484812986` | 14.5M–17.0M |
| Indie folk | `6003278095799` | 8.3M–9.8M |

### Indie / Alternative (general)

| Name | Interest ID | Audience size |
|---|---|---|
| Indie pop | `6003278637610` | 74.5M–87.7M |
| Indie | `6004169139624` | 2.4M–2.9M |
| Indie music scene | `6003199285265` | 0.4M–0.5M |

### Other

| Name | Interest ID | Audience size |
|---|---|---|
| Disco | `6003320931941` | 105.0M–123.4M |
| Emo (music) | `6003347724581` | 62.4M–73.4M |
| Grime (music) | `6003061520481` | 0.6M–0.7M |

## Finding more, or something more specific

If the genre/subgenre you need isn't above (a hyper-specific microgenre, an
artist-adjacent niche, a regional scene not listed), search live with
`search_meta_targeting` rather than guessing an ID:

1. **Free-text search:** `{"params": {"type": "adinterest", "q": "<genre>"}}`.
   Try a few phrasings if the first doesn't return a good match — Meta's
   interest catalog is inconsistent about whether a genre exists as
   `"X"`, `"X music"`, or `"X (music)"`.
2. **Validate before using in a real campaign:**
   `{"params": {"type": "adinterestvalid", "interest_fbid_list": ["<id>", ...]}}`
   — confirms the ID is still valid/deliverable rather than trusting a name
   match. Prefer this over `adinterest` search results for anything you're
   about to spend budget on.
3. **Related suggestions:** `{"params": {"type": "adinterestsuggestion", "interest_list": ["<Known Interest Name>"]}}`
   surfaces interests Meta considers related — useful for building out a
   must-match/broad interest set around a genre you already have an ID for.
4. **Watch for false-positive noise in free-text search results** — Meta's
   search is fuzzy and surfaces a lot of unrelated matches for short/common
   genre-adjacent words. Filter what comes back, don't take every result at
   face value. Known traps hit while building this table:
   - `disco` matches "Discount stores", "Discovery Channel", "Discounting"
   - `ska` matches "Skateboarding", "Skatepark", "Skateboard"
   - `country` matches "Backcountry skiing", "Cross-country running",
     "Country club", "Country Road" (clothing brand)
   - `house` matches "Coffeehouses"
   - `pop` matches "PopSugar", "Popsicle", "Popcorn"
   - `opera` matches "Opera (web browser)"
   - `rap`/`trap` match "Rapid transit", "Drilling rig", "Drill bit"
   - Rapper/artist names (e.g. "Drake (rapper)", "Daft Punk") come back
     mixed in with genre results — these are real, valid interests, but
     they're artist fandom, not a genre; don't merge them into a genre list
     unless that's specifically what's wanted.
5. **`adTargetingCategory` with `class: "interests"`** returns a smaller,
   curated top-level browse list (a few hundred entries total, mostly major
   categories) rather than the full searchable catalog — useful for
   sanity-checking major genres exist as first-class interests, not for
   finding subgenres or regional scenes (those only surface via free-text
   `adinterest` search).
6. **Rebuilding/extending this table:** if doing a broader sweep again,
   search a batch of genre-family seed terms (pop, rock, hip hop, r&b,
   electronic/edm/house/techno, metal, punk, country, jazz, blues,
   classical, reggae, latin/cumbia/salsa/bachata, k-pop/j-pop/mandopop/
   cantopop, world/folk, indie/alternative — see the category headers
   above for the full seed list actually used), collect + dedupe by
   interest ID across all results, then apply the false-positive filtering
   in point 4 before treating anything as genre-relevant.
