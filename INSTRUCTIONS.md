# Setup & Configuration Guide (`INSTRUCTIONS.md`)

This guide explains how to configure Home Assistant and customize the dashboard configuration variables to deploy this iPad Web Dashboard on your own Home Assistant instance.

---

## 1. Required Configuration Variables (`index.html`)

Open `index.html` in an editor and adjust the global **CONFIG** variables located around line 1824:

```javascript
// ======== CONFIG ========
var HA_URL = "http://YOUR_HA_IP:8123";  // Your Home Assistant Server IP & Port (e.g. http://192.168.1.100:8123)
var HA_TOKEN = "YOUR_LONG_LIVED_ACCESS_TOKEN_HERE"; // Long-Lived Access Token from Home Assistant
var STORAGE_KEY = "ha_controls_config";  // LocalStorage key for caching options
var REFRESH_INTERVAL = 15000;            // Full-state reconcile interval in ms (15 seconds; skipped entirely when nothing changed)
var CAMERA_ENTITY = "camera.album_slideshow_kitchen_icloud_album"; // Lock screen camera entity ("" = disabled)
var CAMERA_REFRESH_MS = 5 * 60 * 1000;   // How often the lock screen camera refreshes (default 5 minutes)

// ======== PARTY PAGE ========
var PARTY_MODE_ENTITY = "input_select.house_mode";                 // Entity that activates the Party page when set to "Party"
var PARTY_QR_URL = "http://192.168.40.89:8095/#/guest";            // URL encoded in the Party page QR code (guest music queue)
var PARTY_MEDIA_ENTITY = "media_player.family_room_apple_tv";      // Media player whose group is shown on the Party page
var PARTY_HIDDEN_ENTITIES = ["theater", "game_room", "kitchen"];   // Substring patterns: matching entities never appear on the Party page (still on Home)
var PARTY_POOL_LIGHTS_ENTITY = "light.scott_pool_lights";          // Pool-lights shortcut card on the Party page (half-column, pool + bulb icon)
var PARTY_SPA_LIGHT_ENTITY = "light.spa_light";                    // Spa-light shortcut card on the Party page (half-column, hot-tub + bulb icon)
var PARTY_EXIT_MODE = "Day";                                       // house_mode option the 🎉 header toggle switches to when ending a party
```

### Party Page
When the `PARTY_MODE_ENTITY` (default `input_select.house_mode`) is set to `Party`, the dashboard automatically switches to a dedicated **Party page**: home-screen controls stacked in one column on the left, and the QR code plus a compact media card for the `PARTY_MEDIA_ENTITY` media group on the right. The card uses the same renderer as the Media tab's multi-player layout (no album-cover square, transport + volume controls only). Entities whose id contains any `PARTY_HIDDEN_ENTITIES` substring (e.g. every kitchen light) are excluded from the left column (they remain on the Home tab). The top of the left column is a pair of half-width shortcut cards for `PARTY_POOL_LIGHTS_ENTITY` and `PARTY_SPA_LIGHT_ENTITY` — big high-contrast pool-and-bulb / hot-tub-and-bulb icons that toggle their light on tap and highlight yellow while on; anything named "outdoor" (plus these two entities) is dropped from the generic stack below so nothing renders twice. The layout is tuned so the right column — QR panel plus media card — fits a 1024x768 landscape screen with no scrolling. Parties follow the **normal theme rules** again (night or a manual dark override makes the Party page dark — every party-page surface has a dark variant), but **self-dimming stays suppressed** (the dim overlay and night dimming never engage mid-party), and the automatic return-home timers are pinned so the page stays put; a floating Home button in the top-right corner remains as a manual escape hatch, and a one-line strip showing the welcome message (`input_text.dashboard_welcome_message`) sits at the top of the left column in standard bold body text at 56px — it never wraps, overly long messages just clip with an ellipsis.

You can also start or end a party by hand: the All Controls header has a 🎉 button next to Settings that toggles `PARTY_MODE_ENTITY` between `Party` and `PARTY_EXIT_MODE`. The Home screen also has a "🎉 Party Mode" button next to "All Controls" that performs the same toggle — it highlights pink when a party is active. And if you navigate to Home mid-party, a floating "🎉 Back to Party" pill appears at the bottom of the Home screen (the footer nav bar is hidden there).

