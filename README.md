This is an iOS 9 compatable web dashboard for Home Assistant. It's optimized for use on an iPad Mini gen 1, but should fit other display models and sizes well.
Parts of this project were written with Gemini. I have read and edited almost all of the code, but I still don't reccomend running this in a secure environment.

Below is an AI generated description for the project:

```markdown
# iOS 9 Home Assistant Web Dashboard

A custom, high-performance, single-page web dashboard designed specifically to repurpose legacy wall-mounted iPads (running iOS 9 / WebKit) into Home Assistant control panels.

## 🚀 How to Set Up
1. **Configure Credentials**: Open [index.html](file:///Users/ryan/.gemini/antigravity/scratch/ha-controls/index.html) and update the `CONFIG` variables at the top of the script tag:
   ```javascript
   var HA_URL = "http://YOUR_HA_IP:8123";
   var HA_TOKEN = "YOUR_LONG_LIVED_ACCESS_TOKEN";
   ```
2. **Label Dashboard Entities**: In Home Assistant, assign the label `ipaddashboard` to any entity (switches, lights, fans, media players, sensors) you want to show up on the dashboard.
3. **Deploy**: Upload `index.html` to your Home Assistant `/config/www/local/` directory (making it accessible at `http://YOUR_HA_IP:8123/local/index.html`).

---

## 💪 Strengths
* **Legacy WebKit Compatibility**: Engineered using strict ES5 JavaScript and `-webkit-flex` layouts. It will not crash or hang on old WebKit rendering engine engines (tested on iOS 9).
* **Zero Dependencies**: Entire dashboard runs inside a single, lightweight HTML file with no build steps or bundlers.
* **Smart Pool/Hot Tub Math**: Automatically calculates chemical dosages and links low chlorine alerts directly to custom Home Assistant timers.

---

## ⚠️ Weaknesses
* **Manual Code Configuration**: Changing visible items or backend paths requires editing the HTML/JS file directly.
* **Strict Syntax Boundaries**: Adding modern ES6 code (e.g., `let`, `const`, arrow functions) will break iOS 9 rendering.
* **Label Dependency**: Entities will not display unless they are explicitly tagged with the `ipaddashboard` label in Home Assistant.
```
