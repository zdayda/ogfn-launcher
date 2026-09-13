<div align="center">

# OGFN Launcher

**A lightweight, cross-backend launcher for OGFN (OG Fortnite) private servers — Windows only.**

[![Build](https://img.shields.io/github/actions/workflow/status/OWNER/REPO/release.yml?branch=main&label=build)](https://github.com/OWNER/REPO/actions)
[![Release](https://img.shields.io/github/v/release/OWNER/REPO?include_prereleases)](https://github.com/OWNER/REPO/releases)
[![Platform](https://img.shields.io/badge/platform-Windows%2010%2F11-blue)](#-for-users)
[![License](https://img.shields.io/github/license/OWNER/REPO)](LICENSE)

Built with **C++ + vanilla HTML/CSS/JS** — no Electron, no frameworks, no bloat.

</div>

---

## About

OGFN Launcher is a generic, open-source desktop launcher for OG Fortnite private servers.
It is **not tied to any single backend**: point it at any OGFN-compatible server via its
configurable backend URL, import or download your Fortnite builds, and launch.

The UI is plain HTML/CSS/JS rendered inside **Microsoft Edge WebView2**, hosted by a small
native **C++** core that handles everything performance- and system-critical: resumable
downloads, hash verification, process orchestration, and injection.

<!-- Screenshots: add /docs/screenshots images here for v1.0 -->

## Features

| Area | What you get |
|---|---|
| **Build Library** | Import existing local Fortnite installations, or download builds from a configured archive |
| **Downloads** | Chunked, resumable downloads (handles 20–40 GB builds), SHA-256 verified against a manifest |
| **Launching** | One-click launch of any build, per-build launch arguments, multi-client support |
| **Backend** | Login against any OGFN-compatible backend (Epic-style OAuth endpoints); configurable URL |
| **Injection** | Loads the backend-provided auth DLL through the standard `FortniteLauncher.exe` proxy method |
| **Updates** | Built-in launcher self-updater with release channels (stable / beta) |
| **QoL** | News feed from your backend, structured log viewer, settings page, per-build notes |
| **Packaging** | Signed Inno Setup installer, bundles the VC++ redistributable |

## Architecture

```
┌────────────────────────────────────────────────────┐
│                  OGFN Launcher (Win32)             │
│                                                    │
│  ┌──────────────────┐        ┌──────────────────┐  │
│  │   WebView2 UI    │◄──────►│   C++ Core       │  │
│  │   (src/web)      │ bridge │   (src/native)   │  │
│  │                  │        │                  │  │
│  │  vanilla HTML    │  JSON  │  downloads.cpp   │  │
│  │  vanilla CSS     │ msgs   │  crypto.cpp      │  │
│  │  vanilla JS      │        │  process.cpp     │  │
│  │  — no frameworks │        │  config.cpp      │  │
│  └──────────────────┘        │  updater.cpp     │  │
│                              └────────┬─────────┘  │
└───────────────────────────────────────┼────────────┘
                                        │ HTTPS (WinHTTP)
                              ┌─────────▼─────────┐
                              │  OGFN Backend(s)  │
                              │  auth · news ·    │
                              │  manifest · lobby │
                              └───────────────────┘
```

- **`src/web`** — the entire UI in vanilla HTML/CSS/JS. Served locally via WebView2's
  `SetVirtualHostNameToFolderMapping` (no local HTTP server, no origin issues).
- **`src/native`** — C++20 core. Win32 window + WebView2 host, JSON message bridge
  (`window.chrome.webview` ⇄ `PostWebMessageAsJson`).
- **Communication** — a single, typed JSON bridge protocol; the UI never touches the OS directly.

### Native modules

| Module | Responsibility |
|---|---|
| `main.cpp` | Win32 entry point, window creation, WebView2 environment host |
| `bridge.cpp` | Bidirectional JS ⇄ C++ message protocol, request routing |
| `downloads.cpp` | WinHTTP chunked downloads, HTTP range resume, retry/backoff |
| `crypto.cpp` | SHA-256 verification (Windows CNG / BCrypt) |
| `process.cpp` | `CreateProcessW` orchestration, auth DLL proxy launch, client monitoring |
| `config.cpp` | JSON config persistence in `%APPDATA%\OGFNLauncher\` |
| `updater.cpp` | Version check + self-update against the release channel |

## For Users

1. Grab the latest installer from [**Releases**](https://github.com/OWNER/REPO/releases).
2. Run it — the installer bundles the VC++ redistributable. WebView2 is preinstalled on
   Windows 10 (1809+) and Windows 11; the installer bootstraps it if missing.
3. On first launch, enter your backend URL and log in.
4. Import a build from disk, or pick one from your backend's build archive.

## Building from Source

**Prerequisites**

- Windows 10 (1809+) or Windows 11
- Visual Studio 2022 with the *Desktop development with C++* workload (MSVC v143)
- CMake 3.20+
- WebView2 SDK (pulled automatically via NuGet/vcpkg)

```bash
git clone https://github.com/OWNER/REPO.git
cd REPO
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --config Release
```

The debug build loads `src/web` live from disk, so you can edit the UI and hit `F5`
in the WebView2 DevTools without recompiling.

## Configuration

`%APPDATA%\OGFNLauncher\config.json`

```json
{
  "backendUrl": "https://your-ogfn-backend.example.com",
  "releaseChannel": "stable",
  "defaultBuildPath": "D:/Games/Fortnite Builds",
  "launchArguments": {},
  "keepLogs": true
}
```

## Backend API Contract

The launcher is **backend-agnostic**. Any server implementing this minimal contract works —
including Epic-style OAuth endpoints exposed by LawinServer/Neonite-compatible backends.
A reference backend (Node.js) is planned under `/backend`.

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/account/api/oauth/token` | Login (Epic-style OAuth token exchange) |
| `GET` | `/account/api/oauth/verify` | Token validation on launch |
| `GET` | `/launcher/api/news` | News feed entries shown on the home tab |
| `GET` | `/launcher/api/manifest` | Build list: version, size, SHA-256, download URL |
| `GET` | `/launcher/api/status` | Backend status / maintenance banner |

> The launcher only needs the token endpoints to authenticate; the game itself talks to
> the lobby backend (XMPP/MCP) independently after launch.

## Roadmap

- [x] **v1.0.0 — Core**
  - Backend login (configurable URL, Epic-style OAuth)
  - Build import from local folders
  - Resumable manifest-based downloads with SHA-256 verification
  - Launch builds on Windows
- [x] **v1.1.0 — Quality**
  - Launcher self-updater with stable/beta channels
  - Settings page, structured log viewer, per-build launch arguments
  - Multi-client launching
- [ ] **v1.2.0 — Polish** *(this release)*
  - Injection manager UI (DLL library, per-build presets)
  - News feed and backend status dashboard
  - UI theming (light/dark/custom accent)
  - Signed installer + verified GitHub releases

## Project Layout

```
ogfn-launcher/
├── src/
│   ├── native/          # C++20 core (Win32, WebView2 host, bridge, services)
│   └── web/             # Vanilla HTML/CSS/JS UI
│       ├── index.html
│       ├── css/
│       └── js/
├── installer/           # Inno Setup script
├── backend/             # Reference Node.js backend (planned)
├── docs/                # Bridge protocol spec, screenshots
├── .github/workflows/   # CI: build + signed release on tag
└── CMakeLists.txt
```

## Contributing

PRs are welcome. Please:

1. Keep the **no-framework rule** for `src/web` (vanilla HTML/CSS/JS only).
2. Keep C++ conformant to C++20 and warning-clean on MSVC.
3. Run a build before submitting — CI must pass.
4. Discuss larger changes (bridge protocol, new native modules) in an issue first.

## Legal Disclaimer

This project is an unofficial, community-made launcher. It is **not affiliated with,
endorsed by, or connected to Epic Games, Inc.** in any way. Fortnite is a trademark of
Epic Games, Inc.

- The launcher **does not distribute any game files, builds, or Epic Games assets**.
  Builds are imported by the user from their own copies or downloaded from archives
  they configure themselves.
- The launcher **does not circumvent any DRM**; it launches builds against private,
  community-run backends.
- Use of private servers may violate the Epic Games Terms of Service. You assume all
  responsibility for how you use this software.

## License

Distributed under the [MIT License](LICENSE). Third-party libraries remain under their
own licenses.

## Acknowledgements

The OGFN ecosystem stands on the shoulders of community projects — credit and thanks to
the authors of **LawinServer**, **Neonite**, **Reboot Launcher**, and every build archivist
keeping old versions alive. This launcher aims to be a clean, generic front-end for all of them.

---

<div align="center">
<sub>OGFN Launcher — because a launcher should be smaller than the game.</sub>
</div>
