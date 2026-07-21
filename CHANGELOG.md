# Changelog

All notable changes and features to the Home Assistant iPad Mini Web Dashboard are documented chronologically in this file by version number.

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
