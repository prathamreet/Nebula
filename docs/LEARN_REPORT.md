

# Building a Full-Featured Torrent Client with libtorrent, C++, Rust, Tauri & React

This roadmap breaks down the **concepts, technologies, and skills** needed to implement a cross-platform torrent client with a native backend (libtorrent in C++/Rust) and a modern GUI (React + Tauri). Each section lists key topics, official docs or tutorials to study, and tools/best practices.

## BitTorrent Protocol & Peer-to-Peer Networking

* **Peer-to-peer fundamentals:** Understand pure vs. hybrid P2P, peers vs. seeds, and swarms. BitTorrent is a *hybrid* P2P protocol: peers both download and upload, and trackers help new peers join a swarm.
* **Torrent metadata (.torrent) and **Bencoding**:** Learn the .torrent file format (BEP 3) which contains file info, piece hashes, and tracker URLs. Libraries (like libtorrent) handle bencoding, but know how to inspect or generate torrent files.
* **Trackers:** HTTP and UDP trackers (BEP 15) coordinate peers. Clients announce their info to trackers and receive peer lists. Study tracker protocols (send `info-hash`, stats) and how peers exchange lists.
* **Distributed Hash Table (DHT) & Peer Exchange:** Learn Kademlia-based DHT (BEP 5, BEP 44) for tracker-less peer discovery, and the PEX extension for exchanging peers among connected clients. Libtorrent supports standard DHT, PEX, and Local Peer Discovery.
* **Magnet URIs:** Understand magnet links (`magnet:?xt=urn:btih:...`) which encode the torrent info-hash and optionally trackers. Implement parsing of magnets (libtorrent has `parse_magnet_uri`) so users can add torrents without .torrent files. Note BitTorrent v2 introduces a new “btmh” magnet prefix (SHA-256) and hybrid torrents.
* **BitTorrent v1 vs. v2:** Learn changes in BitTorrent v2 (BEP 52).  V2 uses SHA-256 (32-byte) instead of SHA-1 and **Merkle hash trees** for piece hashes. This shrinks metadata and allows block-level validation.  Hybrid torrents can support both schemes simultaneously (two info-hashes, two swarms).
* **Piece selection & transfer:** Study the rarest-first and endgame algorithms, choking/unchoking, and `have`/`bitfield` messaging.  Also learn `uTP` (Micro Transport Protocol, BEP 29) which uses UDP with congestion control, and optional protocol encryption (RC4 or TLS) to obfuscate traffic.
* **NAT Traversal:** BitTorrent often works with NATs. Learn techniques like UDP hole-punching, UPnP/NAT-PMP port mapping, and torrent *local peer discovery* on LAN (multicast) for peers behind NAT.
* **Resource Management:** Understand how peers rate-limit, use quotas, and throttle uploads/downloads. BitTorrent clients typically manage concurrency (max peers, upload slots) and fairness (tit-for-tat sharing).
* **Relevant resources:** Official BEP list and specification (bittorrent.org), libtorrent’s tutorial and blog (e.g. \[libtorrent v2 blog]\[19]). Network-debug tools (e.g. Wireshark) can help analyze peer communication.

## libtorrent Library Internals (C++)

