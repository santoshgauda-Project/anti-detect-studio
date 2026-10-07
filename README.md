# 🛡️ Anti-Detect Studio

> **One Browser. Unlimited Isolated Profiles.**  
> A professional, Chromium-based desktop workspace for privacy-focused multi-profile management and browser automation.

---

## 📌 Product Principle & Intended Use

Anti-Detect Studio is inspired by the multi-profile architecture of tools like BitBrowser, engineered specifically for **legitimate privacy separation, QA testing, development, and multi-session operations**.

> [!IMPORTANT]
> **Anti-Detect Studio does NOT claim to make users "undetectable".**  
> We do NOT implement mechanisms intended to bypass CAPTCHA, defeat website access controls, defeat fraud prevention systems, or violate website terms of service. The application focuses on **strict local session isolation, dedicated storage partitions, and consistent, customizable browser characteristics**.

### Legitimate Use Cases
- **Multi-Account QA & Testing**: Test web applications under distinct user sessions and screen resolutions without cookie bleed.
- **E-Commerce Operations**: Manage multiple vendor storefronts and store dashboards with dedicated cookies and cached assets.
- **Privacy Separation**: Separate work, personal, and research browsing activities into isolated sandbox containers.
- **Automated Workflow QA**: Drive independent browser profiles programmatically with Playwright, Puppeteer, or Python over Chrome DevTools Protocol (CDP).

---

## ✨ Features

- **🔒 True Local Profile Isolation**: Each profile receives an independent user-data directory with its own SQLite/LevelDB storage, cookies, IndexedDB, and cache.
- **🖥️ Configurable Hardware Characteristics**:
  - Operating System Identity presets (Windows 10/11, macOS Sonoma, Linux)
  - Synchronized User-Agent strings and platform headers
  - Screen resolution presets (1080p, 1440p, 4K, custom) & Device Scale Factor
  - Hardware Concurrency (CPU Cores) & Device Memory (RAM)
  - Timezone emulation & Accept-Language locales
- **🛡️ Privacy & WebRTC Protection**:
  - WebRTC UDP policies (`disable_non_proxied_udp`, `disable`, `default`)
  - HTML5 Canvas, WebGL, and AudioContext fingerprint behavior toggles
  - Do Not Track, HTTPS-First upgrades, third-party cookie controls
- **🌐 Proxy Manager**:
  - HTTP, HTTPS, and SOCKS5 proxy support
  - Integrated latency and ping testing tool
  - AES-256-GCM encrypted credential storage
  - One-click copy/paste proxy string parser (`host:port:user:pass` or `protocol://user:pass@host:port`)
- **🤖 Built-in Local REST API (Port 36900)**:
  - BitBrowser-compatible local HTTP REST endpoints
  - Bearer token authentication
  - Launch, stop, and inspect profiles via code
  - Returns `wsEndpoint` for instant Playwright/Puppeteer CDP connection
- **🎨 Sleek Dark Design System**:
  - Inter typography with custom dark glassmorphic UI
  - Real-time session indicators with animated pulse dots
  - Color-coded profile tags and group management
  - Dense Table View vs Card Grid View toggle
  - Live activity log console with level filtering and export

---

## 🚀 Quick Start

### 1. Launch Pre-Packaged Binary (Windows)
The production Windows 64-bit binary has been compiled and is ready to run:
```bash
dist/win-unpacked/anti-detect-studio.exe
```

### 2. Run in Development Mode
```bash
# Install dependencies
npm install

# Start development server with hot-reload
npm run dev
```

### 3. Build & Package
```bash
# Typecheck both Node and Web TypeScript
npm run typecheck

# Build Vite and Electron bundles
npm run build

# Package unpacked application directory (dist/win-unpacked/)
npm run build:unpack
```

---

## 🤖 REST API & Automation Guide

When enabled in **Settings → REST API**, Anti-Detect Studio runs an embedded HTTP server on `http://127.0.0.1:36900` secured by Bearer token authentication.

