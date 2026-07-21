# Changelog

All notable changes and features to the Home Assistant iPad Mini Web Dashboard are documented chronologically in this file by version number.

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