* **Overview:** Libtorrent (Rasterbar) is a *complete C++ BitTorrent library*. It provides all core protocol features (trackers, DHT, PEX, web seeds, uTP, etc.) in an efficient engine. Familiarize yourself with the **API structure**: the central `session` object, `torrent_handle`, `settings_pack`, and the alert notification system.
* **Session & torrents:** Learn how to create and configure a `lt::session`, then add torrents via `add_torrent(params)`. For example, the tutorial shows instantiating a session, setting `save_path`, parsing a magnet URI, and calling `session.add_torrent(p)`. Understand `alert_handler` callbacks for progress, errors, etc.
* **Settings & tuning:** Libtorrent exposes almost every protocol parameter via `settings_pack`. Study key settings (e.g. `listen_interface`, encryption, timeout, connection limits) and performance presets (`min_memory_usage()`). Use the *tuning reference* to see how settings (connection count, I/O threads, etc.) affect memory and speed.
* **Piece picking & storage:** Understand piece selection (`rarest-first`, strict order options, *share mode* to prioritize seeding) and *endgame mode*. Libtorrent handles disk I/O asynchronously with a pool of worker threads so the network loop never blocks on disk. Study how to configure the number of disk threads and cache behavior for performance.
* **Networking layer:** Libtorrent uses Boost.Asio under the hood. It supports serving *multiple torrents on one port in one thread*, and a high-quality uTP (UDP-based) implementation for congestion control. Learn how it manages sockets: IPv4/IPv6 support, listening/accepting, peer connections, and global rate-limiting (`settings_pack`).
* **Extensions & features:** Note support for **HTTP seeding** (BEP 17/19), `no_peer_id`, `compact` peer lists, and **super-seeding**. It also supports IPv6 and Proxy usage. Key features include: multi-file torrents, fast-resume (saving resume data), partial/selective file download, and private torrents (no DHT).
* **Alerts & logging:** Libtorrent communicates via *alerts*. Familiarize with `alert::type` (e.g. `torrent_finished_alert`, `peer_connected_alert`, `session_stats_alert`). For performance analysis, use `session_stats_alert` and the provided Python scripts (`tools/parse_session_stats.py`) to graph stats.
* **Building libtorrent:** Learn to build libtorrent (v2) from source. It supports Boost.Build (b2) or CMake. For CMake, enable options like `encryption=ON` (TLS/Obfuscation) and choose static/shared libs. Follow the build guide carefully (matching configuration macros is critical).
* **Cross-compiling libtorrent:** If you need ARM or other targets, libtorrent’s docs show how to define a toolset for b2 or use CMake with a cross toolchain. In practice, cross-compilation is complex – consider building on the target platform or CI.
* **Debugging/performance:** Learn to use Valgrind/ASAN for memory issues, and built-in counters for profiling. Use example clients shipped with libtorrent (e.g. `bittornado` example) to experiment. Reference the libtorrent reference docs and \[librtorrent blog]\[19]\[58].

## Rust Concepts & C++ Interop (FFI)

* **Rust fundamentals:** Master Rust ownership, borrowing, lifetimes, traits, and error handling (Results). These ensure memory safety and concurrency guarantees when building the backend. **Concurrency** in Rust (threads, `async/await`, channels, `Mutex`/`Arc`, etc.) is crucial when orchestrating libtorrent and Tauri.
* **Cargo and Crates:** Learn Cargo’s workflow and how to structure a workspace (maybe one crate for the C++ FFI wrapper, one for the Tauri app). Understand `Cargo.toml`, workspaces, and Rust edition settings.
* **FFI basics:** Learn how Rust interacts with C and C++ libraries. Key concepts:

  * Declare `extern "C"` functions in Rust and mark them `#[no_mangle]` to create C-callable symbols. Compile Rust as a `cdylib` or `staticlib` for C consumption (via `[lib] crate-type`).
  * In C/C++, call Rust functions by linking against the Rust library; use `extern "C"` declarations in headers. Always match allocators: **“allocators and deallocators come in pairs”**. For example, if Rust allocates (Box::into\_raw), you must free in Rust (Box::from\_raw) not with C++ `delete`.
* **Rust–C++ bridging tools:**

  * **cxx** crate: Provides *safe, zero-overhead FFI* between Rust and C++. You write a Rust `#[cxx::bridge]` module declaring C++ types/functions, and it generates the binding code. It natively supports common types (`String`, `Vec`, `Box`, `unique_ptr`, etc.). Dtolnay’s `cxx` is ideal for calling C++ from Rust (and vice versa) without hand-writing unsafe glue.
  * **autocxx**: If you have a large existing C++ API, `autocxx` can auto-generate safe Rust bindings by combining `bindgen` with `cxx`-style enforcement. Use it if you need many C++ types.
  * **bindgen**: For lower-level bindings, `bindgen` generates Rust `unsafe` declarations from C/C++ headers. Note: bindgen may expose raw pointers and won’t automate constructors/destructors. Use it only if `cxx`/`autocxx` can’t fit your use case.
  * **cbindgen**: (Optional) if you need Rust -> C/C++ bindings, `cbindgen` can generate a C header for your Rust library.