**Nothing auto-switches pages during a party**: tagged media players starting playback, hot tub chemical red alerts and all return-home timers are suppressed until the party ends (a still-red chemical alert fires as soon as it's over). The Settings/Reload/Home buttons in the All Controls and Chemical Status headers stay visible too — their icon color follows `isDashboardDarkTheme()` like every other theme decision. Note: actual hardware brightness is controlled from iPad Settings or Guided Access — a web page cannot change it. The dashboard's own night/away **lock screen (the camera-feed splash overlay) never shows during a party** — not for alarm/away state, not at night, and not via the manual "Lock Now" button (which shows a "disabled during a Party" toast instead). Normal lock behavior resumes automatically when house mode leaves `Party`. Leaving `Party` returns the dashboard to Home automatically. The QR code points guests to whatever URL you place in `PARTY_QR_URL` (e.g. your local queue service), and the media card stays visible for the whole party even when nothing is playing — its built-in Play pill starts the music.

### Media Refresh Behavior
Media data updates through three cooperating layers, so track changes appear as fast as Home Assistant reports them without redundant UI rebuilds:
- **WebSocket (primary)**: `state_changed` events are diffed per media player (`mediaDisplayDiff`: state, title, artist, album, artwork, volume). Any display change re-renders the Media page instantly; Home / All Controls tabs update in place and only fully re-render on play/pause flips.
- **Unified REST poll (backup, 3s)**: `pollStatesHA` fetches `/api/states`, diff-applies media players and silently refreshes all other entity caches.
- **Track-boundary fast poll**: when the progress ticker sees a playing player reach 0:00, touch the end of its duration, or report a new duration, it fires an immediate one-shot poll (throttled to once per 2s) so new track metadata lands right as the progress bar resets.
- **Freshness guards**: every REST snapshot is checked against the state already applied by the WebSocket (`isNewerState`, comparing `last_updated`) before it may overwrite caches — so a slow poll captured before a track change can't repaint the previous song afterwards. The same newer-wins rule protects the 15s full refresh.
- **Artwork loading**: the Media page, Party page and dominant-color prewarming all request artwork via the raw `entity_picture` URL Home Assistant reports (signed CDN/proxy URLs break if extra query parameters are added). Only the Home tab's small thumbnails append a `_v=<title_artist>` cache-buster to keep covers fresh when the proxy URL itself doesn't change.
- **Continuity**: the previous album cover and its dominant color stay on screen until the new artwork has fully downloaded, so track changes never flash blank cards or generic tint colors.

### Lock Screen Camera Feed
When `CAMERA_ENTITY` is set to a camera entity ID (e.g. `camera.album_slideshow_kitchen_icloud_album`), the night / away lock screen shows a camera snapshot as its background instead of a plain black screen. The snapshot is loaded via a plain `url()` background-image from Home Assistant's `camera_proxy` endpoint (`/api/camera_proxy/<entity>?token=<access_token>`). The camera's rotating access token (published in the entity's state attributes) is used instead of the Bearer token, so iOS 9 Safari decodes the JPEG natively — no XHR or base64 conversion needed. The image refreshes automatically every `CAMERA_REFRESH_MS` (5 minutes by default) while the lock screen is visible — it does not update while unlocked to save bandwidth and iPad Mini battery. Each fetch includes a cache-busting timestamp so slideshow cameras (like Album Slideshow) always serve the latest rendered frame.

You can also configure this from the dashboard without editing the file: open **Settings → Display → Lock Screen Camera**, enter the entity ID and refresh interval in minutes, then tap **Save Camera**.

### How to generate a Long-Lived Access Token in Home Assistant:
1. Log into your Home Assistant web interface.
2. Click your **Profile** icon at the bottom of the left sidebar.
3. Scroll down to the **Long-Lived Access Tokens** section.
4. Click **Create Token**, name it `iPad Dashboard`, and copy the token string into `HA_TOKEN`.

The token lives only in the source file (or a previously saved `localStorage` value) — the on-screen **Connection Settings** card is intentionally IP-only, so a server address change after a reboot can be fixed from the iPad without exposing or re-entering the token.

---

## 2. Required Home Assistant Helpers

The dashboard relies on specific Home Assistant **Helpers** (`input_boolean` and `input_text`) for dynamic welcome messages and pool chemical tracking state gates. Create these in **Settings -> Devices & Services -> Helpers**:

| Helper Type | Entity ID | Friendly Name | Description |
| :--- | :--- | :--- | :--- |
| **Text** (`input_text`) | `input_text.dashboard_welcome_message` | `Dashboard Welcome Message` | Sets the custom welcome text shown at the top-left of the header bar (e.g., "Welcome Home, Family!"). Defaults to "Welcome Home" if unavailable. |
| **Toggle** (`input_boolean`) | `input_boolean.chemicals_checked_today` | `Chemicals Checked Today` | Tracks whether chemical levels have been logged today. Displays a yellow warning banner when turned `off`. |
| **Toggle** (`input_boolean`) | `input_boolean.chlorine_logged_since_changing` | `Chlorine Logged Since Changing` | Safety gate used by pool/spa chemical dosage recommendations to prevent accidental chlorine overdosing. |
| **Toggle** (`input_boolean`) | `input_boolean.ryan_is_home` | `Ryan Is Home` | Occupant presence toggle for Ryan. Renders iOS System Blue (`#007AFF`/`#0A84FF`) background tint when no media is playing. |
| **Toggle** (`input_boolean`) | `input_boolean.mason_is_home` | `Mason Is Home` | Occupant presence toggle for Mason. Renders iOS System Green (`#34C759`/`#30D158`) background tint when no media is playing. |
| **Toggle** (`input_boolean`) | `input_boolean.chad_is_home` | `Chad Is Home` | Occupant presence toggle for Chad. Renders iOS System Orange (`#FF9500`/`#FF9F0A`) background tint when no media is playing. |
| **Toggle** (`input_boolean`) | `input_boolean.keira_is_home` | `Keira Is Home` | Occupant presence toggle for Keira. Renders iOS System Pink (`#FF2D55`/`#FF375F`) background tint when no media is playing. |
| **Toggle** (`input_boolean`) | `input_boolean.elise_is_home` | `Elise Is Home` | Occupant presence toggle for Elise. Renders iOS System Purple (`#AF52DE`/`#BF5AF2`) background tint when no media is playing. |
| **Dropdown** (`input_select`) | `input_select.house_mode` | `House Mode` | House mode selector used for theming and the **Party page**. Include a `Party` option — selecting it switches the dashboard to the Party page until the mode changes. |

---

## 3. Supported Entity Naming Conventions

The dashboard uses automated string pattern matching on entity IDs and friendly names to automatically discover and map devices to floors, domains, and icons.

### Chemical Sensors & Helpers (`input_number` / `sensor`)
For chemical tracking and recommendation features, create `input_number` helpers or connect sensors containing any of the following keywords in their `entity_id` or `friendly_name`:
- **Free Chlorine**: `free_chlorine`, `fc`, or `chlorine`
- **Total Chlorine**: `total_chlorine` or `tc`
- **pH**: `ph`
- **Total Alkalinity**: `alkalinity` or `ta`
- **Calcium Hardness**: `calcium`, `hardness`, or `ch`
- **Cyanuric Acid**: `cyanuric`, `cya`, or `stabilizer`
- **Salt**: `salt` or `salinity`
- **Water Volume Target**: Entities containing `pool` (assumes ~20,000 gal) or `hottub`/`spa` (assumes ~600 gal).

### Automatic Floor & Area Mappings
Entities are dynamically assigned to floors on the **Controls** tab based on keywords in their `entity_id`, friendly names, or Home Assistant Area/Floor registries:
- **Pool & Spa**: `pool`, `spa`, `hottub`, `hot_tub`, `chlorine`, `ph`, `salt`, `cya`
- **Main Floor**: `main`, `living`, `kitchen`, `dining`, `foyer`, `entry`, `hallway`
- **Upper Floor**: `upper`, `master`, `bedroom`, `upstairs`, `kids`, `bath`
- **Basement**: `basement`, `rec`, `theater`, `gym`, `cellar`, `downstairs`
- **Outdoor**: `patio`, `deck`, `yard`, `outdoor`, `porch`, `garage`

### Master Switches
To list a switch or toggle in the **Master Switches** section on the Overview tab:
- Add the tag or phrase **`Master Room Switch`** or **`ipaddashboard`** to the entity's friendly name or entity ID.

---

## 4. Web Server Deployment

To serve the dashboard to your iPad Mini (or any web browser):
1. Place `index.html` inside your Home Assistant `www/` folder (e.g. `/config/www/dashboard/index.html`).
2. Access the dashboard from your iPad browser at `http://YOUR_HA_IP:8123/local/dashboard/index.html`.
3. Tap the **Share** button in Safari and select **Add to Home Screen** for full-screen kiosk display (`apple-mobile-web-app-capable`).
