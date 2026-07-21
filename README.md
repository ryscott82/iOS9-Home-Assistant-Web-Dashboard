# Home Assistant iPad Mini Web Dashboard

A ultra-minimal, high-readability Home Assistant web dashboard designed specifically for wall-mounted tablet displays (including legacy devices such as the 1st Gen iPad Mini running iOS 9 Safari).

---

## ⚠️ Important Security Disclaimers & Warnings

> [!CAUTION]
> **NOT FOR PRODUCTION OR HIGH-SECURITY ENVIRONMENTS**  
> This software is designed strictly for local, private home automation wall displays and should **NOT** be exposed publicly to the internet or deployed in high-security environments.

> [!WARNING]
> **SECURITY RISK: HARDCODED ACCESS TOKENS**  
> Embedding a Home Assistant Long-Lived Access Token directly into client-side source code (`HA_TOKEN` in `index.html`) or browser `localStorage` is a known security risk. Anyone with access to the client device, source code, or browser developer tools can inspect the token and gain administrative access to your Home Assistant instance. Use dedicated low-privilege accounts or local network isolation whenever possible.

> [!NOTE]
> **AI GENERATED & HUMAN REVIEWED**  
> This codebase was built with the assistance of Google Gemini (Antigravity AI) and has been human-reviewed, iterated, and verified for real-world hardware deployment.

---

## 🌟 Key Features

### 1. iPad Mini 1st Gen & Legacy Browser Compatibility
- Crafted with strict ES5 JavaScript, zero modern build steps, and explicit `-webkit-flex` vendor prefixes.
- Fully compatible with iOS 9 Safari, wall-mounted iPad displays, and modern browsers alike.
- Configured with Web App meta tags (`apple-mobile-web-app-capable`, `viewport-fit=cover`, `black-translucent` status bar) for immersive full-screen presentation.

### 2. High-Readability Overview Page
- **Massive 30px Title Text Scale**: Feature names render in a uniform 30px bold font across tiles, filling the card naturally without truncated `...` text.
- **Display Height Fitting**: Cards expand dynamically to fit the display height evenly.
- **Zero Room Labels**: Omitted redundant room headers on Overview cards to maximize title readability from across the room.

### 3. Persistent Unified Header
- Single-row top header persistent across all views (`Overview`, `Controls`, `Chemicals & Sensors`):
  - **Top Left**: Custom welcome message (`Welcome Home` or custom Home Assistant input text), dynamically scaled up to 1/3 page width (`max-width: 33vw`).
  - **Top Right**: Live Date & Time readout (`Monday, Jan 1 • 12:00 PM`) + 3-line SVG hamburger menu button for 1-tap navigation to detailed controls.

### 4. Smart Thermostat & Water Heater Controls
- **1° Whole-Number Adjustments**: `[ - ]` and `[ + ]` buttons increment/decrement in 1° whole numbers.
- **Nearest Integer Rounding**: All target and actual temperatures are formatted using `Math.round()` to eliminate decimal clutter.
- **Vertically Stacked Readouts**: Actual temperature (`73°`) is displayed on top (`19px` bold), and set temperature (`Set 72°`) is stacked underneath (`13px` bold), saving ~50px of horizontal card space.
- **Automatic HVAC Mode Switching**: Setting a target temp above current room temp automatically switches HVAC mode to **`heat`**; setting target temp below current room temp automatically switches HVAC mode to **`cool`**.
- **Red Outline Highlight**: Active heating/cooling tiles display a red border (`#ff3b30`). When target temperature equals actual room temperature, the tile retains the red outline but clears the inner background fill.

### 5. Hot Tub 100°F Fire Quick-Boost Button
- **Standard Flame SVG Icon**: Features a clean, standard classic Flame SVG button (`40px x 40px`).
- **Active / Inactive Toggle State**:
  - **Inactive (Target != 100°F)**: Button renders in muted gray (`#8e8e93`). Tapping it boosts the Hot Tub target temperature to **`100°F`**.
  - **Active (Target == 100°F)**: Button renders in glowing orange (`#ff9f0a` border & fill). Tapping it lowers the Hot Tub target temperature to **`68°F`**.
- **0ms Optimistic Visual Feedback**: Direct DOM element targeting (`btn_fire_<eid>`) updates button styling instantly (<1ms) when pressed.

### 6. Overview Media Player Cards
- Displays speaker/room names in big bold title font (`30px`), with track title and artist (`Title • Artist`) in subtext.
- Integrated **Play/Pause** and **Next Track (Skip)** media buttons.
- **15-Minute Pause Expiry**: Paused media players automatically hide from the Overview tab after 15 minutes of inactivity (`last_changed` > 15 mins).

### 7. Dual-Protocol Communication Engine
- **WebSocket Protocol**: Connects directly to Home Assistant WebSocket API (`ws://<HA_IP>:8123/api/websocket`) for real-time state synchronization.
- **HTTP REST API Fallback**: Sends service commands over HTTP POST (`/api/services/<domain>/<service>`) with 300ms debouncing and 3-attempt retries for guaranteed execution.
- **Global `getNextMsgId()` Handler**: Safe message ID generator preventing WebSocket scope crashes.

---

## 🛠 Setup & Usage

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/ryscott82/iOS9-Home-Assistant-Web-Dashboard.git
   cd iOS9-Home-Assistant-Web-Dashboard
   ```

2. **Configure Connection**:
   - Open `index.html` in your text editor.
   - Update `HA_URL` and `HA_TOKEN` near the top of the `<script>` section:
     ```javascript
     var HA_URL = "http://192.168.40.88:8123";
     var HA_TOKEN = "YOUR_LONG_LIVED_ACCESS_TOKEN_HERE";
     ```
   - Alternatively, open the dashboard in a browser and click the connection status badge in the footer to configure credentials via the setup overlay.

3. **Host & Serve**:
   - Serve the directory using any web server (e.g. Nginx, Apache, or Python HTTP server):
     ```bash
     python3 -m http.server 8080
     ```
   - Access the dashboard on your tablet or wall-mounted display at `http://<SERVER_IP>:8080`.

---

## 📜 Version History & Changelog

All chronological releases, feature updates, and bug fixes are documented in [`CHANGELOG.md`](CHANGELOG.md).