* **FFI Implementation:** Decide whether to build libtorrent as a C++ shared library and call it from Rust, or to compile libtorrent into a static library and link directly. Common approach: build libtorrent as a static lib, then in Rust use `cc` or `cmake` crate in `build.rs` to compile/link it. Alternatively, write thin C wrappers around needed libtorrent APIs and call those from Rust with `extern`.
* **Memory Safety:** When passing data across FFI, carefully manage ownership. For example, convert C++ strings/vectors into Rust `&str`/`Vec<u8>` safely (CXX handles this). Mark FFI pointers as `NonNull` or raw pointers and use `unsafe` blocks. Remember to respect alignment and `#[repr(C)]` if sharing structs.
* **Concurrency and FFI:** If libtorrent runs its own threads, you may call its API from any Rust thread, but ensure thread-safety (e.g. `Arc<Mutex<>>` around non-Send handles). Understand Rust’s `Send`/`Sync` traits – generated bindings in `cxx` enforce many safety checks.
* **Crates & Examples:** Investigate existing crates like `libtorrent-sys` or `libtorrent` (Rust wrappers) on crates.io for guidance. Even if you write your own bindings, example projects (e.g. \[tylerjw\.dev interop series]\[24]) illustrate patterns.
* **Testing FFI:** Write Rust unit tests calling the FFI, and C++ tests for your C wrappers. Use `cargo test` for Rust and the C++ unit test framework of choice. Enable UB sanitizers (ASan, TSAN) during development to catch misuses.

## Tauri Framework & System-Level Integration

* **Tauri overview:** Tauri is a framework to build *lightweight desktop apps* with a Rust backend and a WebView frontend. The backend is a Rust binary exposing commands/APIs, while the UI is built with HTML/CSS/JS (any framework) rendered in the system’s WebView (WKWebView on macOS/iOS, WebView2 on Windows, WebKitGTK on Linux).
* **Architecture:** Learn how Tauri’s architecture works: it uses `tao` for window management and `wry` for a unified WebView abstraction. Tauri employs a multi-process model (backend + webview process). Read the Tauri docs on *Process Model* and *Architecture* to understand IPC and security boundaries.
* **IPC (Rust ↔ JS):** Study how to define and call commands. In Rust, annotate functions with `#[tauri::command]` (or add them to the `invoke_handler`). In React (JS), use the Tauri API: import `{ invoke }` from `@tauri-apps/api/tauri` or use `window.__TAURI__`. For example:

  ```js
  import { invoke } from '@tauri-apps/api/tauri';
  invoke('start_torrent', { torrent_path: path })
    .then(response => console.log('Started:', response))
    .catch(e => console.error(e));
  ```

  The `invoke` call returns a Promise with the Rust return value. Similarly, Rust can emit events to the frontend which React can listen to via `@tauri-apps/api/event`.
* **Commands & Events:** Use Tauri’s command scope configuration (`tauri.conf.json` and the `tauri::Builder`) to expose only specific commands. Ensure each command has permission scopes (Tauri 2.0’s security). Tauri’s event system allows Rust to send async updates (e.g. torrent progress) that JS can `listen()` to. See Tauri’s docs on \[Invoking Rust from JS]\[39] and \[Events]\[80†L86-L94] for examples.
* **Plugins & Sidecars:** Leverage Tauri plugins for additional system features. For example, use the **shell** plugin to spawn external binaries (e.g. if you want a separate tracker or helper executable). Use the **file system** plugin or Tauri APIs (`@tauri-apps/api/fs`) to read/write files (save downloaded data). For notifications, Tauri has a \[notification plugin]\[69]. You can also embed “sidecar” binaries (standalone executables) into your app bundle via `tauri.conf.json`, and spawn them at runtime (e.g. a CLI for advanced tasks).
* **Configuration & Packaging:** Familiarize with `tauri.conf.json` (for window settings, icons, bundle identifiers). Tauri’s bundler can produce OS-native installers (see *Build Systems & Packaging* below). The \[GitHub README]\[38†L354-L360] highlights Tauri’s built-in bundler and updater.
* **Cross-Platform Nuances:** Learn OS specifics: on Windows you may need manifest files, on macOS you must notarize, and on Linux use the correct WebKitGTK library. Tauri handles most via its CLI, but you should understand how to code-sign your binaries (CI docs) and choose appropriate assets.
* **Security:** Tauri is designed with security in mind. Always enable a strict **Content Security Policy (CSP)** and set `tauri.conf.json.security.allowlist` as tight as possible. Understand Tauri’s *trust boundaries*: Rust code is fully privileged, whereas the WebView JS is sandboxed. Only expose safe commands and validate all inputs. Consult the Tauri Security Guide for best practices.

## React Frontend Architecture for Desktop Clients

