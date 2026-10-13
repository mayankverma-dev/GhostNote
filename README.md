<div align="center">

<img src="favicon.svg" alt="GhostNote Logo" width="80" height="80" />

# GhostNote

**A sleek, distraction-free markdown notepad with instant, zero-backend real-time broadcasting.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Zero Build](https://img.shields.io/badge/Build-Zero%20Dependencies-brightgreen.svg)](#features)
[![Storage: LocalFirst](https://img.shields.io/badge/Storage-LocalFirst%20%28localStorage%29-orange.svg)](#storage-and-privacy)
[![MQTT WebSockets](https://img.shields.io/badge/Sync-MQTT%20over%20WebSockets-00a4e4.svg)](#architecture)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#contributing)

[Live Demo](https://mayankverma-dev.github.io/GhostNote/) • [Key Features](#key-features) • [Quick Start](#quick-start) • [How It Works](#how-it-works) • [Keyboard Shortcuts](#keyboard-shortcuts) • [Configuration](#configuration)

---

</div>

## 📖 Overview

**GhostNote** is an ultra-lightweight, local-first note-taking web application engineered for speed, privacy, and frictionless sharing. It pairs a distraction-free writing environment with an instant, peer-to-peer live broadcasting system that lets you share notes with anyone via a shareable URL—**without accounts, authentication, databases, or application servers.**

Everything runs right inside your browser through a single clean `index.html` file powered by vanilla JavaScript, MQTT over WebSockets, and modern CSS.

```
┌─────────────────┬────────────────────────────────────────────────────────┐
│ 📝 Notes  [+New]│ 🟢 Live · sharing "System Design"   [Copy Link] [Stop] │
├─────────────────┼────────────────────────────────────────────────────────┤
│ System Design * │ Saved  [Preview] [Live share]                          │
│ Meeting Notes   ├────────────────────────────────────────────────────────┤
│ Quick Scratch   │                                                        │
│ Untitled        │ # System Architecture                                  │
│                 │                                                        │
│                 │ Real-time distributed sync using ephemeral MQTT topics.│
│                 │                                                        │
└─────────────────┴────────────────────────────────────────────────────────┘
```

---

<a id="key-features"></a>
<a id="features"></a>
## ✨ Key Features

- **⚡ Zero Build, Zero Install:** No `npm install`, no webpack/vite bundling, no Node.js runtime needed. Open [`index.html`](index.html) in any modern browser and start writing immediately.
- **🔒 Local-First & Private:** Notes are persisted directly to your browser's `localStorage` (`minimal_notes_data`). Your notes remain strictly on your device unless you explicitly initiate a live broadcast.
- **📡 Instant Real-Time Broadcasting (Live Share):**
  - Generate a shareable live room URL (`#live=ROOMID`) in one click.
  - Changes to note titles and content broadcast over MQTT WebSockets in real time as you type (400ms debounce).
  - **MQTT Last Will & Testament (LWT)** triggers an offline alert to viewers if the host closes their tab or loses internet connection.
  - Real-time **Presence Detection** notifies the host via toast notification whenever a peer connects.
- **📥 One-Click Note Import:** Anyone watching a shared broadcast can click **"Import to my notes"** to fork the note straight into their local notebook.
- **✍️ Markdown Editor & Rendered Preview:**
  - High-fidelity markdown rendering with GitHub-flavored styling powered by `marked.js`.
  - Toggle between editing and preview mode seamlessly via <kbd>Ctrl</kbd>+<kbd>P</kbd> or <kbd>Cmd</kbd>+<kbd>P</kbd>.
  - Input protection automatically locks fields while in preview or viewer mode.
- **🎨 Refined Dark Aesthetic:**
  - Curated dark palette with subtle borders, smooth hover transitions, and custom scrollbars.
  - Crisp iconography powered by [Lucide Icons](https://lucide.dev/).
  - Responsive layout with a mobile drawer navigation on viewports under 720px.
- **💾 Reliable Auto-Save:** Keystroke auto-save with a 300ms debounce and dedicated status indicator (`Saved` / `Saving...`).

---

<a id="quick-start"></a>
## 🚀 Quick Start

GhostNote is 100% client-side. You can launch it in any of the following ways:

### Option 1: Direct File Open (Easiest)

Double-click [`index.html`](index.html) or launch it from your command line:

```bash
# Windows (PowerShell)
Start-Process index.html

# macOS
open index.html

# Linux
xdg-open index.html
```

### Option 2: Local Static Server (Recommended)

Running over a local HTTP server provides an ideal environment for WebSockets and clipboard access:

```bash
# Python 3
python -m http.server 3000

# Node.js (npx)
npx serve .

# PHP
php -S localhost:3000
```

Open [`http://localhost:3000`](http://localhost:3000) in your browser.

---

<a id="deploy"></a>
## 🌐 One-Click Deployment

Because GhostNote is entirely static, it can be hosted for free on any static host:

| Platform | Deployment Instructions |
|---|---|
| **GitHub Pages** | Go to **Repo Settings** &rarr; **Pages** &rarr; Set Branch to `main` (root `/`) &rarr; Save. |
| **Vercel** | Run `npx vercel deploy` or import repo directly via dashboard. |
| **Cloudflare Pages** | Connect your Git repository, set build command to empty, build directory to `/`. |
| **Netlify** | Drag and drop the folder into [Netlify Drop](https://app.netlify.com/drop) or run `netlify deploy --prod`. |

---

<a id="architecture"></a>
<a id="how-it-works"></a>
## 🛠️ How It Works (Architecture)

GhostNote operates on a hybrid model: **Browser Local Storage** for private data persistence and a **public MQTT WebSocket broker** for live peer-to-peer broadcasting.

```
+-------------------------------------------------------------------------------+
|                                  HOST CLIENT                                  |
|   [ Browser localStorage ] <=======> [ GhostNote State / Active Note ]        |
+----------------------------------------------------+--------------------------+
                                                     |
                                                     | Publish (QoS 1, retain: true)
                                                     v
                         +---------------------------------------+
                         |       PUBLIC MQTT BROKER (WSS)        |
                         |     (wss://broker.emqx.io:8084/mqtt)  |
                         |                                       |
                         |  Topic: minimalnotesapp/v1/{ROOM_ID}/ |
                         |    ├── /data     (title, content, ts) |
                         |    ├── /status   (active: true/false) |
                         |    └── /presence (viewer beacons)     |
                         +-------------------+-------------------+
                                             |
                                             | Subscribe (Real-time Stream)
                                             v
+-------------------------------------------------------------------------------+
|                                 VIEWER CLIENT                                 |
|  [ Read-Only Live View ] ====( "Import to my notes" )====> [ localStorage ]   |
+-------------------------------------------------------------------------------+
```

### Protocol & Topic Structure

When a host starts **Live Share**:
1. **Room Generation:** A unique 5-character room code is generated (e.g., `B7K9Q`).
2. **Broker Connection:** Connects over WSS to `wss://broker.emqx.io:8084/mqtt`.
3. **LWT (Last Will and Testament):** Registers an LWT on `minimalnotesapp/v1/{ROOM_ID}/status` with `{ active: false }` (`retain: true`, `qos: 1`). If the host window abruptly disconnects, the broker notifies all viewers automatically.
4. **Data Sync (`/data`):** The active note's title and content are published with `retain: true` so new viewers immediately receive the latest document state upon connection.
5. **Presence (`/presence`):** When a viewer joins via `#live=ROOM_ID`, it posts to `/presence`, triggering an instant connection confirmation toast on the host's screen.
6. **Dynamic Note Switching:** Switching to another note while live broadcasts the new note immediately to all viewers without needing a new URL.

---

<a id="keyboard-shortcuts"></a>
## ⌨️ Keyboard Shortcuts

| Shortcut (Win / Linux) | Shortcut (macOS) | Action |
|---|---|---|
| <kbd>Ctrl</kbd> + <kbd>S</kbd> | <kbd>Cmd</kbd> + <kbd>S</kbd> | Force save active note immediately |
| <kbd>Ctrl</kbd> + <kbd>P</kbd> | <kbd>Cmd</kbd> + <kbd>P</kbd> | Toggle between Markdown Edit and Preview modes |

---

<a id="user-guide"></a>
## 🧭 User Guide

### 1. Managing Notes
- **Create a Note:** Click the **`+ New`** button in the sidebar.
- **Edit Note:** Enter a title and markdown content in the editor area. Content auto-saves on every stroke (300ms debounce).
- **Delete a Note:** Hover over any note in the sidebar and click the trash can icon. If the note is actively broadcasting, the live stream is terminated.

### 2. Markdown Preview Mode
- Click the **`Preview`** button in the toolbar or press <kbd>Ctrl+P</kbd> / <kbd>Cmd+P</kbd>.
- Renders headings, blockquotes, code blocks, bullet points, checklists, images, and links.
- Fields are locked into read-only mode to prevent editing mistakes while reading.

### 3. Broadcasting a Note (Host)
- Open any note and click **`Live share`** in the toolbar.
- The shareable link (`https://.../#live=ROOM_ID`) is copied to your clipboard.
- The host banner displays active status with **`Copy link`** and **`Stop sharing`** controls.
- Anyone opening the link will see your note updating live as you type.

### 4. Joining a Broadcast (Viewer)
- Navigate to the shared link containing `#live=ROOM_ID`.
- GhostNote enters **viewer mode**: the document updates live in real-time.
- Click **`Import to my notes`** to clone the note into your own local notebook and continue editing independently.

---

<a id="configuration"></a>
## ⚙️ Configuration & Customization

All configurations are easily accessible directly in [`index.html`](index.html).

### 1. Using a Custom or Private MQTT Broker
By default, GhostNote connects to EMQX's public test broker. You can swap this for any WebSocket-enabled MQTT broker (e.g., Mosquitto, HiveMQ, AWS IoT, or a self-hosted EMQX instance):

```javascript
// Located in index.html (around line 404)
const MQTT_BROKER = 'wss://broker.emqx.io:8084/mqtt'; // Your WebSocket broker endpoint
const TOPIC_ROOT = 'minimalnotesapp/v1';             // Your topic namespace
```

### 2. Theme & Styling
Adjust colors, typography, or spacing by modifying the CSS variables in the `<style>` block:

```css
/* Located in index.html (around line 14) */
:root {
  --bg-main: #18181a;       /* Primary editor background */
  --bg-sidebar: #1d1d1f;    /* Sidebar background */
  --accent: #4fb3e8;        /* Brand accent (blue) */
  --accent-live: #34d399;   /* Live broadcast status (emerald) */
  --danger: #f26d6d;        /* Destructive action accent */
  --text-main: #e4e4e5;     /* Main typography color */
  --text-muted: #85858a;    /* Secondary/date typography */
}
```

---

<a id="storage-and-privacy"></a>
## 🔐 Privacy & Security Considerations

- **Local Storage Boundary:** All notes are stored locally inside the browser (`localStorage['minimal_notes_data']`). No data is transmitted externally unless you explicitly start a live broadcast.
- **Public Broker Advisory:** The default setup uses `broker.emqx.io`, a free and publicly accessible MQTT broker. Although room IDs are randomized, topics are unencrypted at the application layer. **Do not broadcast passwords, private credentials, or confidential secrets over public brokers.**
- **Private Production Setup:** For confidential or production environments, host a private MQTT broker (such as [Eclipse Mosquitto](https://mosquitto.org/) or [EMQX](https://www.emqx.io/)) configured with TLS (`wss://`) and username/password or token authentication.

---

<a id="project-structure"></a>
## 📂 Project Structure

```text
GhostNote/
├── index.html        # Single-file application (HTML layout, CSS styles, JavaScript logic)
├── favicon.svg       # Vector icon for modern browsers
├── favicon.ico       # Multi-resolution fallback icon
├── LICENSE           # MIT License
└── README.md         # Production documentation
```

---

<a id="browser-compatibility"></a>
## 🌐 Browser Compatibility

GhostNote uses standard web APIs and is tested across all modern browsers:

| Browser | Compatibility |
|---|---|
| **Google Chrome / Chromium** | Version 80+ |
| **Mozilla Firefox** | Version 75+ |
| **Apple Safari** | Version 13.1+ |
| **Microsoft Edge** | Version 80+ |
| **Brave / Arc / Opera** | Fully Supported |
| **Mobile Browsers** (iOS Safari, Android Chrome) | Fully Supported (Responsive UI) |

---

<a id="stack"></a>
## 🧰 Technology Stack

- **Markup & Styling:** Vanilla HTML5 & CSS3 with CSS custom properties (variables), Flexbox, and CSS Grid.
- **Logic:** Vanilla JavaScript (ES6+ modular closure pattern).
- **Markdown Parser:** [Marked.js](https://marked.js.org/) via jsDelivr CDN.
- **Icons:** [Lucide Icons](https://lucide.dev/) via unpkg CDN.
- **Real-Time Engine:** [MQTT.js](https://github.com/mqttjs/MQTT.js) via unpkg CDN.

---

<a id="contributing"></a>
## 🤝 Contributing

Contributions, feedback, and feature suggestions are welcome!

1. Fork the Repository (`https://github.com/mayankverma-dev/GhostNote/fork`)
2. Create your Feature Branch (`git checkout -b feature/NewFeature`)
3. Commit your Changes (`git commit -m 'feat: Add NewFeature'`)
4. Push to the Branch (`git push origin feature/NewFeature`)
5. Open a Pull Request

---

<a id="license"></a>
## 📄 License

This project is licensed under the [MIT License](LICENSE) © 2026 [Mayank Verma](https://github.com/mayankverma-dev).

---

<div align="center">
  Built with ❤️ for clean writing and effortless sharing.
</div>