### Available Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Health check and API server status |
| `GET` | `/api/profiles` | List all configured profiles with status |
| `GET` | `/api/profiles/:id` | Get details and specs of a specific profile |
| `POST` | `/api/profiles` | Create a new isolated profile |
| `POST` | `/api/browser/start` | Launch a browser profile; returns CDP `wsEndpoint` |
| `POST` | `/api/browser/stop` | Stop a running profile instance |
| `POST` | `/api/browser/restart` | Restart a profile instance |
| `GET` | `/api/browser/running` | List IDs of currently running profiles |

### Python (Playwright) Example

```python
import requests
from playwright.sync_api import sync_playwright

API_BASE = "http://127.0.0.1:36900/api"
HEADERS = {"Authorization": "Bearer YOUR_API_TOKEN"}

# 1. Start profile
res = requests.post(f"{API_BASE}/browser/start", json={"id": "PROFILE_ID_HERE"}, headers=HEADERS)
data = res.json()
ws_endpoint = data.get("wsEndpoint")

# 2. Attach Playwright over Chrome DevTools Protocol
with sync_playwright() as p:
    browser = p.chromium.connect_over_cdp(ws_endpoint)
    context = browser.contexts[0]
    page = context.new_page()
    page.goto("https://www.google.com")
    print("Page Title:", page.title())

# 3. Stop profile
requests.post(f"{API_BASE}/browser/stop", json={"id": "PROFILE_ID_HERE"}, headers=HEADERS)
```

### Node.js (Puppeteer) Example

```javascript
const puppeteer = require('puppeteer-core');

async function automate() {
  const res = await fetch('http://127.0.0.1:36900/api/browser/start', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': 'Bearer YOUR_API_TOKEN'
    },
    body: JSON.stringify({ id: 'PROFILE_ID_HERE' })
  });
  const { wsEndpoint } = await res.json();

  const browser = await puppeteer.connect({ browserWSEndpoint: wsEndpoint });
  const [page] = await browser.pages();
  await page.goto('https://example.com');
  console.log(await page.title());
}

automate();
```

---

## 📁 Architecture & File Layout

```
anti-detect-studio/
├── dist/win-unpacked/         # Packaged production desktop application
│   └── anti-detect-studio.exe # Main executable
├── out/                       # Compiled bundles (main, preload, renderer)
├── src/
│   ├── main/                  # Electron Main Process
│   │   ├── index.ts           # App lifecycle, single-instance lock, tray
│   │   ├── browser-manager.ts # Chromium spawn, CLI args, CDP extraction
│   │   ├── database.ts        # Atomic file-backed persistent data store
│   │   ├── ipc-handlers.ts    # Type-safe IPC channels implementation
│   │   ├── api-server.ts      # Local Express REST automation server
│   │   ├── security.ts        # AES-256-GCM encryption for proxy passwords
│   │   └── logger.ts          # Structured audit and activity logging
│   ├── preload/               # Preload bridge
│   │   └── index.ts           # Secure contextBridge API definition
│   ├── shared/                # Shared types and constants
│   │   ├── types.ts           # Complete TypeScript interface contracts
│   │   └── constants.ts       # Presets, resolutions, timezones, IPC keys
│   └── renderer/              # Modern React 18 UI
│       ├── index.html         # HTML root with Inter font
│       └── src/
│           ├── index.css      # Comprehensive dark theme design system
│           ├── main.tsx       # React root mount
│           ├── App.tsx        # View routing, state coordination, layout
│           ├── stores/        # Zustand state store
│           └── components/
│               ├── common/    # TitleBar, Sidebar, Modal, ToastContainer
│               ├── profiles/  # ProfileList, ProfileCard, TableRow, Wizard
│               ├── groups/    # GroupsPage
│               ├── proxies/   # ProxiesPage with latency testing
│               ├── logs/      # LogsPage console viewer
│               ├── settings/  # SettingsPage
│               └── welcome/   # WelcomeScreen for first-run users
└── electron-builder.yml       # Electron builder configuration
```

---

## 📄 License
MIT License. Built for developers, QA engineers, and multi-session operations.