* **React fundamentals:** Ensure proficiency in React (hooks, JSX/TSX, state management). Use **TypeScript** to catch errors early. Understand how to structure components (e.g. a TorrentList, TorrentDetail, Settings) and manage state (e.g. use `useState` or Context/Redux) for torrent data and UI state.
* **Project setup:** Use a modern bundler. Tauri recommends **Vite** for SPA frameworks like React. For example, create a React+TypeScript app with Vite (`npm init vite@latest`), then integrate Tauri by adding it in the `src-tauri` folder. Ensure your frontend build output (`dist/` or `build/`) is where Tauri expects it.
* **State & UI libraries:** For complex state (multiple torrents, dialogs), consider a state manager (React Context or Redux). Use UI component libraries (Material-UI, Ant Design, or Tailwind) for consistent styling. Design the UI for desktop use: resizable windows, menus, system tray integration.
* **Communication with backend:** Use Tauri’s JS APIs to call Rust: e.g. `invoke('get_torrent_status')` or `@tauri-apps/api/event` to listen for torrent progress. Make all such calls asynchronous (`async/await`). Handle results and errors gracefully. For example, after adding a torrent, subscribe to progress events and update state.
* **Routing & Views:** If your app is multi-page (Add Torrent, Settings, About), use React Router or a similar library. Ensure any navigation is client-side; Tauri’s WebView loads your single-page app. (Tauri **does not** natively support server-side rendering, only client SPA.)
* **Security & Best Practices:** Even though the app is local, sanitize any dynamic data inserted into the DOM to avoid XSS. Use environment variables securely (Tauri allows only certain `window.__TAURI__` calls). Disable any unneeded browser integration.
* **Testing & Dev tools:** Set up React testing (Jest + React Testing Library). Use React Developer Tools for profiling. For layout debugging, open the DevTools (Tauri apps allow Chrome DevTools). Use linting/formatting (ESLint, Prettier). Finally, run `npm run build` (or `vite build`) for production assets.

## Networking, IO, and OS Concepts

* **Network fundamentals:** Know the BSD socket API (or Winsock) concepts: TCP vs UDP, ports, IP addressing. Understand how BitTorrent opens TCP (for peer wire) and UDP (uTP, DHT) sockets. Study concepts like non-blocking sockets and select/epoll for high-performance network code.
* **Asynchronous I/O:** Learn event-driven I/O (e.g. epoll on Linux, kqueue on BSD/macOS, IOCP on Windows). Libtorrent uses Boost.Asio, which abstracts these. Understanding how async I/O works helps in tuning performance and avoiding blocking calls.
* **File I/O and filesystem:** Understand cross-platform file handling (POSIX vs Windows API). Torrent clients often pre-allocate files and write non-sequentially; learn how to use OS APIs to handle large files and avoid fragmentation. Know file permissions (especially on Unix) to safely store downloads.
* **Concurrency and threading:** The OS-level threading model (pthreads on Unix, Win32 threads) underpins both libtorrent and Rust concurrency. Understand thread pools: libtorrent itself spins disk-IO threads. Managing CPU vs I/O bound tasks is key (e.g. hashing is CPU-bound, network I/O is I/O-bound).
* **NAT/Firewall traversal:** Study how BitTorrent reaches peers: NAT-PMP, UPnP (to punch holes), and fallback (Connecting only to publicly reachable peers). Learn about STUN/ICE if planning any future extensions.
* **System resources:** Familiarize with OS limits (max file descriptors, port ranges). On Linux, learn `ulimit` for FDs and how to tune system-level TCP/UDP buffers for thousands of connections. On Windows, know the equivalent limits and the modern IOCP model.
* **Performance tools:** Use `netstat`/`ss` to debug ports, Wireshark to trace BitTorrent protocol messages, and OS tools like `strace`/`dtruss` to trace syscalls. For disk I/O analysis, use `iotop` or Windows Resource Monitor.

## Concurrency and Multithreading

