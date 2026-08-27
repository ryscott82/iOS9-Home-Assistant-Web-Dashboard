# Changelog

All notable changes and features to the Home Assistant iPad Mini Web Dashboard are documented chronologically in this file by version number.

## [v4.9.2] - 2026-08-27
- **Buffered grouped media players through automatic track switches**: When a track changes on a group of speakers (e.g. one track auto-advancing on a HomesPod-equivalent / multi-room group), Home Assistant reports the new title/artist/artwork one player at a time. Previously each arriving update ran the grouping logic immediately, so mid-switch one player already reported the NEW track while the others still held the OLD one — the group got different grouping keys and briefly split into separate cards before re-merging (the card separation/glitch). Now a transition buffer glues the whole group together for a 15-second window: the moment any member reports a new track, everyone still on the old title is locked into a shared sticky group until the laggards catch up, so the group stays in ONE card throughout the switch and re-groups naturally once everyone agrees. The buffer triggers from both the WebSocket realtime path and the REST poll path, merges with overlapping sticky groups (so players finishing the switch one-at-a-time never get competing keys), and only fires on track-relevant attribute changes (volume nudges are ignored). The card's displayed title/cover now majority-votes among the group members' reported tracks instead of picking the first member, so a single lagging player can't flap the title between old and new — the new track/cove appears the moment the majority has it.

## [v4.9.1] - 2026-08-27
- **Home page volume slider now reliably follows the album cover color**: The Home tab's extended media card loads its dominant color asynchronously from the artwork, and the previous code only recolored the card's background/border/glow once the color arrived — the volume bar fill stayed slate-blue until the next full re-render. The async color extraction now also repaints every `.media-large-vol-fill` bar (fill + right-edge accent) inside the home media card the moment the color is ready, so the slider matches the artwork immediately even when a freshly-queued track's cover wasn't cached yet.
- **Home page control cards are now translucent like the pool & hot tub cards**: Light switches, dimmers and other control tiles on the Home tab use semi-transparent backgrounds instead of the opaque white/near-black tiles, giving the whole control grid the same glassy look as the pool and hot tub heater cards. In light mode every Home-tab card (including the at-temp pool/hot tub state and all regular control tiles) sits at a uniform very-light fill — `rgba(255,255,255,0.10)` — while dark mode uses a subtler translucent dark (`rgba(22,27,34,0.35)`). The pool/hot tub cards now fill yellow/orange **only while the current temperature is at or below the set temperature** (actively heating or at target); once the water climbs above the set point they fall back to the same faint translucent white as the other cards, with just an orange border as the heater-on indicator. Active/glowing states and the inline-styled media card are unaffected.
- **Shadows stay fully black in dark mode**: The home tiles' glow shadows render solid black (`rgba(0,0,0,...)`) in dark theme instead of the white-tinted light-mode shadows, and the home media card's surrounding glow uses a darkened variant of the album's dominant color so it reads as a shadow rather than a colored halo at night.

## [v4.9.0] - 2026-08-26
- **Enlarged home screen buttons for distance readability**: The "All Controls" and "Log Chemicals" buttons on the Home page now use 62px height and 22px font size (up from 52px / 17–18px) so they're easier to read and tap from across the room.
- **Home screen Party Mode toggle button**: A new "🎉 Party Mode" button sits to the right of "All Controls" on the Home screen. Tapping it toggles party mode on/off via `input_select.house_mode` — the same action as the 🎉 button in the All Controls header. The button highlights with a pink accent when a party is active, matching the existing party-mode visual language.

## [v4.8.3] - 2026-08-24
- **Minute-cadence black-bar guard**: The dashboard hides the iOS status bar by scrolling itself 24px past the black-translucent band, but Safari occasionally resets that offset to zero (rubber-band bounce, keyboard, tab restore), re-exposing a black strip at the top of the screen in light mode. A shared `ensureBlackBarHidden()` helper now drives the scroll on every tab change and the first render (as before), plus a once-a-minute check that nudges the page back down whenever the bar has crept into view — without ever yanking a deeper scroll on long pages.
- **Reverted the artwork cache-buster propagation (album covers stopped loading)**: v4.8.1 made the Media/Party page card and `prewarmAlbumColors` request artwork through `getGroupTrackInfo`'s `_v=` cache-busted URL. In practice some media integrations serve signed CDN/proxy artwork URLs that reject modified query strings, so covers stopped appearing entirely. The page card is back to using the raw `entity_picture` URL and dominant-color prewarming keys on it again; only the Home tab's small thumbnails keep the buster, exactly as before v4.8.1. The REST-vs-WebSocket freshness guards from v4.8.1 remain in place.

## [v4.8.2] - 2026-08-24
- **Party page no longer forces light mode**: `isDashboardDarkTheme()` lost its party gate — parties now follow the normal night / manual-dark-override rules like every other page, so the Party page goes dark at night again. Self-dimming (dim overlay + night dimming) is still suppressed mid-party, and the lock screen stays disabled. All party-page surfaces gained dark-theme variants: the welcome-message strip, the new pool/spa shortcut cards, and the existing exit/back buttons.
- **Pool & spa light shortcut cards**: The top of the Party page's left column now has two half-width cards for `PARTY_POOL_LIGHTS_ENTITY` (`light.scott_pool_lights`) and `PARTY_SPA_LIGHT_ENTITY` (`light.spa_light`) — big high-contrast inline SVGs (pool ladder + waves / steaming hot tub, each with a yellow bulb badge) with a bold label, toggling their light on tap via the normal `toggleEntity` path. They highlight yellow while on: each card carries the standard `card_<entity_id>` id, so the realtime targeted DOM updater flips the active class instantly. These two entities and anything whose id or friendly name contains "outdoor" are removed from the generic card stack below (they remain on the Home tab).

## [v4.8.1] - 2026-08-24
- **Fixed song info briefly reverting to the previous track**: The request-sequence guard only ordered concurrent REST responses against each other — a `/api/states` snapshot captured *before* a WebSocket `state_changed` event but landing *after* it overwrote the fresher state in `entities[]`/`lastStatesCache` and repainted the old song until the next 3s poll. A new `isNewerState()` freshness check (ISO `last_updated` comparison; missing values still apply) now guards all three apply paths: the 3s poller's media branch, its non-media branch (which also synced unconditionally), and `refreshAll`'s full rebuild (newer-wins per entity before rendering).
- *(Its artwork cache-buster change was reverted in v4.8.3 — see above.)*

