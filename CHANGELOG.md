# Changelog

All notable changes and features to the Home Assistant iPad Mini Web Dashboard are documented chronologically in this file by version number.

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
- **Uniform 30px Card Title Font Scale**: Replaced length-based step scaling with a consistent, uniform `30px` bold title font across all Overview cards (using natural 1-2 line wrapping for title length).

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

## [v1.0.0] - 2026-07-20
- Initial release of the iPad Mini 1st Gen Home Assistant Web Dashboard (iOS 9 Safari compatible, WebSocket + HTTP REST API fallback).