* **Threading vs Async:** Understand the difference between spawning threads vs using async runtimes. Rust has both models (`std::thread` vs `tokio`/`async-std`). Libtorrent uses a thread for networking plus a pool for disk I/O. In Rust, you may choose an async model or use threads to integrate with libtorrent callbacks.
* **Rust concurrency:** Learn `std::thread::spawn`, `Arc`, `Mutex`, `RwLock`, and channel types (`std::sync::mpsc`, `crossbeam`). Understand the `Send` and `Sync` traits – Rust enforces thread-safety at compile time. For async: learn `async/await`, `Future`, and executors like Tokio. Rust’s model avoids data races and deadlocks if used correctly.
* **C++ concurrency:** Know `std::thread`, `std::future`, `std::mutex`, `std::atomic`. Understand thread safety of libtorrent objects (typically `torrent_handle` is thread-safe for status queries, but some operations must happen on the session thread). Use locks or message queues if sharing data structures between threads.
* **Parallelism:** BitTorrent benefits from multi-core: libtorrent parallelizes disk hashing. You might also download multiple peers concurrently. Design your code so CPU-intensive tasks (hashing pieces) and I/O (network/disk) run in parallel.
* **Synchronization:** Carefully coordinate data between Rust and C++ threads. For example, if libtorrent on C++ thread emits an alert that you process on the Rust side, use thread-safe Rust channels or Tauri’s async runtime (`tauri::async_runtime::spawn`) to avoid blocking.
* **Race conditions:** Use tooling like ThreadSanitizer (TSAN) on Rust/C++ to catch races. In Rust, the compiler helps, but FFI edges are unsafe zones—review those carefully.

## Security Considerations

* **Data integrity:** BitTorrent inherently verifies each piece via its SHA-1/256 hash. This guards against corrupted or tampered data. Ensure your client rejects any piece failing the hash check. (Rust’s libraries will panic/abort on mismatch by default, which is safe by design).
* **Encryption & Privacy:** By default, BitTorrent traffic can be sniffed. Libtorrent supports **protocol encryption** (RC4 or TLS). Learn to enable it (`settings_pack`) to obscure traffic. Also allow optional proxy (SOCKS5, HTTP) in libtorrent to route through VPNs/TOR for anonymity.
* **Malicious peers:** Implement IP filtering and banning. Libtorrent has an `ip_filter` feature (e.g. block known bad IPs). Also monitor peers for malicious behavior (sending invalid data) and use libtorrent’s smart ban heuristics.
* **Tauri security model:** Treat Rust backend and WebView as **separate trust domains**. Only expose a minimal set of commands to the frontend. Strictly validate all input from JS (e.g. file paths to save). Use Tauri’s allowlist/capabilities and CSP headers to prevent injection.
* **Memory safety:** Rust helps eliminate buffer overflows and use-after-free in your own code. However, be cautious in `unsafe` FFI blocks. Always pair `new`/`delete` or `Box::into_raw`/`Box::from_raw` correctly. For any raw pointer, ensure proper lifetimes and use tools (ASan) during testing.
* **OS-level:** Run with least privilege. Don’t auto-run executables from downloads. On Windows/macOS, sign your app to prevent tampering. On macOS, comply with notarization (Apple’s guidelines).
* **Dependency security:** Keep libraries (Rust crates, npm packages) up-to-date. Audit especially any native libs. Use tools like `cargo-audit` and `npm audit`.
* **Network attacks:** Protect against denial-of-service (e.g. limit how many peers/connections you accept). Use libtorrent’s rate-limits and connection caps. If using Web APIs, guard against SSRF.
* **Best practices:** Refer to Tauri’s security docs and OWASP guides for desktop apps. For example, avoid `innerHTML` with untrusted data, and never `eval` JS.

## Testing, Debugging, and Performance Profiling

* **Unit & integration testing:** Write unit tests for individual components. In C++, use a testing framework (Catch2, GoogleTest) for libtorrent wrapper code. In Rust, use `cargo test` for your logic and FFI boundary tests. In React, use Jest + React Testing Library to test components and simulate user actions.
* **End-to-end tests:** Consider automating UI tests (e.g. using Selenium or Playwright on the compiled app) to simulate adding/removing torrents and checking UI responses. Tauri supports WebDriver-based tests.
* **Mocking network:** For libtorrent integration tests, run a local tracker or use simple torrent seeds to test interactions without real swarms. Libtorrent’s own tools (like `tools/parse_session_stats.py`) can verify its internal stats.
* **Debugging:** Use debuggers: `gdb`/`lldb` for the Rust/C++ backend (Tauri uses native debuggers in `cargo tauri dev`). For the front end, use browser DevTools (open with `appWindow.openDevTools()`) to inspect React state and console logs. Enable detailed logging (Rust’s `env_logger` or `tracing`) for backend events. Tauri’s \[CrabNebula DevTools]\[88†L1-L4] can log IPC and command events.
* **Profiling (Performance):**

  * **Backend:** Use lttng/perf/gprof to profile the Rust/C++ code. Libtorrent provides *session counters* – call `ses.post_session_stats()` and collect `session_stats_alert` messages to measure queue sizes, throughput, etc. Use the provided Python script to graph these. Also use OS profilers (e.g. `perf record`) to find hot spots.
  * **Frontend:** Use React Profiler (in DevTools) to find slow renders, and Chrome’s performance tab to measure UI responsiveness.
  * **Memory:** Use Valgrind/Memcheck or AddressSanitizer for leaks. In Rust, use `cargo valgrind` or `cargo asan`. Track memory usage under heavy load (many torrents).
