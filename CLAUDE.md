# CLAUDE.md

Guidance for Claude Code (and other AI agents) working in this repository.

## Project

`ttyd` — a command-line tool for sharing a terminal over the web. A small C backend
serves a single-page web app (xterm.js) and bridges browser WebSocket connections to a
PTY running an arbitrary command.

**This repository is a maintained fork.** Upstream `tsl0922/ttyd` is effectively inactive
and carries unfixed bugs. Development happens here (`lamngockhuong/ttyd`); the `upstream`
remote is kept only for reference/cherry-picking. Do **not** target `upstream` for changes.

- `origin` → `lamngockhuong/ttyd` (this fork — push here, open PRs here)
- `upstream` → `tsl0922/ttyd` (read-only reference)

## Architecture

Two halves that meet at a generated header:

```
Browser (html/)                    Backend (src/)
  xterm.js + preact  <-- WebSocket -->  libwebsockets + libuv --> PTY --> command
     |                                        ^
     | yarn build                             | #include
     v                                        |
  inlined bundle  ------------------>  src/html.h (GENERATED)
```

### Backend (`src/`, C99)
- `server.c` — entry point, CLI option parsing, libwebsockets context + libuv event loop, HTTP/WS protocol registration. `struct server` (see `server.h`) holds all global config.
- `protocol.c` — WebSocket callback: auth handshake, client/server message dispatch, JSON preferences.
- `pty.c` / `pty.h` — cross-platform PTY spawn + I/O (libuv). Windows uses ConPTY, POSIX uses forkpty.
- `http.c` — static file serving (index, token endpoint) from the embedded bundle.
- `utils.c` / `utils.h` — helpers (base64, string, signal name lookup).
- `compat.h` — platform shims.
- `server.h` — shared structs (`pss_tty`, `pss_http`, `server`) and the protocol message constants.

**WebSocket message protocol** (single-char prefix on each frame, defined in `server.h`):
- Client → server: `INPUT '0'`, `RESIZE_TERMINAL '1'`, `PAUSE '2'`, `RESUME '3'`, `JSON_DATA '{'`
- Server → client: `OUTPUT '0'`, `SET_WINDOW_TITLE '1'`, `SET_PREFERENCES '2'`

### Frontend (`html/`, TypeScript)
- `src/components/terminal/` — xterm.js wrapper, WebGL/canvas renderers, addons (`overlay.ts`, `zmodem.ts` for file transfer).
- `src/components/app.tsx`, `modal/` — preact UI.
- Built with webpack + gulp; the inlined, gzipped result is written to **`src/html.h`**.

### `src/html.h` is GENERATED — never hand-edit it
It is the frontend bundle embedded as a C byte array (~16k lines). Change the UI in
`html/`, then run the frontend build to regenerate it. Treat it as a build artifact.

## Build & Run

### Backend (CMake)
Dependencies: `libwebsockets` (≥3.2.0, built with libuv support), `libuv`, `json-c`,
`zlib`, and optionally `OpenSSL` or `Mbed TLS` for SSL.

```bash
cmake -B build            # add -DCMAKE_BUILD_TYPE=Release for release
cmake --build build
./build/ttyd bash         # run: serves http://localhost:7681
```

### Windows (MSVC + vcpkg)
```pwsh
vcpkg install --triplet x64-windows-static libwebsockets libuv json-c zlib openssl getopt-win32
cmake -S . -B build -G "Visual Studio 17 2022" -DCMAKE_TOOLCHAIN_FILE="$env:VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake" -DVCPKG_TARGET_TRIPLET=x64-windows-static
cmake --build build --config Release
```

### Cross-compilation (static binaries)
`BUILD_TARGET=aarch64 ./scripts/cross-build.sh` — targets: i686, x86_64, arm, armhf, aarch64, mips*, ppc64*, s390x, win32.

### Frontend (`html/`, yarn v2 / Berry)
```bash
cd html
yarn install
yarn run start   # dev server (also run `ttyd bash` in another terminal)
yarn run build   # production build → regenerates ../src/html.h
yarn run check   # gts lint
yarn run fix     # gts autofix
```

## Conventions
- **C style**: enforced by `.clang-format` (2-space indent). Format touched files before committing.
- **Frontend style**: `gts` (Google TS Style) — run `yarn run check` before committing frontend changes.
- **Version**: single source of truth is `project(ttyd VERSION x.y.z)` in `CMakeLists.txt` (currently 1.7.7). The build appends the git commit automatically.
- **Keep files small and single-purpose**; prefer editing existing files over adding parallel "enhanced" copies.

## Workflow for changes
1. Sync `main` with upstream when useful: `git fetch upstream && git rebase upstream/main`.
2. Branch off `main`: `git checkout -b fix/<slug>` or `feat/<slug>`.
3. If you touched `html/`, run `yarn run build` so `src/html.h` reflects the change — commit the regenerated header together with the source.
4. Build the backend and verify it runs before committing.
5. Conventional-commit messages (`fix:`, `feat:`, `refactor:`, …). No AI references in commit messages.
6. Push to `origin` and open the PR against `lamngockhuong/ttyd`.

## CI (`.github/workflows/`)
- `backend.yml` — MSVC build + cross-compile matrix (triggered by changes under `src/`, `CMakeLists.txt`, `scripts/`).
- `frontend.yml` — frontend lint/build.
- `docker.yml`, `release.yml` — image + release artifacts.
