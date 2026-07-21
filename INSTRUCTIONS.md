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
var REFRESH_INTERVAL = 15000;            // Data refresh polling interval in ms (15 seconds)
```

### How to generate a Long-Lived Access Token in Home Assistant:
1. Log into your Home Assistant web interface.
2. Click your **Profile** icon at the bottom of the left sidebar.
3. Scroll down to the **Long-Lived Access Tokens** section.
4. Click **Create Token**, name it `iPad Dashboard`, and copy the token string into `HA_TOKEN`.

---

## 2. Required Home Assistant Helpers

The dashboard relies on specific Home Assistant **Helpers** (`input_boolean` and `input_text`) for dynamic welcome messages and pool chemical tracking state gates. Create these in **Settings -> Devices & Services -> Helpers**:

| Helper Type | Entity ID | Friendly Name | Description |
| :--- | :--- | :--- | :--- |
| **Text** (`input_text`) | `input_text.dashboard_welcome_message` | `Dashboard Welcome Message` | Sets the custom welcome text shown at the top-left of the header bar (e.g., "Welcome Home, Family!"). Defaults to "Welcome Home" if unavailable. |
| **Toggle** (`input_boolean`) | `input_boolean.chemicals_checked_today` | `Chemicals Checked Today` | Tracks whether chemical levels have been logged today. Displays a yellow warning banner when turned `off`. |
| **Toggle** (`input_boolean`) | `input_boolean.chlorine_logged_since_changing` | `Chlorine Logged Since Changing` | Safety gate used by pool/spa chemical dosage recommendations to prevent accidental chlorine overdosing. |

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
