
﻿<img src="https://capsule-render.vercel.app/api?type=soft&color=gradient&height=10&section=header" width="1080" align="center"/>

# Nebula: A High-Performance, Lightweight Torrent Client
Nebula is a cross-platform, blazing-fast BitTorrent client crafted for modern desktops. With its modular architecture and rock-solid performance, Nebula delivers an elegant, smooth experience for power users and casual downloaders alike.

## 🌌 Project Highlights

- Ultra-Responsive UI: Enjoy 60 fps interactions powered by silky-smooth Framer Motion animations.
- Minimal Resource Usage: Stays under 100 MB RAM with finely-tuned libtorrent settings.
- Cross-Platform: Native builds for Windows (x64/ARM), macOS (Intel/Apple Silicon), and Linux (x64/ARM).
- Feature-Rich: Supports magnet links, file-level selection, RSS auto-downloads, and global/per-torrent bandwidth caps.
- Secure & Sandboxed: Leverages Tauri’s default security model with minimal privileges and isolated native code.
- Extensible Architecture: Cleanly separated layers—C++ core, Rust bridge, and React frontend—for ultimate flexibility.


## 🛠 Technology Stack


| Layer            | Technology & Purpose                                                                 |
|------------------|--------------------------------------------------------------------------------------|
| Torrent Engine   | C++17 + libtorrent: Handles peer connections, DHT, and torrent session management.   |
| Build System     | CMake (3.16+) & vcpkg: Manages dependencies and enables multi-platform builds.       |
| IPC / CLI        | JSON-RPC over stdin/stdout: Lightweight command API for seamless communication.      |
| Bridge           | Rust + Tauri: Spawns the daemon, multiplexes commands, and emits events.             |
| Frontend         | React + TypeScript + TailwindCSS: Responsive UI with theming and Zustand state management. |
| Animations       | Framer Motion: Delivers smooth transitions and modal effects.                       |
| State Management | Zustand or Redux Toolkit: Tracks torrents, settings, and UI state effortlessly.     |
| Bundling         | Tauri Bundler: Creates .exe, .dmg, .deb, and .AppImage installers.                   |
| CI/CD            | GitHub Actions: Automates cross-platform build, test, and release workflows.        |



﻿<img src="https://capsule-render.vercel.app/api?type=soft&color=gradient&height=10&section=header" width="1080" align="center"/>

## 📂 Detailed Folder Structure

```sh
nebula/
├── backend/                # C++ torrent engine & CLI
│   ├── include/            # Public headers (torrent_engine.hpp)
│   ├── src/                # Implementation
│   │   ├── main.cpp        # CLI: JSON-RPC dispatcher & daemon mode
│   │   └── torrent_engine.cpp # Core session and settings management
│   ├── tests/              # GoogleTest suites for engine & CLI
│   └── CMakeLists.txt      # Build configuration & dependencies
│
├── src-tauri/              # Rust/Tauri bridge & packaging
│   ├── src/
│   │   ├── commands/       # Command modules: add, pause, resume, status, remove
│   │   └── main.rs         # Tauri builder, event loop, updater plugin
│   ├── assets/             # Icons (.ico, .icns, .png) & splash screens
│   ├── build.rs            # Pre-build tasks (copy backend binary, inject metadata)
│   └── tauri.conf.json     # App metadata, bundle targets, resource inclusion
│
├── frontend/               # React + TypeScript + TailwindCSS UI
│   ├── public/             # index.html, manifest.json, favicon, robots.txt
│   ├── src/
│   │   ├── components/     # Reusable UI: TorrentList, TorrentItem, Modals
│   │   ├── hooks/          # Custom hooks: useTorrents, useSettings
│   │   ├── layouts/        # AppShell: Sidebar, Header, Footer
│   │   ├── pages/          # Views: Dashboard, Feeds, Settings, About
│   │   ├── store/          # Zustand slices: torrents, settings
│   │   ├── utils/          # API wrappers, formatters, validators
│   │   ├── App.tsx         # Route definitions & global providers
│   │   └── index.tsx       # Entry point & render root
│   ├── tailwind.config.js  # Design tokens, dark mode, plugins
│   └── vite.config.ts      # Path aliases (@/ → src/), optimization tweaks
│
├── scripts/                # Automation scripts
│   ├── build-all.sh        # Orchestrates C++ → frontend → Tauri build
│   ├── clean.sh            # Cleans all build artifacts
│   └── release.sh          # Tags version, builds bundles, publishes to GitHub Releases
│
├── docs/                   # Documentation
│   ├── architecture.md     # High-level design & module diagrams
│   ├── coding-standards.md # C++/Rust/JS style guides, lint configs
│   ├── user-guide.md       # End-user manual, FAQs, troubleshooting
│   └── dev-setup.md        # Local dev environment setup & known issues
│
├── .github/                # CI/CD & community templates
│   ├── workflows/
│   │   ├── ci.yml          # Build/test on push/PR across OSs
│   │   └── release.yml     # Automated release on Git tag
│   └── ISSUE_TEMPLATE.md   # Bug & feature request templates
│
├── .editorconfig           # Consistent formatting rules
├── .gitignore              # Git ignore rules
├── LICENSE                 # MIT License
├── README.md               # This file
└── CONTRIBUTING.md         # Contribution guidelines & code of conduct
```