## [v4.8.0] - 2026-08-23
- **New Party Page (Automatic Mode-Driven View)**: When `input_select.house_mode` is set to `Party` (case-insensitive), the dashboard auto-switches to a dedicated full-screen Party page; when house mode leaves `Party` while viewing it, the dashboard returns to Home automatically. Details:
  - **Forced light mode**: While a party is active the dashboard always renders in light theme — night mode and any manual dark override are ignored until house_mode leaves `Party` (maximum perceived brightness on the wall panel). Both theme application paths in `renderDashboard` (initial class assignment and the per-render body rebuild + background colors) are gated through `isDashboardDarkTheme()`.
  - **Full brightness (no self-dimming)**: The page-dim overlay systems (`input_boolean.dim_dashboard` and automatic night dimming) are suppressed during parties; they resume automatically when it ends.
  - **Lock screen never shows**: The dashboard's night/away splash lock screen (the camera-feed overlay) is fully disabled while house mode is `Party` — regardless of alarm state, time of day, kitchen-light state, the 30-minute unlock window or a previously forced lock. The manual "Lock Now" settings button is also refused mid-party with a toast. Normal lock behavior resumes automatically when the party ends.
  - **Two-column layout (56/44, fits 1024x768)**: The left column stacks the same `ipadhomescreen` control cards as the Home tab one-per-row (identical click behavior and stable element IDs, so realtime targeted DOM updates keep working; Pool & Hot Tub heater cards override their half-grid width to span the full column with extra height headroom). `PARTY_HIDDEN_ENTITIES` holds substring patterns matched against entity_id (`"theater"`, `"game_room"`, `"kitchen"`) so every theater, game-room and kitchen light is excluded from this column regardless of how many exist. The right column holds a QR code panel on top ("Scan to queue music", rendered at 8px/module with a slim 2-module quiet zone for a bigger, easier-to-scan code) with the Family Room media card below it — sized so both fit one 768px-tall landscape screen without scrolling.
  - **No chrome**: The entire header (welcome message + clock), footer, chemical banner, sparse bottom clock and All Controls button are hidden on this tab for maximum space. A small floating circular Home button pinned to the top-right corner (explicit `min-height` so the global button reset can't stretch it) provides an escape hatch since header/footer navigation is gone, and a one-line welcome strip (`input_text.dashboard_welcome_message`) sits above the left column's controls — standard bold body text at 56px that never wraps (long messages clip with an ellipsis so the layout can't shift).
  - **QR code generation**: A minimal strict-ES5 QR encoder (adapted from Project Nayuki's MIT-licensed QR Code generator library; fixed parameters — version 3, ECC level M, byte mode, mask 2, 42-byte capacity) renders `PARTY_QR_URL` once to a canvas and caches the PNG data URL. Verified decoding correctly with an independent QR decoder across multiple render scales.
  - **Always-visible media card (compact multi-player variant)**: The right column shows the media group containing `PARTY_MEDIA_ENTITY` (`media_player.family_room_apple_tv`) rendered by the same renderer the Media tab uses for multiple playing groups (`renderMediaCardLayout` with `cardCount` 2) — i.e. the compact card with no crisp album-cover square, just the speaker badge, artist/title info and transport + per-speaker volume controls over a subtle tinted background — even when nothing has played for over 15 minutes or the player is off/idle. Party-page CSS applies grid-4-style compaction (46px buttons/pill, 38px volume bars) to suit the narrower column; when idle, the card's built-in Play pill starts the music. Group members playing the same track are bundled using the same sticky/title grouping as the Home/Media tabs.
  - **Auto-page rotation suppression**: Media-started playback no longer force-switches to the Media tab during a party (the Party page IS the now-playing view), and the play-pause return-home timer plus the 5-minute inactivity return timer are suppressed so the dashboard stays pinned to the Party page until house mode changes. The hot tub chemical red alert also no longer yanks the dashboard to the Chemicals page mid-party — the alert latch is held open so it fires as soon as the party ends if the water is still red. Media playback-state tracking keeps running during parties, so ending a party can't produce a phantom "just started playing" edge that would jump to the Media page.
  - **Manual party toggle (🎉)**: All Controls' header row gains a 🎉 button right next to Settings: one tap posts `Party` to `PARTY_MODE_ENTITY` via `input_select/select_option` (or `PARTY_EXIT_MODE`, default `Day`, when a party is already running). The button gets pink accent styling while a party is active, and the resulting state change flows through the normal poll → render path so all party behavior engages itself.
  - **Back-to-Party pill**: The bottom navigation bar is hidden on the Home page, so during a party a floating "🎉 Back to Party" pill appears bottom-center of Home to return to the Party page after manually navigating away.
  - **Sub-page header buttons visible in party mode**: The Settings/Reload/Home buttons in the All Controls and Chemical Status headers picked their icon color from raw night/dark state instead of `isDashboardDarkTheme()`, rendering white icons on the forced-white party background (invisible). Now gated like every other theme decision.
  - **Live updates**: Track changes now re-render the Party tab instantly alongside Home/Controls (`renderTabsForMediaChange`).
  - **Navigation guards**: Auto-switching fires only on actual house-mode transitions; manual navigation away mid-party is unaffected. A stale cached `party` tab in `localStorage` after a reload recovers to Home, and a reload during an active party lands directly on the Party page.
  - New CONFIG constants: `PARTY_MODE_ENTITY`, `PARTY_QR_URL`, `PARTY_MEDIA_ENTITY`, `PARTY_HIDDEN_ENTITIES`, `PARTY_EXIT_MODE`.

## [v4.7.1] - 2026-08-20
- **Connection Settings Simplified (IP-Only)**: The setup / connection card no longer asks for a Long-Lived Access Token — the token is managed in the dashboard source file (`HA_TOKEN` constant, or a previously saved value in `localStorage`). Only the server URL / IP is editable now (with the same Default IP and mDNS quick buttons), which is all that's needed after a power-cycle changes the server address. Saving preserves the existing stored token.
- **Settings & Connection Cards Respect Dark Mode Fully + Larger Text**: All overlay chrome (page backdrop, card, headings, descriptions, labels, inputs, checkboxes, close button) now has proper dark-theme styling via shared classes (`settings-heading`, `settings-row-title`, `settings-desc`, `settings-input`) instead of hardcoded inline colors. Text sizes bumped across both cards (headings 28px, descriptions 14px, inputs 18px, row titles 17px, buttons 15–19px).
- **Dark Mode Ambient Album Tint**: The background tint from currently-playing album art (previously light-mode-only) now works in dark mode too — the same dominant-color gradient is pulled down near-black (`albumDarkWash`) so it stays subtle, fading to black instead of white. The `html.dark-theme` overscroll rule dropped `!important` so the inline tint can take over while music plays and falls back to pure black when idle.
- **Robustness Fixes**:
  - **No more setup-screen on transient errors**: Two failed polls used to kick the wall-mounted iPad into the Connection Settings card. Now only an HTTP 401/403 (rejected token) opens it — with an explanatory message — while network blips just show the red status dot and an occasional toast.
  - **Optimistic play/pause & volume revert on failure**: If `media_play_pause`, `volume_set` or turn-on-then-play fails, the UI now restores the previous state/volume (and volume bar fill) instead of lying until the next poll.
  - **Clean diff baseline**: `entities[]` now stores shallow clones (`es5ShallowClone`) so optimistic UI mutations ("Skipping...", volume nudges) on `lastStatesCache` no longer poison the poller's change detection into producing phantom re-renders.
  - **WebSocket exponential backoff**: Reconnect delays double from 3s up to a 60s cap during outages instead of hammering HA every 3s; reset on successful open.
  - **Poller covers non-media DOM when WS is down**: The 3s poller now detects non-media changes via `last_updated` and refreshes those tiles in place, so automation-driven brightness/volume changes don't sit stale for up to a minute during WebSocket outages.
  - **Single fetch at startup**: The 3s REST poller starts only after first data arrives (WS initial states or first successful refresh) instead of firing alongside them — boot no longer triple-pulls `/api/states`.
- **Cleanup**: Removed the dead duplicate `switchTab` definition (the persistent one at the bottom always won) and the never-referenced `mediaSetVolume` function.
- **Artwork failure fallback**: If a new album cover fails to download, the player's previous artwork stays committed (`__mediaArtError`) so the next render keeps showing the old cover instead of degrading to a blank card.

## [v4.7.0] - 2026-08-20
- **Track Changes Now Land Instantly (0:00 Boundary Refresh)**: Track titles, artists and albums frequently appeared stuck because attribute-only track changes (title/artist/album/artwork while the player stays `playing`) were applied silently to the caches by the WebSocket handler — the Media page only saw them on the next 15-second full refresh. Now:
  - **WebSocket track changes render immediately**: The realtime handler diffs every media_player update against the previous state (`mediaDisplayDiff` — state, title, artist, **album name**, artwork, volume) and re-renders the Media page the moment any display attribute changes. Album-name-only changes (previously not diffed at all) are now detected too.
  - **0:00 / end-of-track fast poll**: The 1-second progress ticker detects track boundaries — position rolling over to 0:00, touching the end of `media_duration`, or a new duration appearing — and fires an immediate one-shot `/api/states` pull (throttled to once per 2s) so the new track's metadata lands right as the progress bar resets. The 0:00 trigger only fires when *entering* the zero zone, so a player parked at 0:00 can't loop polls.
- **Album Art & Color Continuity**: The previous album picture and its extracted dominant color now stay on screen until the new ones are fully loaded, eliminating blank flashes and slate-blue tint flashes on every track change:
  - Media page cards render the **previous** cover on both the blurred backdrop and the crisp single-card art; a hidden preloader `<img>` swaps the backdrop (`__mediaArtLoaded`) and the new cover cross-fades in via opacity transition once it has downloaded. URLs are compared with the `_v=` cache-buster stripped (`normalizeArtUrl`) so same-album tracks don't needlessly restack.
  - Card tint falls back through: exact-art color cache → album-key cache → **this player's previous color** (`lastEntityColor`, now actually written) → default slate blue, so glows/pills/sliders never flash generic colors mid-extraction.
  - Home-tab cards preload new artwork with an in-memory `Image()` and swap the `<img>` src only on load.
- **Streamlined Data Pulling & Rendering**: One fetch pipeline, no redundant UI work:
  - **Unified 3s poller** (`pollStatesHA`): replaces the media-only `pollMediaPlayersHA`. It is the only REST polling fetcher — media players are diff-applied (UI work only when something visible changed), and all non-media entities are absorbed silently into the caches so the WebSocket stays the primary path and the slow loop stays a pure reconcile.
  - **Signature-gated full refresh**: `refreshAll` (15s) now computes a cheap whole-dashboard fingerprint (`buildGlobalStateSig`: state + last_changed per entity, plus media display attributes) and skips `renderDashboard` entirely when nothing changed — previously it rebuilt the whole DOM every 15s even when idle. A forced render every 4th cycle (~1 min) keeps purely time-based rules (splash lock window, long-paused idle hiding) fresh.
  - **Targeted re-render policy**: title/artwork/volume changes update the Home and All Controls tabs strictly in place (zero flicker); full rebuilds there happen only on play-state flips. The Media page rebuilds on any display change since titles are baked into its HTML.
  - Removed the redundant per-second `updateTargetedEntityDOM` sweep from the ticker (the WebSocket + unified poller own those updates now); the ticker only advances progress bars/times.
- **No White Top Bar in Dark Mode**: The 20px scrollable top bar (`#scrollBlackBar`), the `html.dark-theme` overscroll background, and the iOS status-bar style are now black / `black-translucent` in **both** themes. Dark mode previously rendered a white status-bar-height strip (with black status text) at scroll-top; now the bar blends into the black page, white status text stays readable at scroll-top and blends into the background when scrolled — matching light mode's behavior.

## [v4.6.7] - 2026-08-15
- **Album-Art Colors Now Apply Fast & Reliably (No More Waiting on a Page Change)**: Dominant-color extraction used to start only when a card rendered, and each render fired a fresh duplicate image download that styled only whatever card happened to be in the DOM when it finished — so the card tint / volume slider colors appeared late or not at all until a tab switch or refresh. Now:
  - **Pre-warm extraction**: `renderDashboard` kicks off color extraction for every currently-playing (or paused) media player's artwork the moment fresh states arrive, so by the time the Media page or home cards render, the tint is usually already cached and applied instantly (`prewarmAlbumColors`).
  - **Single-load waiter registry**: `extractAlbumDominantColor` now deduplicates in-flight loads per artwork and notifies **every** registered waiter (media card, home card, page tint) when the color finishes computing — even if the DOM was rebuilt while the image was loading. Colors therefore always land on whatever is on screen now.
  - This complements the v4.6.5 persistent Media page container (which stops cards being torn down every 15s) and the v4.6.6 uniform group tint for volume sliders.

## [v4.6.6] - 2026-08-15
- **Grouped Media Cards Stay Together During a Skip**: When multiple speakers are playing the same song and one of them skips a track (Next/Previous), the software now assumes **all** speakers in the group skipped for **15 seconds** instead of letting the group card split apart and re-merge. Previously the Media page grouped speakers by raw title/artist and its Next/Previous buttons sent the skip without the optimistic grouping, so the card broke into separate cards (each speaker reporting the track change at its own pace) and then snapped back together. Now:
  - The Media page uses the same sticky-group-aware grouping as Home / All Controls, so the whole speaker group is held in one card for the skip window even while their individual state updates arrive out of order.
  - Media page cards also route their displayed title through the skip-filter logic (`getGroupTrackInfo`), so a member that lags behind and still reports the old song is treated as having skipped too — the card shows the new track (or "Skipping...") instead of reverting to the old song mid-transition.
  - The optimistic skip window and the sticky-group window are both **15 seconds** (was 8s/10s) to fully cover Home Assistant update time.
- **Uniform Volume Slider Colors**: Volume sliders inside a grouped media card now all share a single color derived from the group's primary player, so every bar matches while the speakers are playing the same thing. Two per-entity color leaks were fixed: the All Controls page volume bars (`renderVolumeControls`) used each speaker's own dominant color, and WebSocket in-place updates re-colored individual sliders per-entity — both now preserve the group's uniform tint and only update the fill width.

## [v4.6.5] - 2026-08-15
- **Media Page Flicker & "Stuck Song" Fixes**:
  - **Persistent Media Page Container**: The Media page now renders into its own persistent `#mediaPageHost` container that sits outside `#entityContainer`. Previously the whole Media page (including every album-art `<img>` and `background-image` div) was destroyed and re-created on every 15-second refresh and on any unrelated state change — on iOS 9 WebKit this re-decoded and re-painted the artwork, causing constant flicker. The media DOM now survives re-renders and is only repopulated when the actual media data changes (title, artist, album, artwork, state), so album art only ever loads once.
  - **Stale Response Race Fixed**: `refreshAll` (15s) and `pollMediaPlayersHA` (3s) both fetch `/api/states` concurrently with no ordering, so a slow older response could land AFTER a newer one and roll the dashboard back to stale data — which presented as the Media page getting stuck on an old song. Both pollers now share a monotonic request sequence (`stateReqSeq` / `stateReqApplied`); a response is only applied if it was issued after the last applied one.
  - **Album Art Double-Load Eliminated**: Removed `img.crossOrigin = "Anonymous"` from `extractAlbumDominantColor`. HA's `media_player_proxy` sends no CORS headers, so on iOS 9 the CORS load always failed and triggered a guaranteed second, duplicate artwork download (double flash + double bandwidth). The proxy URL is same-origin with `HA_URL`, so a plain load is read by the canvas without tainting.

## [v4.6.4] - 2026-08-15
- **Lock Screen Schedule Tied to Kitchen Lights**: The night lock screen now locks at **12:30 AM** and stays locked until the kitchen overhead lights (`light.kitchen_kitchen_ceiling_lights`) are switched on in the morning — no more time-based auto-unlock at 7:00 AM or house-mode heuristics. If the kitchen lights never turn on, the iPad stays locked (until the user taps "Tap to Unlock" for the usual 30-minute window). Locking/unlocking is re-evaluated on every 15-second state refresh, so the display unlocks within seconds of the lights coming on.

## [v4.6.3] - 2026-08-15
- **Lock Screen Camera Feed Rebuilt on Native `?token=` URLs (fixes black/decode failures)**: The XHR + base64 data-URL approach kept failing on iOS 9 WebKit — `responseType="arraybuffer"` silently yields nothing, and `overrideMimeType("text/plain; charset=x-user-defined")` is ignored so `btoa` receives UTF-8-mangled text and throws. Replaced the entire binary pipeline with the mechanism the HA frontend itself uses: load the snapshot as a plain `url()` CSS background from `/api/camera_proxy/<entity>?token=<access_token>`. The camera's rotating access token is read from the entity's state attributes (`attributes.access_token`, published from `/api/states` and rotated every 5 min, last two kept valid) so iOS 9 WebKit decodes the JPEG natively — no XHR, no btoa, no data URL. Retries every 10s while visible, reloads automatically when the token rotates, and the on-screen status line now reports "Cam unavailable (no token)" when the entity/token is missing.

## [v4.6.2] - 2026-08-15
- **Lock Screen Camera Feed Fix: Binary-String Fetch + Retry + Status**: Replaced the flaky `xhr.responseType="arraybuffer"` fetch (which silently yields nothing on older iOS 9 WebKit, leaving the lock screen black) with the bulletproof `overrideMimeType("text/plain; charset=x-user-defined")` binary-string transport converted via chunked `btoa` — works on every iOS Safari version. Lowered the camera request to `width=960&height=540` (HA scales JPEG frames down by discrete factors, returning a light image a 2012 iPad Mini can decode). Added a 30s timeout, automatic retry every 10s while the lock screen is visible, and a subtle on-screen status line (`#splashCamStatus`) that shows "Loading camera feed…" while fetching and "Camera feed unavailable" on failure so issues are visible instead of a silent black background.

## [v4.6.1] - 2026-08-15
- **Lock Screen Camera Feed Set to Album Slideshow & Robust Fetch**: Set the default `CAMERA_ENTITY` to `camera.album_slideshow_kitchen_icloud_album` (Album Slideshow iCloud Shared Album camera). Hardened the lock screen snapshot fetch to read the response `Content-Type` header (handles PNG stills from the slideshow integration, not just JPEG) and added a cache-busting `t=<timestamp>` query parameter so the slideshow's latest rendered frame is always fetched instead of a stale cached JPEG.

## [v4.6.0] - 2026-08-15
- **Lock Screen Camera Feed Background**: The night / away splash lock screen now supports a Home Assistant camera snapshot as its full-screen background. Configurable via `CAMERA_ENTITY` and `CAMERA_REFRESH_MS` in `index.html` (or Settings → Display → Lock Screen Camera on-device). Snapshots are fetched over XHR from `/api/camera_proxy/<entity>` with the Bearer token, converted to a base64 data URL via chunked `btoa` (no `fetch` / `responseType=blob` dependency), and refreshed every 5 minutes (configurable 1-60 min) only while the lock screen is visible. Includes a subtle dark gradient overlay to keep the welcome message and unlock button readable on bright scenes. Fully iOS 9 Safari compatible.

## [v3.18.1] - 2026-08-08
- **Media Page Animation Removal & Processing Optimization**: Completely purged continuous CSS keyframe animations (`glassSweep` sheen animation and `eqWave` equalizer wave animation) and background timer routines from the Media page. Replaced equalizer waves with static indicator bars to significantly reduce GPU and CPU processing load on the iPad Mini Gen 1 (iOS 9.3.5 / WebKit).

## [v3.18.0] - 2026-08-08
- **Daytime Album Art Lightening & Contrast-Protected Text**: Updated media player card rendering to lighten album cover artwork backdrops during the day (`rgba(255,255,255,0.45)` top overlay fading to `0.98` white at bottom) instead of darkening them. Introduced `getMediaCardStyleInfo` helper to force **black text** (`#1c1c1e`) during daytime and **white text** (`#ffffff`) at night, with dynamic contrast switching if the backdrop luminance ($Y = 0.299R + 0.587G + 0.114B$) requires text color inversion for perfect readability.

## [v3.17.3] - 2026-08-08
- **Pure Image Average RGB Color Extraction Fallback**: Purged all hash-based color generation functions (`hslToRgb` and `getArtworkHashColor`). The fallback for album artwork color extraction is now strictly calculated as the true mathematical average RGB color (`overallAvgRgb`) of all pixels in the album art image. Default slate blue (`{ r: 65, g: 122, b: 160 }`) is strictly reserved for cases where no album art is provided at all (`!artSrc`).

## [v3.17.2] - 2026-08-08
- **Album Art 4-Tier Color Extraction Hierarchy**: Overhauled `extractAlbumDominantColor` in `index.html` to process pixels through a 4-tier selection hierarchy: (1) Vibrant non-skin hues, (2) Warm / skin-tone / brown / tan hues, (3) Monochrome black / white / gray pixel colors, and (4) Artwork hash-derived HSL color (for browser CORS security errors). Default slate blue (`{ r: 65, g: 122, b: 160 }`) is now strictly reserved for cases where no album art is provided at all.

## [v3.17.1] - 2026-08-08
- **Album Art Dominant Color Warm/Skin-Tone Fallback**: Enhanced `extractAlbumDominantColor` in `index.html` to collect separate `primaryBuckets` and `skinBuckets`. If an album cover has no vibrant non-skin-tone colors (e.g., warm beige, brown, or tan monochromatic backgrounds), the extraction algorithm falls back to the top warm/skin-tone bucket instead of defaulting to slate blue (`{ r: 65, g: 122, b: 160 }`).

## [v3.17.0] - 2026-08-08
- **Removed All Off Feature**: Completely purged the "All Off" feature from the dashboard. Removed the "🌙 Test All Off" footer button, late-night auto-scheduled "All Off" Home screen card grid button, sleep mode grid "All Off" pill button, override state variables (`forceAllOffButtonOverride`), and `turnAllOff()` / `toggleAllOffButtonOverride()` JavaScript service execution functions.

## [v3.16.1] - 2026-08-08
- **Fixed Chemical Status Button Click Handler & iOS 9 Touch Pass-Through**: Resolved an issue where tapping chemical beaker buttons failed to trigger navigation on iOS 9 WebKit. Added `chem-alert-btn` class exception to `handleCardClick` to prevent parent tile click handlers from capturing touch events, added `pointer-events: none` to the embedded SVG icon so touch events land directly on the button element, and introduced atomic `openChemicalsTab(vol, event)` helper to set target volume and switch tabs in a single DOM render pass.

## [v3.16.0] - 2026-08-08
- **Persistent Pool & Hot Tub Chemical Status Buttons**: Added persistent chemical beaker status buttons to both the Pool and Hot Tub cards. Buttons dynamically indicate water chemistry status with dynamic color-coding: **Blue** (`#0a84ff`) when water chemistry is normal, **Deep Yellow** (`#d97706`) for warning-level recommendations, and **Red** (`#ff3b30`) for critical chemical alerts. Tapping the chemical button automatically switches the active volume view to that body of water (`setRecVolume('pool'|'hottub')`) and navigates directly to the Chemicals & Sensors tab (`switchTab('chemicals')`).

## [v3.15.3] - 2026-08-08
- **Removed Bottom Clock Container Box & Compact Spacing**: Removed the background container box (`background: transparent; border: none;`) from the Home page sparse bottom clock (`.sparse-bottom-clock`). Reduced margins (`margin-top: 4px; margin-bottom: 8px; padding: 6px 0;`) and scaled typography (`54px` time, `18px` date) so it sits flush directly under the last card to maximize vertical screen space.

## [v3.15.2] - 2026-08-08
- **Set Static IP Address to `http://192.168.40.89:8123`**: Updated `HA_URL` and setup placeholders in `index.html` to stick directly to the static IP address `http://192.168.40.89:8123`.

## [v3.15.1] - 2026-08-08
- **Set Default Home Assistant Address to mDNS (`http://homeassistant.local:8123`)**: Set `http://homeassistant.local:8123` as the primary default server address. mDNS resolution is natively supported by iOS 9 Safari / macOS, ensuring the dashboard remains connected across power surges and router DHCP lease renewals without needing static IP maintenance.

## [v3.15.0] - 2026-08-08
- **Post-Power Surge Connection Recovery & Full Settings Overlay**: Added a full **Connection Settings Modal** (`showSetup()`) accessible via the new `⚙️ Settings` footer button or by tapping "Connection Status". Allows users to quickly view, edit, and test their Home Assistant URL/IP (with 1-tap presets for default IP `http://192.168.40.88:8123` and mDNS `http://homeassistant.local:8123`) and Long-Lived Access Token. Automatically pops up the settings modal if persistent network errors occur after router/server reboots.

## [v3.14.2] - 2026-07-31
- **iOS 9 WebKit Deterministic Home Screen Layout Calculation**: Replaced DOM `querySelectorAll` querying (`calculateHomeRows()`) with a deterministic count calculated directly from JavaScript array lengths (`mediaGroups` + `homeEntities`). Completely eliminates DOM reflow loops, class toggling lag, and greeting font size glitching on iOS 9 Mobile Safari.

## [v3.14.1] - 2026-07-31
- **Eliminated Home Screen Greeting Font Glitch & Timing Flicker**: Fixed a bug where WebSocket state updates or polling rerenders wiped out the `sparse-home-layout` class on `document.body`, causing `#headerWelcome` to glitch from 68px to 22px and back every few seconds. Preserved `sparse-home-layout` during `showTab()` calls and executed `checkSparseHomeLayout()` synchronously.

## [v3.14.0] - 2026-07-31
- **Ground-Up Clean Rewrite of All Controls Media Cards & Purged Duplicate Function**: Completely purged a duplicate leftover `renderMediaPlayer` function definition in `index.html` that was overriding active media cards with an old plain-white compact layout. Every media player card on the All Controls page now renders as a standalone 100% full-width card (`flex: 1 1 100%`) with an 85% opacity blurred album art backdrop (`filter: blur(22px) saturate(1.4)`), a 42px Play/Pause button glowing with the album's extracted dominant color, and a full volume control slider bar.

## [v3.13.1] - 2026-07-31
- **Enhanced Paused State Card Layout & Artwork Fallback**: Updated `renderMediaPlayer` on the All Controls page so players in the `PAUSED` state (with a loaded track) trigger full active card styling. Includes robust `artUrl` extraction across all media entity attributes, dynamic title-hash backdrop gradients when image URLs are absent, all 3 control buttons (`Previous`, `Play/Pause`, `Next`), and full volume control bars.

## [v3.13.0] - 2026-07-31
- **Vibrant Blurred Album Backdrop & 100% Full-Width Media Cards on All Controls Page**: Rewrote `renderMediaPlayer` on the All Controls page so active media players render as 100% full-width cards (`flex: 1 1 100%`). Features an 85% opacity blurred album art backdrop (`filter: blur(22px) saturate(1.4)`), a translucent glass tint overlay (`linear-gradient`), a 42px Play/Pause button glowing with the album's extracted dominant color, and a full volume control bar at the bottom.

## [v3.12.1] - 2026-07-31
- **Fixed Volume Controls & Album Art Cache Key in All Controls Media Tiles**: Fixed `artUrl` cache key generation in `getGroupTrackInfo` by removing dynamic timestamps so album dominant colors cache reliably across renders. Restored `renderVolumeControls(group, false)` on `renderMediaPlayer` tiles to ensure volume control sliders and buttons are prominently displayed on active media tiles.

## [v3.12.0] - 2026-07-31
- **Dynamic Blurred Background & Dominant Color Control Highlights on All Controls Page**: Active media player tiles on the All Controls page (`renderMediaPlayer`) feature a 24px blurred backdrop image of the playing album cover with a light (`rgba(255,255,255,0.82)`) or dark (`rgba(13,17,23,0.78)`) glass tint overlay depending on current theme mode. Highlighted playback controls (Play/Pause button) dynamically use the album's extracted dominant color (`extractAlbumDominantColor`) for background fill and glow shadow.

## [v3.11.8] - 2026-07-31
- **Fixed All Controls Media Player Rendering**: Replaced `renderMediaCardLayout` (which was rendering the large Media page background cards) with `renderMediaPlayer` inside `tile-grid` on the All Controls page (`activeTab === "controls"`). Media player cards on the All Controls page now render as clean compact tiles without album cover artwork and with buttons fitting perfectly inside.

## [v3.11.7] - 2026-07-31
- **Compact Media Tiles on All Controls Page**: Removed album cover images from media tiles on the All Controls page (`renderMediaPlayer`). Scaled control buttons to compact 30px/34px circles (`gap: 6px`) to ensure buttons fit comfortably inside each tile without overflowing or wrapping.

## [v3.11.6] - 2026-07-31
- **Inline Home Icon Button on Non-Home Tabs**: On non-Home tabs (`All Controls`, `Chemicals`, `Media`), the Home icon button (🏠) is placed in the exact same inline flex row directly to the right of `.tab-bar` (`.menu-bar-wrapper`), hiding the separate top header button to eliminate line breaks. On the Home tab, the full-width segmented tab bar and top header hamburger menu (☰) are active.

## [v3.11.5] - 2026-07-31
- **Restored Clean Full-Width Segmented Tab Bar**: Completely removed the standalone extra blue square Home icon button (`.menu-home-btn`). Restored `.tab-bar` as a clean, full-width 4-tab segmented control containing `[ Home ] [ All Controls ] [ Chemicals ] [ Media ]`.

## [v3.11.4] - 2026-07-31
- **Eliminated Duplicate Home Icon Button**: Updated top right `headerNavBtn` to remain strictly as the 3-line Hamburger Menu icon across all tabs instead of morphing into a Home icon, ensuring `.menu-home-btn` directly to the right of the menu bar is the single Home button on the screen.

## [v3.11.3] - 2026-07-31
- **Full-Width Menu Bar with Single Dedicated Home Icon Button**: Expanded `.tab-bar` to stretch across the full width of the screen minus the Home button (`flex: 1; min-width: 0`), and removed the duplicate text `Home` button from inside `.tab-bar` so that `.menu-home-btn` on the right acts as the single Home button.

## [v3.11.2] - 2026-07-31
- **Left-Aligned Compact Menu Bar with Adjacent Home Button**: Wrapped `.tab-bar` in a `.menu-bar-wrapper` flex container aligned on the left (`flex: 0 1 auto`), placing a dedicated Home icon button (`.menu-home-btn`) directly to its right for instant navigation to the Home screen.

## [v3.11.1] - 2026-07-31
- **Simplified Top Header**: Completely removed date and time text from the top header container. The welcome message (`#headerWelcome`) is now displayed exclusively on the Home tab (`activeTab === "home"`), and automatically hidden on all other tabs for a clean, distraction-free view.

## [v3.11.0] - 2026-07-31
- **Single Active Player Full-Space Layout**: When only 1 media player is active on the Media page, the card expands to fill the entire available height (`height: calc(100vh - 180px)`). Features a 32px blurred album art backdrop (`blur(32px)`), an unblurred crisp `320px x 320px` album cover art image inside the card frame, and enlarged UI controls (`38px` track title, `60px` play/pause pill, `60px` skip buttons, and `36px` volume bar).

## [v3.10.1] - 2026-07-31
- **Enlarged Home Media Card Album Art & Play Button**: Increased album art thumbnail size to `88px x 88px` (`border-radius: 18px`), enlarged play/pause overlay button to `52px` circle with `40px` SVG icon, and scaled skip button to `54px x 54px` for prominent visibility and easy tapping from across the room.
- **Home Media Card Navigation**: Tapping anywhere on the middle/text area of the Home media player card navigates directly to the Media page (`showTab('media')`).

## [v3.10.0] - 2026-07-31
- **Dynamic Album-Tinted Home Media Cards**: Replaced static generic blue background tint (`.active-blue`) on the Home page with dynamic background, border, and glow tinting derived directly from the currently playing album's extracted dominant color.

## [v3.9.7] - 2026-07-31
- **Vibrant Saturated Light Mode Card Glow (`getLightModeGlowRgb`)**: Replaced heavy black drop shadows in light mode with a luminous 38px saturated color halo (`rgba(gR, gG, gB, 0.58)`), creating a neon-like ambient glow surrounding media cards in light mode while leaving dark mode untouched.

## [v3.9.6] - 2026-07-31
- **Light Mode Album-Tinted Speaker Badge**: Tinted the light mode speaker badge background with a soft pastel blend of the extracted album color (`tr + (255 - tr) * 0.82`) and a matching color border (`rgba(tr, tg, tb, 0.35)`), leaving dark mode untouched (`rgba(tr, tg, tb, 0.22)`).

## [v3.9.5] - 2026-07-31
- **Solid Pure White Speaker Badge in Light Mode**: Updated `.media-card-speaker-badge` in light mode to solid pure white (`#ffffff`) background, `#1c1c1e` dark text, and subtle drop shadow (`0 2px 8px rgba(0,0,0,0.15)`), leaving dark mode untouched (`rgba(tr, tg, tb, 0.22)`).
- **High-Visibility Dark Artist Name in Light Mode**: Updated `.media-card-artist-name` in light mode to bold `#1c1c1e` text (`font-weight: 850`), making artist names crisp and readable across the room.

## [v3.9.4] - 2026-07-31
- **Richer Light Mode Album Color Gradient**: Reduced white blending from 70% to 35% (`0.35 * white blend`) on the bottom background gradient in light mode, restoring rich, vibrant album color tone while keeping dark mode unchanged.
- **High-Contrast White Speaker Badge in Light Mode**: Updated `.media-card-speaker-badge` background in light mode to `rgba(255, 255, 255, 0.94)` with subtle shadow, leaving dark mode speaker badge untouched (`rgba(tr, tg, tb, 0.22)`).

## [v3.9.3] - 2026-07-31
- **Light Mode Soft Pastel Color Gradient**: Configured bottom background gradient in light mode to fade into a light pastel shade of the dominant album color (`tr + (255 - tr) * 0.7`), while maintaining the deep darkened shade in dark mode (`0.30 * RGB`).

## [v3.9.2] - 2026-07-31
- **Restored Dominant Album Color Bottom Gradient**: Restored bottom card gradient overlay to transition smoothly into a rich, deep darkened shade of the extracted dominant album color (`0.30 * RGB`), matching the seamless ambient album color fade (e.g., deep royal blue for Djo *End of Beginning*).

## [v3.9.1] - 2026-07-31
- **Dark Mode Light Vibrant Accent Boost (`ensureLightVibrantTint`)**: Dynamically boosts color brightness for dark extracted album colors in dark mode so UI buttons (play pill, volume slider, equalizer wave, outer glow) render in light, luminous, high-contrast accent colors against `#0d1117` dark card backgrounds.
- **Theme-Adaptive Bottom Gradient Overlay**: Configured card background overlay to fade down into dark `#0d1117` in dark mode and pure white `#ffffff` in light mode.

## [v3.9.0] - 2026-07-31
- **Dynamic 1/2 Height Grid for > 3 Media Players**: Automatically applies `.media-page-grid.compact-grid` (245px 1/2 height cards) when > 3 media players are active on the Media page, fitting up to 6 media players on a single screen without vertical scrolling.
- **Mini Album Art Cards on All Controls Page**: Replaced basic media text tiles on the All Controls page with mini album cover cards featuring background artwork, speaker badges, play/pause controls, and volume sliders.
- **Home Page Media Card Tap Navigation**: Tapping anywhere on the middle/text area of a media player card on the Home page instantly navigates to the dedicated Media page (`showTab('media')`).

## [v3.8.0] - 2026-07-31
- **Half-Height Volume Control Bar**: Compacted `.media-large-vol-bar` height by 50% down to `25px`, with scaled `36px x 25px` stepper buttons (`16px` SVG icons) and `12px` bold volume percentage readout.
- **Curved Bottom Border Progress Bar**: Positioned a 6px progress track at the absolute bottom edge of media cards (`.media-card-bottom-progress-track`). Because `.media-page-card` uses `border-radius: 20px; overflow: hidden`, the progress fill seamlessly curves along the bottom 20px rounded border frame.
- **Ultra-Compact Media Card Layout**: Removed time duration and position text numbers to maximize vertical card compactness and eliminate visual clutter.

## [v3.7.0] - 2026-07-31
- **Dynamic Highlight-Color Bottom Fade Overlay**: Replaced static black/white bottom background fades with a dynamic gradient fading into a rich, deep darkened tint of the album's extracted highlight color (`0.30 * RGB`). Provides an ambient color glow across the bottom of media player cards while maintaining 100% WCAG AAA contrast ratio and legibility for white text and controls.

## [v3.6.3] - 2026-07-31
- **Color Histogram Bucket Scoring Algorithm**: Implemented a 12-hue histogram quantization bucket algorithm weighted by saturation squared (`sat * sat`). Eliminates dull gray tinting on album covers with dark hair or shadows (such as America's *Ventura Highway* album), cleanly isolating the true vibrant golden yellow/orange sunset sky for UI controls and card glows.

## [v3.6.2] - 2026-07-31
- **Perimeter-First Album Color Sampling**: Updated `extractAlbumDominantColor` to sample outer perimeter border pixels (outer 2–3 rows of canvas) first. This matches the true backdrop/frame of album covers (like the textured silver/gray metal background on Coldplay's *Mylo Xyloto / Paradise*) rather than sampling inner cutouts or focal subjects.

## [v3.6.1] - 2026-07-31
- **Hybrid Vibrant Color Preference Algorithm**: Updated `extractAlbumDominantColor` to prioritize non-neutral vibrant color pixels (`sat > 18`) over dull background grays when vibrant pixels are present, while retaining clean monochromatic gray/black/white fallback for black & white album art.

## [v3.6.0] - 2026-07-31
- **Skin Tone Color Avoidance (`isSkinTone`)**: Built human skin tone range detection (`R > G > B` with warm hue bounds `0° – 48°`). Filters out artist face/skin pixels in central album cover pixels so portrait artwork does not bleed skin colors onto UI buttons or card glows.
- **Subtle Playing-Only Glass Sheen Wipes**: Reduced glass wipe sheen opacity to `0.08` and restricted animations exclusively to media cards actively playing music.
- **Staggered Independent Wipe Scheduler (`scheduleNextGlassSweep`)**: Each active player triggers glass wipes at independent randomized 20–45s intervals, preventing simultaneous wipes across multiple media cards.

## [v3.5.1] - 2026-07-31
- **Instant Volume Slider Color Update**: Updated extraction callback to immediately query and tint `.media-large-vol-fill` elements (`rgba(r,g,b,0.35)`).
- **Monochromatic & Grayscale Album Color Support**: Removed artificial saturation/brightness filters from canvas sampler to allow true gray, white, dark, and monochromatic album artwork color matching.
- **iOS 9 WebKit Parse Error Fix**: Converted block-scoped function declarations inside `if/else` statements to ES5 function expressions (`var hue2rgb = function(...)`), resolving fatal parse errors on iOS 9.3.5 Safari.

## [v3.5.0] - 2026-07-31
- **HTML5 Canvas Pixel Color Extraction**: Built `extractAlbumDominantColor()` to sample 20x20 canvas thumbnails of album cover artwork, calculating the true visual dominant color.
- **Default Slate Blue Highlight Palette (`rgb(65, 122, 160)`)**: Set `rgb(65, 122, 160)` as the default accent highlight for buttons, progress fill, speaker badges, and volume sliders.
- **Smooth Asynchronous DOM Accent Update**: Automatically updates card glow, play pill, progress bar, and volume sliders as soon as album images finish loading.

## [v3.4.0] - 2026-07-31
- **Apple Glass Sweep Shimmer Animation**: Added `.media-card-glass-sweep` keyframe animation for a subtle diagonal light sweep across album cover backgrounds.
- **Dynamic UI Accent Tinting**: Tinted play/pause pill buttons, progress bars, speaker badges, equalizer wave bars, and volume sliders to match album art colors.
- **Outer Card Ambient Glow**: Added dynamic outer glow shadows to media cards (`box-shadow: 0 10px 32px rgba(0,0,0,0.42), 0 0 28px rgba(r,g,b,0.32)`).

## [v3.3.0] - 2026-07-31
- **Real-Time Optimistic Progress Bar Ticker**: Integrated `startMediaTickers()` to advance media playback progress bars and timestamps live second-by-second (`01:24`) without waiting for Home Assistant WS updates.
- **2-Line Song Title Clamping**: Applied `-webkit-line-clamp: 2` with `text-overflow: ellipsis` on `.media-card-track-title`.
- **Borderless Compact Controls Container**: Removed enclosing dark/frosted box around media controls (`background: transparent; border: none; box-shadow: none`), floating buttons directly over card artwork.
- **Compact Bottom Vertical Spacing**: Reduced margins between title, progress bar, play controls, and volume sliders to `4px – 6px`.

## [v3.2.3] - 2026-07-31
- **`ipaddashboard` Labeled Media Player Inclusion**: Added `allowedEntityIds` checking and space/underscore normalization (`.replace(/[\s_]/g, "")`) to `isIpadDashboardMedia()`.

## [v3.2.2] - 2026-07-31
- **iOS 9 WebKit Height Collapse Fix**: Updated `.media-card-content` to use absolute stretching (`position: absolute; top:0; left:0; right:0; bottom:0`) so flex containers do not collapse height on older iOS 9 Safari.
- **3-Card Side-by-Side Grid Layout**: Configured `.media-page-card` flex sizing (`-webkit-flex: 1 1 calc(33.333% - 11px); min-width: 270px`) to display up to 3 media players side-by-side in a single row.

## [v3.2.0] - 2026-07-31
- **Media Player Tab Redesign**: Rebuilt Media Player tab with full-card album cover background, top-right equalizer audio wave animation, and bottom frosted glass control bar.
- **Theme-Adaptive Album Cover Bottom Fade**: Fades album cover to solid pitch black (`#0d1117`) in dark mode or solid pure white (`#ffffff`) in light mode.

## [v3.1.1] - 2026-07-31
- **Sparse Home Layout Flickering Fix**: Replaced pixel measurement (`offsetTop`) with immutable grid-span row math (`Math.ceil(rowUnits / 2)`) to eliminate font size flickering on home screen layout updates.

## [v3.0.0] - 2026-07-31
- **Dynamic Sparse Home Screen Layout**: Automatically scales welcome header to `68px` bold and displays a giant digital clock at the bottom of the home screen when $\le 4$ card rows are present.

## [v2.99.0] - 2026-07-31
- **10-Second Siren Tone & Harsh Full-Screen Strobe**: Updated Lightning Safety popup with a 10-second warbling siren tone (synthesized via Web Audio API) and harsh red/black full-screen flashing strobe.

## [v2.96.0] - 2026-07-31
- **Tactile Apple Click Sound Effect**: Integrated Web Audio API frequency sweep (`1400Hz → 300Hz` in 12ms) for audible button press feedback with a toggle button in the footer.

## [v2.95.0] - 2026-07-31
- **Apple San Francisco System Font Stack**: Standardized typography across all views using Apple native San Francisco fonts (`-apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "SF UI Display", "SF UI Text"`).

## [v2.93.0] - 2026-07-31
- **iPad Dark Mode Status Bar Matching**: Dynamically updated `html.dark-theme` background to match dark site theme (`#0d1117`), fixing white status bar gaps on iPad displays.

## [v2.91.0] - 2026-07-31
- **Chemical Alert Red Button**: Added deep red chemical alert button (`#ff3b30`) with white beaker icon on Home screen for out-of-range pool/hot tub chemical readings.

## [v2.63.0] - 2026-07-21
- **Mobile Header Optimization**: Hidden date and time display (`.header-datetime`) under `@media (max-width: 600px)` and enlarged the main welcome message font size to `28px` bold (`850` weight).

## [v2.62.0] - 2026-07-21
- **Mobile Temperature Readout Vertical Stacking**: Configured `.thermostat-readout-box` under `@media (max-width: 600px)` to stack current and set target temperatures vertically (`flex-direction: column; align-items: center`), ensuring all Home page heater card controls fit comfortably on mobile screens.
- **Controls Grid Track & Overflow Overhaul**: Capped all grid column tracks with `minmax(0, 1fr)` and set `overflow: hidden; min-width: 0;` on `.controls-grid .tile` to guarantee cards never expand beyond their assigned columns or cut off on the right edge of the screen.

## [v2.61.0] - 2026-07-21
- **Half-Width (2-Across) Card Layout Restoration**: Removed 2-column full-width override to restore the 4-column grid layout where climate, water heater, and 2x2 cards span 2 columns (**exactly half width = 2 cards per row** on iPad).
- **Proportional Control Button Scaling**: Scaled heater card control buttons to `48px` height with `18px` font size and `26px` steppers to ensure half-width cards fit side-by-side with zero overflow or right-edge clipping.

## [v2.60.0] - 2026-07-21
- **Controls Grid Width & Overflow Elimination**: Set `@media (max-width: 900px)` for iPad Mini & tablet viewports to use a 2-column grid (`repeat(2, 1fr)`), expanding climate and water heater cards (`card-2x1`) to full grid width (`span 2`).
- **Flex Wrap Mode Selector Buttons**: Enabled flex wrap (`flex-wrap: wrap; gap: 4px`) and compact sizing (`padding: 6px 10px; font-size: 11px`) on `.mode-selector` buttons so climate and thermostat mode rows never force cards past the right display boundary.

## [v2.59.0] - 2026-07-21
- **Controls Grid Right-Edge Clipping Fix**: Reset legacy `margin: 5px` and flex properties on `.controls-grid .tile` to `margin: 0 !important; width: 100% !important; flex: none !important;` so CSS Grid `gap: 12px` sizes cards precisely to fit inside the viewport without clipping off the right edge.

## [v2.58.0] - 2026-07-21
- **Removed Internal Floor Scrolling**: Set `overflow: visible`, `height: auto`, and `max-height: none` on `.controls-grid` and section containers. Converted `.floor-nav-bar` to flex wrap (`flex-wrap: wrap; overflow: visible`) so all floor controls flow naturally in a continuous grid without inner sub-scrolling.

## [v2.57.0] - 2026-07-21
- **Room Name Subtext on Controls Section Cards**: Added room name subtext (`tile-room`) to all feature cards on the Controls page when a room is assigned (leaves blank if unassigned).
- **Strict Dimmer Control Filtering**: Restricted `-` / `+` brightness stepper controls exclusively to lights that explicitly support dimming / brightness functions.
- **Horizontal Overflow Prevention on Controls Grid**: Enforced `width: 100%`, `max-width: 100%`, and `overflow-x: hidden` across `.controls-grid` and section containers so tiles fit the exact width of the display without horizontal scrolling.

## [v2.56.0] - 2026-07-21
- **Light Card 2x2 vs 1x1 Classification Fix**: Fixed light classification so standard lights and dimmers without effects or RGB color controls stay as compact **1x1 cards (`card-1x1`)**. Only lights with effects or RGB color controls expand to 2x2.
- **Mobile Home Page 1 Card Per Row**: Enforced 1 card per row (`100%` width) for all Home page tiles under `@media (max-width: 600px)`.
- **Responsive Mobile Text & Control Scaling**: Dynamically scaled down title font sizes (`22px`) and temperature control buttons (`48px` height, `18px` font) on mobile screens so all controls fit cleanly without overflowing.

## [v2.55.0] - 2026-07-21
- **Expanded Preset Temperature Buttons**: Expanded horizontal padding (`padding: 0 18px`), font size (`22px` bold), and minimum width (`64px`) on preset buttons (`68°`, `78°`, `100°`) so temperature numbers fit cleanly without clipping.
- **Side-by-Side Temperature Display**: Replaced vertical column layout with a side-by-side row (`align-items: baseline`) displaying Current Temp in `34px` bold and Set Target Temp in `22px` bold (`Set 78°`).

## [v2.54.0] - 2026-07-21
- **Enlarged Home Page Heater Controls**: Scaled up font sizes and button sizes for temperature controls on Home page heater cards (stepper buttons `56px x 56px`, `32px` font; preset buttons `56px` height, `20px` font; current temp `28px` bold; target temp `16px`). Card container outer size remains unchanged.
- **Card-Tap Heater Power Toggle**: Removed the dedicated power toggle button inside the control row; pressing anywhere on the heater card toggles heater power on/off (with `event.stopPropagation()` on stepper/preset buttons).
- **Conditional Outline vs Fill Highlight**: Displays a soft red outline (`active-red-outline`) whenever the heater is ON, and fills the inside background (`active-red`) ONLY when the set target temperature is greater than the current temperature (`set > current`, actively heating).

## [v2.53.0] - 2026-07-21
- **Controls Page 4-Column CSS Grid Overhaul**: Overhauled Controls page layout into a clean, standardized 4-column CSS Grid (`repeat(4, 1fr)`) with `grid-auto-flow: dense` for automated gap packing.
- **Standardized Card Unit Dimensions**:
  - **1x1 Cards (`card-1x1`)**: Standard 1-tap lights, switches, fans, locks, covers, input_booleans (1 column wide, 1 row tall).
  - **2x1 Cards (`card-2x1`)**: Climate thermostats and water heaters (2 columns wide, 1 row tall).
  - **2x2 Cards (`card-2x2`)**: Lights with effects, dimming, or color controls (2 columns wide, 2 rows tall).
- **Media Player Exemption**: Media player cards retain their specialized layout.

## [v2.52.0] - 2026-07-21
- **Smart Heater Highlight Logic**: Disabled red active highlight on climate and water heater cards whenever the current temperature is higher than the set target temperature (`current > target`).
- **Target Equal/Active Heating Refinement**: Maintained soft red outline when `current == target` and full red highlight tint when actively heating (`current < target`).

## [v2.49.0] - 2026-07-21
- **Album Art Live Refresh**: Appended dynamic cache-busting version parameter (`_v=` + track title hash + timestamp) to album art URLs so Safari on iOS 9 instantly re-renders new artwork on track changes.
- **`ipadhomescreen` Tag Filtering for Home Tab**: Added `homeEntityIds` parsing and `isHomeScreenEntity()` helper. Only devices tagged with `ipadhomescreen` appear on the Home tab.
- **`ipaddashboard` Tag Filtering for Controls Menu**: Devices with `ipaddashboard` tag continue to appear on the Controls menu screen.

## [v2.48.0] - 2026-07-21
- **Clean 100° Button Label**: Replaced the fire emoji with clean text `100°` on the Hot Tub quick heat preset button.
- **Muted Orange Heater Controls**: Unified all heater power buttons and preset buttons to a cohesive, muted orange palette (`rgba(255, 159, 10, ...)`).
- **Matching Icon Highlight Colors**: Tile SVG icons now dynamically match their card's background highlight color (Yellow for lights, Orange/Red for heat, Blue for media players, Green for other features).
- **Blue Playing Media Highlight**: Media cards highlight in **blue** (`active-blue`) when playing.
- **Track Title Big Text Formatting**: Media cards format Track Title as big text (`26px` bold) and speaker names as subtext (`13px`), fitting multiple speaker names without exceeding the standard 84px card height.

## [v2.47.0] - 2026-07-21
- **Green Active Highlight Color**: Configured green tint (`active-green`, `background: rgba(52, 199, 89, 0.14); border: 2px solid #34c759;`) for all non-light and non-heat feature cards.
- **Playing-Only Media Card Highlight**: Media cards only display active green highlight when actively playing (`state === 'playing'`); unhighlighted when paused/idle.
- **1-Tap Media Play/Pause Anywhere & Art Overlay**: Tapping anywhere on a media card triggers Play/Pause; moved Play/Pause SVG icon directly over top of enlarged album art (`64px x 64px`).
- **Enlarged Skip Button**: Enlarged Next track skip button to `48px x 48px` with a `24px` icon for far-away visibility.
- **Heater Power Button & Quick Preset Buttons**: Added dedicated On/Off power toggle buttons and quick heat buttons to heater cards (Pool: **68°** & **78°**; Hot Tub: **68°** & **100°** with Fire SVG).

## [v2.46.0] - 2026-07-21
- **Full Width Climate & Heater Cards**: Expanded climate thermostats and water heater cards on the Home page to full width (`flex: 0 0 calc(100% - 12px)`), providing ample room for full 34px titles ("Upstairs", "Hot Tub", "Pool", "Main AC") alongside thermostat +/- buttons and boost fire controls without wrapping or truncating.
- **Larger Centered Header Navigation Icon**: Increased header button to `48px x 48px` and SVG icon to `26px x 26px`, adding `style="display:block; margin:auto;"` for perfect flex alignment on iOS 9 Safari.

## [v2.45.0] - 2026-07-21
- **Stripped Filler Words ("Overheads", "Thermostat")**: Cleaned card names on the Home page (e.g. "Theater Overheads" -> "Theater", "Main AC Thermostat" -> "Main AC").
- **Uniform 34px Bold Title Font**: Enforced a uniform **`34px` bold** font (`font-weight: 850`, `line-height: 1.05`, `letter-spacing: -0.8px`) across all card titles for complete visual continuity.
- **Dynamic Card Height Allowance**: Configured `.tile.overview-big-tile` to `height: auto; min-height: 84px; padding: 6px 12px;` so cards with extra controls or 2-line titles dynamically expand height to fit the 34px font without truncation.

## [v2.44.0] - 2026-07-21
- **Large 84px Card Size Maintained**: Retained the large 84px card height (`min-height: 84px`, `height: 84px`, `padding: 0 16px`, `border-radius: 20px`).
- **100% Vertical Card Fill**: Scaled title text to **`36px` bold** (`font-weight: 850`, `line-height: 1.05`), tile icons to `60px x 60px`, and thermostat buttons to `54px x 54px` with 0px top/bottom padding so the text and icon stretch across 100% of the card's height from top edge to bottom edge.

## [v2.43.0] - 2026-07-21
- **Eliminated Vertical Card Whitespace**: Compacted card height to a sleek 60px (`height: 60px`, `padding: 4px 14px`, `border-radius: 16px`, icon `38px`, buttons `36px`), filling 100% of the vertical space inside the card.
- **Uniform 22px Title Font Across All Cards**: Set a consistent `22px` bold font (`font-weight: 800`, `line-height: 1.15`) for all card titles, eliminating font size variance and restoring visual continuity across the dashboard.

## [v2.42.0] - 2026-07-21
- **Dynamic 2x2 Grid Tile Packing**: Grouped single-height 1-tap control cards into 2-card vertical stacks (`singleStacks`) in the Controls menu.
- **Eliminated Excess Empty Space**: Unified the card grid so 4 single-height cards automatically stack 2-high and 2-wide next to double-height heater/thermostat cards, completely filling the grid layout.

## [v2.41.0] - 2026-07-21
- **Full 84px Card Fill (Zero Vertical Dead Space)**: Maintained full 84px card height (`min-height: 84px`, `height: 84px`, `padding: 2px 14px`, `border-radius: 20px`) while scaling icons to `58px x 58px`, title font to `34px` bold for 1-line titles, and control buttons to `52px x 52px`.
- **Top-to-Bottom Edge Stretch**: Scaled elements to stretch across 90%+ of the card height, filling the entire card from top edge to bottom edge.

## [v2.40.0] - 2026-07-21
- **Dynamic Header Navigation Button**: Header button displays a 3-line Hamburger SVG (`title="Menu"`) when on the Home screen (tapping goes to Controls), and a House SVG (`title="Home"`) when on Controls or Chemicals (tapping goes to Home).
- **Removed "Home" from Menu Bar**: Omitted the redundant "Home" button from the `.tab-bar` menu bar.
- **90% Height Card Fill**: Configured card height (`52px`, `padding: 3px 12px`, icon `36px`, controls `34px`) so the icon and text fill 90% of the card height with zero top/bottom dead space.

## [v2.39.0] - 2026-07-21
- **Streamlined House Navigation Handler**: Rewrote `toggleMainNavigation(event)` and `window.switchTab` for 100% reliable tab switching (Home -> Controls on Home tab; Controls/Chemicals -> Home on other tabs).
- **Zero-Dead-Space 84px Overview Cards**: Maintained 84px card height (`min-height: 84px`, `height: 84px`, `padding: 10px 16px`) with 44px icons, 42px controls, and 28px titles so the entire 84px height is filled with zero top/bottom dead space.
- **Clean Text Chemical Banner (No Emojis)**: Updated `#headerChemBanner` text to clean `"Chemical Alert"` or `"Check Chemicals"` without emojis.

## [v2.38.0] - 2026-07-21
- **Header Chemical Banner**: Added a subtle, non-cluttering "Check Chemicals" / "Chemical Alert" banner directly below the date & time in the top-right header area, removing individual card exclamation icons.
- **Sleek 60px Compact Card Height**: Reduced card height from 84px to 60px (`padding: 4px 14px`, `border-radius: 16px`, icon `34px`, buttons `34px`) to eliminate all top/bottom vertical dead space.
- **House Navigation Button Toggle**: Pressing the House button on the Home tab navigates to Controls; pressing it on Controls or Chemicals navigates back to Home.

## [v2.37.0] - 2026-07-21
- **Hidden Footer on Home Tab**: Configured `body.tab-home footer.app-footer { display: none !important; }` so the footer is hidden on the Home page.
- **Fixed Home Button Handler**: Corrected `switchTab` execution to call `renderApp()`, restoring full functionality to the House navigation button.

## [v2.36.0] - 2026-07-21
- **House SVG Navigation Icon**: Replaced top header 3-line hamburger menu icon with a clean House SVG icon (`title="Home / Menu"`).
- **Renamed Overview Tab to Home**: Updated all code references, HTML tab buttons, JavaScript functions (`renderHomeCard`, `renderHomeMediaPlayerCard`), CSS classes (`body.tab-home`), and inactivity prompts from "Overview" to "Home".

## [v2.35.0] - 2026-07-21
- **Equal Card Height & Padding Optimization**: Set uniform 84px height (`height: 84px`, `min-height: 84px`) and reduced internal padding (`padding: 8px 16px`) for Overview cards.
- **Dynamic 2-Line Font Scaling**: Scaled titles (`26px` for 1-line, `19px` for 2-lines) so 2-line cards fit within the same 84px height container as 1-line cards.
- **Red Exclamation Chemical Alert Icon**: Added a pulsing red exclamation SVG button (`[ ! ]`) to Pool and Hot Tub cards when a red chemical alert is present, navigating directly to the Chemicals tab.
- **Reinstated Active Tile Subtle Color Fills**: Re-enabled translucent background fills and colored border glows (`.active`, `.active-blue`, `.active-green`, `.active-red`) across Light and Dark themes.

## [v2.34.0] - 2026-07-21
- **No-Cutoff Header Welcome Message**: Removed `max-width: 33vw` and `text-overflow: ellipsis` from `#headerWelcome`, enabling natural 1-2 line text wrapping so long messages never get cut off.
- **Organic 3-Layer Flame SVG**: Deployed an organic 3-layer Flame SVG icon for the Hot Tub quick boost button.

## [v2.33.0] - 2026-07-21
- **Comprehensive README Documentation**: Created detailed `README.md` documenting all system features, architecture, setup steps, and security notices.
- **AI Generation & Human Review Disclaimer**: Documented that codebase was built with Gemini assistance and human-reviewed.
- **Security Warnings**: Highlighted environment restrictions (not for secure production environments) and hardcoded `HA_TOKEN` security risks.

## [v2.32.0] - 2026-07-21
- **Standard Classic Flame SVG Icon**: Replaced complex fire icon with a clean, standard classic Flame SVG icon.
- **Instant Optimistic Fire Button Switching (<1ms)**: Direct DOM element targeting (`btn_fire_<eid>`) updates Fire button background, border, and flame color instantly in <1ms when pressed.

## [v2.31.0] - 2026-07-21
- **Uniform 30px Card Title Font Scale**: Replaced length-based step scaling with a consistent, uniform `30px` bold title font across all Overview tiles (using natural 1-2 line wrapping for title length).

## [v2.30.0] - 2026-07-21
- **Fire Button State Toggle (100°F / 68°F)**: Fire button is muted gray when target != 100° (tapping sets target to 100°F) and glowing orange when target == 100° (tapping sets target to 68°F).
- **Detailed Multi-Flame SVG**: Replaced simple fire icon with a detailed multi-flame SVG icon.
- **Dynamic Header Text Scaling (1/3 Page Width)**: Header message and Date & Time scale dynamically up to 1/3 page width (`max-width: 33vw`, `clamp(18px, 3.2vw, 32px)`) so the display always feels 100% full.

## [v2.29.0] - 2026-07-21
- **100°F Hot Tub Quick-Boost Button**: Added a dedicated Fire icon SVG button (`setHotTubMaxTemp`) to the Hot Tub card that sets the target temperature to 100°F with 1 tap.

## [v2.28.0] - 2026-07-21
- **Whole-Number Temperatures**: Temperature adjustments step by 1° whole numbers, and all readouts/target temperatures are rounded to the nearest integer (`Math.round`).
- **34px Max Font Scale**: Card feature titles scale up to `34px` bold (`34px`/`28px`/`22px` dynamic scale), taking up to 95% of card height and 100% card length.

## [v2.27.0] - 2026-07-21
- **15-Minute Media Pause Auto-Hide**: Paused media players automatically hide from the Overview tab after 15 minutes of inactivity (`last_changed` > 15 mins).
- **Live Instant Set Temperature UI**: Pressing `+` or `-` updates set temperature text on screen instantly (<1ms).
- **28px Title Font Scale**: Increased card feature titles to `28px` bold (`28px`/`24px`/`20px` dynamic scale).
- **Removed Room Labels**: Removed room names from Overview cards to maximize title font size.

## [v2.26.0] - 2026-07-21
- **Instant DOM Set Temp Target**: Added `set_temp_<eid>` DOM node updates inside `adjustTemp` and `adjustWaterHeaterTemp`.

## [v2.25.0] - 2026-07-21
- **Global `msgId` Fix**: Promoted `globalMsgId` and `getNextMsgId()` to global script scope to eliminate `ReferenceError: msgId is not defined` crash.

## [v2.24.0] - 2026-07-21
- **Ground-Up Temperature Rewrite**: Rewrote `sendTemperatureCommand` to include `target: { entity_id }` and atomic `hvac_mode` payload.
- **300ms Micro-Debounce**: Fast 300ms micro-debounce for multi-tap adjustments.

## [v2.23.0] - 2026-07-21
- **Automatic HVAC Mode Switching**: Auto-switches HVAC mode to `heat` when target > actual temp, and `cool` when target < actual temp.
- **Hardened Water Heater Dispatch**: Dual WebSocket + HTTP REST API dispatch for `water_heater.hot_tub` and `water_heater.scott_pool`.

## [v2.22.0] - 2026-07-21
- **Safe `sendWsMessage`**: Defined safe WebSocket sender to prevent unhandled TypeErrors.

## [v2.21.0] - 2026-07-21
- **Big Speaker Names on Overview**: Displayed speaker/room names in big bold title (`21px`-`24px`) and track info in small text underneath.
- **Vertically Stacked Temperature Readouts**: Stacked actual temp (`73°`) over set temp (`Set 73°`) to save ~50px horizontal space.
- **Reinstated Top Tab Navigation**: Restored top `.tab-bar` menu bar on Controls and Chemicals pages.

## [v2.20.0] - 2026-07-21
- **Removed Duplicate Overview Header**: Removed second header row and blue `⚙️ Controls & Menu` button.

## [v2.19.0] - 2026-07-21
- **Unified Persistent Header**: Persistent top header displaying `Welcome Home` on top left, and Date & Time + 3-line SVG hamburger menu button on top right.

## [v2.18.0] - 2026-07-21
- **3-Line Hamburger Menu Button**: Placed 3-line SVG hamburger button in top right for 1-tap navigation to Controls tab.

## [v2.17.0] - 2026-07-21
- **25px Font Scale & Text Wrapping**: Enabled `white-space: normal; word-break: break-word;` so long card titles wrap naturally over 1-2 lines without `...` truncation.

## [v2.16.0] - 2026-07-21
- **Minimal Header Alignment**: Inline header displaying `Welcome Home` on left, Date & Time on right.

## [v2.15.0] - 2026-07-21
- **Inline Header Date & Time**: Moved Date & Time to the right edge of header.

## [v2.14.0] - 2026-07-21
- **Hidden Tab Bar on Overview**: Hidden top tab bar on Overview tab for ultra-minimal layout.

## [v2.13.0] - 2026-07-21
- **iOS Safari Full-Screen Web App Tags**: Added `apple-mobile-web-app-capable`, `viewport-fit=cover`, and `black-translucent` status bar style.
- **Debounced Thermostat Controller**: 500ms debouncing and 3-attempt retries for temperature adjustments.

## [v2.12.0] - 2026-07-21
- **Overview Media Player Cards**: Added Play/Pause & Next Track (Skip) media cards.
- **Dark Mode Contrast**: Improved media button contrast and formatted set/actual temps.

## [v2.11.0] - 2026-07-21
- **Overview Single Menu Button**: Restored top header with single menu button on Overview page.

## [v2.10.0] - 2026-07-21
- **Overview Thermostat Adjusters ([ - ] / [ + ])**: Added inline temperature adjustment buttons directly on Overview cards.
- **Outline-Only Highlight**: Added `.active-red-outline` CSS class when target equals actual temperature.

## [v2.9.0] - 2026-07-21
- **Dark Mode Contrast**: Fixed text contrast (`#1c1c1e` -> bright white).
- **Card Height Scale**: Set card min-height to 88px for display height fitting.
- **Omitted "General Controls" Label**: Omitted room headers when no specific room is matched.

## [v2.8.0] - 2026-07-21
- **Theme: Simplicity**: Set big Overview controls to 50% width with inline Room & Feature names (18.5px bold). Hidden header/footer on Overview and removed section headers.

## [v2.7.0] - 2026-07-21
- **Area Grouping**: Grouped Overview controls by Room (Area), increased overall text sizes, and reduced min-height to eliminate whitespace.

## [v2.6.0] - 2026-07-21
- **Clean Overview**: Hidden idle media players from Overview tab.
- **Controls Tuning**: Deployed a clearer cascading waterfall SVG icon and increased font size & padding for Controls cards.

## [v2.5.0] - 2026-07-21
- **Tab Layouts**: Reordered floors to Outside, Basement, Upstairs, Main Floor, and fixed target temperature text color contrast in dark theme.

## [v2.4.0] - 2026-07-20
- **Overhaul Layout**: Built side-by-side split grid layouts, customized SVG indicators (waterfall, bubbles, fire, hottub), and centered top floor navigation.

## [v2.3.0] - 2026-07-20
- **Climate density**: Built high-density 25% climate tiles and Pool/Hot Tub presets.
- **Visuals**: Added set/current temperature match highlights and implemented a 15-minute inactivity idle reset.

## [v2.2.0] - 2026-07-20
- **Streamlining CSS**: Deleted unused toggle-switch and thermostat CSS.
- **Layout overhaul**: Implemented dynamic floors, compact card layouts, and pulsing heater animations.

## [v2.1.1] - 2026-07-20
- **Hotfix loading**: Added a fallback timeout to resolve loading screen hangs, and fixed Controls tab crash during dynamic floor key resolution.

## [v2.1.0] - 2026-07-20
- **Floor registries**: Integrated dynamic floor and area fetching via Home Assistant WebSockets, condensed temperature control UI, and removed redundant switches.

## [v2.0.1] - 2026-07-20
- **Safari 9 & WebSocket Fallback**: Integrated 2.5s WebSocket fallback registry timers, safe label filtering guards, and try-catch blocks to prevent blank screen hangs.

## [v2.0.0] - 2026-07-20
- **Landmark Visual Redesign**: Introduced dynamic top header with custom welcome message and live clock, minimalist SVG vector icons (no emojis), and muted outline-only active card styling.
- **Sorting**: Grouped Overview tab into Media, Lights, Features, and Master Switches, and Controls tab by Floor.

## [v1.33.0] - 2026-07-20
- **RGB Preset Picker**: Added one-tap preset buttons (Bright White, Warm White, Red, Blue) directly on color-enabled light cards when turned ON.

## [v1.32.0] - 2026-07-20
- **Muted Badges**: Muted navigation alert badges for enhanced tab contrast, and restructured Overview tab.

## [v1.31.0] - 2026-07-20
- **Navigation alerts**: Removed top red alert banner in favor of a navigation badge priority gate (Red Chemical Alert > Yellow Check Chemicals).

## [v1.30.0] - 2026-07-20
- **Confirm All logs**: Added a single-tap "Confirm All" button to batch-confirm all entered chemical values.
- **Logs Expiry**: Auto-expiry timers to clear unconfirmed chemical selections after 3 minutes.

## [v1.29.0] - 2026-07-19
- **Removed Music Tab**: Completely removed the Music tab, player selector, and associated Music Assistant code.

## [v1.28.15] - 2026-07-19
- **Music Assistant Tab**: Added Music Assistant integration tab with speaker selector, playlist browser, and shuffle playback actions.

## [v1.27.15] - 2026-07-19
- **Logs UX**: Positioned current values inline with chemical titles, increased font sizes, and enlarged Confirm buttons.

## [v1.26.15] - 2026-07-19
- **Presets**: Centered quick log buttons, enforced single row layouts, and disabled Confirm buttons until a value is selected.

## [v1.25.15] - 2026-07-19
- **Chlorine presets**: Excluded chlorine names from isCalcium checks to prevent Free Chlorine from inheriting wrong calcium fallback presets.

## [v1.24.15] - 2026-07-19
- **Safari 9 layout**: Replaced flexbox `gap` spacing with margins for Safari 9 compatibility in stacked media cards and confirmation overlays.

## [v1.23.15] - 2026-07-19
- **Button bars**: Overhauled volume controllers to use rounded rectangle button bars with 6.25% increments and progress fills.

## [v1.22.15] - 2026-07-19
- **Fix Controls Tab Crash**: Restored the `isGroupIdle` helper to resolve Controls page loading ReferenceErrors.

## [v1.22.14] - 2026-07-19
- **Sliderless UX**: Removed all sliders from the dashboard (replaced volume sliders with Down/Mute/Up buttons, brightness with +/- adjustments, and chemical inputs with presets + keypad).

## [v1.21.14] - 2026-07-19
- **Keypad flexbox**: Wrapped Play SVG contents in flex div container to fix button alignment bugs on iPad.

## [v1.20.14] - 2026-07-19
- **Stacked cards**: Columns of up to 3 cards for idle media players to prevent layout heights mismatch on Controls page.

## [v1.19.14] - 2026-07-19
- **Compact players**: Compacted Controls page idle media players to 1/3 height and hid redundant speaker names in single-speaker groups.

## [v1.18.14] - 2026-07-09
- **Track logging**: Tracked and logged skipped grouped speakers and details for robust transitions.

## [v1.17.14] - 2026-07-09
- **Footer details**: Displayed dashboard version tags directly in footer.

## [v1.0.0] - 2026-07-07
- **Initial release of the iPad Mini 1st Gen Home Assistant Web Dashboard (iOS 9 Safari compatible, WebSocket + HTTP REST API fallback).**

## [v0.9.0] - 2026-07-06
- **Speaker Sticky Transitions**: Deployed a 10-second sticky period for grouped speaker cards to prevent visual splitting during transition states.
- **Optimistic Skipped Track Details**: Added transition logging of skipped track metadata and album art to pick up new tracks smoothly when speakers are in flux.

## [v0.8.0] - 2026-07-05
- **Auto-Idle Timeout Reset**: Implemented a 15-minute inactivity automated tab switching timer, resetting active view back to the Overview tab.
- **Overview Check Chemicals Warning Card**: Replaced full-screen chemical splash screens with a yellow reminder card on the Overview tab.

## [v0.7.0] - 2026-07-04
- **Agenda Calendar Tab (Beta)**: Deployed a Calendar tab with active agenda views, subsequently reverted to optimize dashboard performance.

## [v0.6.0] - 2026-07-03
- **Manual Theme overrides**: Introduced manual dark/light theme overrides with persistent state caching.
- **Safari 9 CSS Adjustments**: Handled iOS 9 strict mode scopes and CSS fallbacks.

## [v0.5.0] - 2026-07-02
- **Virtual Keypad Entry**: Built a virtual numeric keypad overlay to replace chemical sliders, enabling tap-to-input logging.
- **Automated Hot Tub Fixes**: Added an automated 15-second hot tub chlorine boost timer switch.

## [v0.4.0] - 2026-07-01
- **Superchlorination Calculator**: Integrated duration formulas matching salt cell output rates to Combined Chlorine burn targets.
- **ORP/pH Alert Cross-Suppression**: Cross-correlated sensor readings to prevent conflicting alert alerts (e.g. pH vs ORP dosing).

## [v0.3.0] - 2026-06-30
- **Chemical Recommendation Engine**: Programmed dosage conversions targeting cups, tbsp, and 40lb salt bags for Pool (20,000 gal) and Hot Tub (600 gal).

## [v0.2.0] - 2026-06-25
- **Speaker Group Manager**: Integrated custom virtual groups to bypass hardware speaker join limitations, resolving HTTP 500 join errors.
- **Media Grouping**: Grouped speaker cards dynamically by current track / media status.

## [v0.1.0] - 2026-06-20
- **Core Boilerplate**: Deployed foundational single-page dashboard architecture.
- **Long-Lived Access Token Auth**: Programmed Home Assistant REST and WebSocket authentication.
- **High-Performance UI Grid**: Programmed CSS grid layouts optimized for 1st generation iPad Mini (iOS 9 Safari).