* **Static analysis:** Run `clippy` on Rust code, `cppcheck` or clang-tidy on C++. Use linters on JS (ESLint). Ensure formatting with `rustfmt`/`cargo fmt` and Prettier.
* **Continuous Integration:** Set up a CI pipeline to build/test on push. Include steps to run unit tests on all languages, and perhaps a sample torrent download test.

## Build Systems, Cross-Compilation, and Packaging

* **Building libtorrent:** Follow \[the official build docs]\[91]. Libtorrent supports **Boost.Build (b2)** and **CMake**. Choose one:

  * **Boost.Build:** Install Boost (including `boost.build`) and run `b2` (commands are detailed in the docs). You can define toolsets for your compiler (see the Cross-Compiling section). Ensure OpenSSL is available if needed.
  * **CMake:** Install CMake and Ninja (or Make). Configure with `cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -Dencryption=ON ..`. Key CMake flags include `encryption` (TLS/obfuscation) and `static_runtime` if desired. Build and install the library.

* **Rust build:** Use Cargo to build your Rust backend. In your `build.rs`, you might invoke `cc::Build` or `cmake` crates to compile/link libtorrent and any C++ wrappers. Make sure to point Cargo to the libtorrent headers and libs (via `println!("cargo:rustc-link-search=...")` etc.).

* **React build:** Use `npm run build` (Vite) or equivalent to produce static assets. Configure output path to `src-tauri/dist` (or as set in `tauri.conf.json`). Ensure production optimizations.

* **Cross-compilation:** Tauri advises that **true cross-compilation is impractical** due to native dependencies. Instead, build on each target OS. In practice, use continuous integration (GitHub Actions, GitLab CI, etc.) with runners for Windows, macOS, and Linux. (Tauri’s **GitHub Action** can automate building for all platforms.)

* **Packaging:** Tauri includes a bundler to create platform-specific installers:

  * **Windows:** Generates `.exe` installers (NSIS) or `.msi` (WiX).
  * **macOS:** Creates `.app` bundles and can produce a signed `.dmg`.
  * **Linux:** Can output `.AppImage`, `.deb`, `.rpm` packages.
    Configure package metadata in `tauri.conf.json` (app name, icons, identifiers). Use the bundler’s `externalBin` to include any sidecar executables.

* **Code signing & updates:** For production, sign your apps: on macOS with an Apple Developer ID (and notarize via `notarytool`), on Windows with a code-signing certificate. Tauri supports auto-updates via GitHub or custom servers – read the *updater* docs and include update config in your app.

* **Continuous Delivery:** Automate releases: use Tauri’s GitHub Action or custom CI to build and publish installers. Tag releases in GitHub; Tauri’s updater can use GitHub releases to fetch updates.

* **Table: Packaging Formats**

  | Platform | Bundle Formats              | Tools/Notes                                                                                 |
  | -------- | --------------------------- | ------------------------------------------------------------------------------------------- |
  | Windows  | `.exe` (NSIS), `.msi` (WiX) | Tauri’s bundler produces an NSIS installer and optionally MSI. Code-sign with `signtool`.   |
  | macOS    | `.app`, `.dmg`              | Tauri creates a signed `.app` bundle and `.dmg`. Requires Apple code-signing/notarization.  |
  | Linux    | `.AppImage`, `.deb`, `.rpm` | Tauri can generate AppImage, `.deb` and `.rpm` packages. Choose formats for target distros. |

* **Best practices:** Always build in Release mode (`cargo tauri build --release`). Test each platform individually. Keep versioning and CHANGELOG up to date, and verify the auto-update flow in a staging environment. Use CI artifacts to archive each build for distribution.

**Sources:** Official documentation and guides for each technology (libtorrent docs, Rust reference, Tauri guides, React docs) and relevant community/tutorial articles. Each section above cites authoritative docs for key concepts.