## 🚀 Getting Started

### Prerequisites
- C++17 Compiler: GCC/Clang (Linux/macOS) or MSVC (Windows).
- CMake ≥ 3.16 & vcpkg: Installs libtorrent, Boost, and OpenSSL dependencies.
- Rust & Cargo: Install via `cargo install tauri-cli`.
- Node.js ≥ 16 & npm (or Yarn): For frontend development.



## Development Workflow

Clone the Repository:
```sh
git clone https://github.com/your-org/nebula.git
cd nebula
```
Build the C++ Backend:
```sh
cd backend
mkdir build && cd build
cmake .. -DCMAKE_TOOLCHAIN_FILE=[vcpkg-root]/scripts/buildsystems/vcpkg.cmake
cmake --build .
```
Set Up the Frontend:
```sh
cd ../../frontend
npm install
npm run dev  # Runs at http://localhost:3000
```
Launch the Tauri Bridge:
```sh
cd ../src-tauri
cargo tauri dev
```
Tip: Run these processes in parallel using `tmux` or multiple terminal panes.


## 🏭 Production Build & Packaging

Build installers for all platforms with a single command:
```
chmod +x scripts/build-all.sh
./scripts/build-all.sh
```

Find the installers in `src-tauri/target/release/bundle/:` `.exe` (NSIS), `.dmg`, `.AppImage`, `.deb`.

### 🔧 Configuration & Environment

- Engine Config (backend/config.json): Customize save paths, DHT settings, and peer limits.
- User Settings: Persisted via Tauri—adjust theme, download directory, and bandwidth caps.
- Environment Variables:


`NEBULA_BACKEND_LOGLEVEL:` Set to `debug`, `info`, `warn`, or `error`.

`NEBULA_FRONTEND_PORT:` Override the default dev server port.

### 🧩 System Architecture

Nebula’s architecture is built for efficiency and clarity:

- React Frontend: Sends commands via Tauri’s `invoke()`.
- Tauri (Rust): Manages the daemon, handles JSON-RPC, and relays events.
- Torrent CLI Daemon: Drives libtorrent for P2P operations and session management.
- libtorrent: Powers core torrent functionality—peers, DHT, and more.

Events flow back to the UI for real-time updates.


### 🧪 Testing & QA

Backend (C++):
```sh
cd backend/build && ctest --output-on-failure
```
Bridge (Rust):
```sh
cd src-tauri && cargo test
```
Frontend (JS/TS):
```sh
cd frontend && npm run lint && npm run test
```

- E2E Tests: Run Cypress/Playwright tests in `frontend/e2e/.`

### 📈 CI/CD Pipelines

- CI (`ci.yml`): Lints, builds, and tests across Linux, macOS, and Windows on push/PR.
- Release (`release.yml`): Tags a version, builds installers, and publishes to GitHub Releases.


﻿<img src="https://capsule-render.vercel.app/api?type=soft&color=gradient&height=10&section=header" width="1080" align="center"/>


### 🤝 Contributing

We welcome all contributions! Whether it's fixing a typo or implementing a major feature, you're helping make the project better.

📄 Read our [Contributing Guidelines](docs/CONTRIBUTING.md) to learn how to fork, branch, commit, and open pull requests the right way.

---

### 📚 Learning Resources

Want to get up to speed before contributing?

💡 Check out [What You Need to Know](docs/LEARNING.md) — a curated list of concepts, tools, and topics used in this project.


﻿<img src="https://capsule-render.vercel.app/api?type=soft&color=gradient&height=10&section=header" width="1080" align="center"/>
