![backend](https://github.com/lamngockhuong/ttyd/actions/workflows/backend.yml/badge.svg)
![frontend](https://github.com/lamngockhuong/ttyd/actions/workflows/frontend.yml/badge.svg)
[![GitHub Releases](https://img.shields.io/github/downloads/lamngockhuong/ttyd/total)](https://github.com/lamngockhuong/ttyd/releases)
![GitHub](https://img.shields.io/github/license/lamngockhuong/ttyd)

# ttyd - Share your terminal over the web

ttyd is a simple command-line tool for sharing terminal over the web.

![screenshot](https://github.com/lamngockhuong/ttyd/raw/main/screenshot.gif)

> [!NOTE]
> **This is a maintained fork of [`tsl0922/ttyd`](https://github.com/tsl0922/ttyd).**
> The original project is no longer actively maintained and has open, unfixed bugs.
> This fork continues development and bug fixes. Issues and pull requests are welcome
> at [`lamngockhuong/ttyd`](https://github.com/lamngockhuong/ttyd).
> All credit for the original work goes to [Shuanglei Tao](https://github.com/tsl0922)
> and the upstream contributors.

# Features

- Built on top of [libuv](https://libuv.org) and [WebGL2](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) for speed
- Fully-featured terminal with [CJK](https://en.wikipedia.org/wiki/CJK_characters) and IME support
- [ZMODEM](https://en.wikipedia.org/wiki/ZMODEM) ([lrzsz](https://ohse.de/uwe/software/lrzsz.html)) / [trzsz](https://trzsz.github.io) file transfer support
- [Sixel](https://en.wikipedia.org/wiki/Sixel) image output support ([img2sixel](https://saitoha.github.io/libsixel) / [lsix](https://github.com/hackerb9/lsix))
- SSL support based on [OpenSSL](https://www.openssl.org) / [Mbed TLS](https://github.com/Mbed-TLS/mbedtls)
- Run any custom command with options
- Basic authentication support and many other custom options
- Cross platform: macOS, Linux, FreeBSD/OpenBSD, [OpenWrt](https://openwrt.org), Windows

# Installation

> [!IMPORTANT]
> The package managers below (Homebrew, apt, WinGet, Scoop, …) install the **upstream**
> `tsl0922/ttyd` build. This fork's releases are tagged `<upstream-version>-fork.N`
> (e.g. `1.7.7-fork.1`) and exist mainly to ship a **working Windows binary** — the
> upstream Windows release is currently broken. Get them from this fork's
> [releases](https://github.com/lamngockhuong/ttyd/releases) page or
> [build from source](#build-from-source).

## Install on macOS

- Install with [Homebrew](http://brew.sh): `brew install ttyd`
- Install with [MacPorts](https://www.macports.org): `sudo port install ttyd`

## Install on Linux

- Install on Debian/Ubuntu: `sudo apt install ttyd`
- Install the snap: `sudo snap install ttyd --classic`
- Install on OpenWrt: `opkg install ttyd`
- Install on Gentoo: clone the [repo](https://bitbucket.org/mgpagano/ttyd/src/master) and follow the directions [here](https://wiki.gentoo.org/wiki/Custom_repository#Creating_a_local_repository).
- Install with [Homebrew](https://docs.brew.sh/Homebrew-on-Linux) : `brew install ttyd`
- Precompiled static binaries: download from the [releases](https://github.com/lamngockhuong/ttyd/releases) page

## Install on Windows

The upstream Windows binary is broken; this fork publishes a working one. Download it
from this fork's [releases](https://github.com/lamngockhuong/ttyd/releases) (tagged like
`1.7.7-fork.1`) and verify it against `SHA256SUMS` in the same release:

- **`ttyd.msvc.exe`** — 64-bit build, native MSVC (recommended)
- **`ttyd.win32.exe`** — 32-bit build

Package managers install the upstream build instead:

- [WinGet](https://github.com/microsoft/winget-cli): `winget install tsl0922.ttyd` (upstream)
- [Scoop](https://scoop.sh/#/apps?q=ttyd&s=2&d=1&o=true): `scoop install ttyd` (upstream)

Or [build from source](#build-from-source) / [compile with MSVC](https://github.com/tsl0922/ttyd/wiki/Compile-on-Windows).

## Build from source

Dependencies: `libwebsockets` (≥ 3.2.0, built with libuv support), `libuv`, `json-c`,
`zlib`, and optionally `OpenSSL` / `Mbed TLS` for SSL.

```bash
git clone https://github.com/lamngockhuong/ttyd.git
cd ttyd
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/ttyd bash
```

See [`CLAUDE.md`](CLAUDE.md) for the full build matrix (MSVC/vcpkg, cross-compilation)
and frontend build instructions.

# Usage

## Command-line Options

```
USAGE:
    ttyd [options] <command> [<arguments...>]

OPTIONS:
    -p, --port              Port to listen (default: 7681, use `0` for random port)
    -i, --interface         Network interface to bind (eg: eth0), or UNIX domain socket path (eg: /var/run/ttyd.sock)
    -U, --socket-owner      User owner of the UNIX domain socket file, when enabled (eg: user:group)
    -c, --credential        Credential for basic authentication (format: username:password)
    -H, --auth-header       HTTP Header name for auth proxy, this will configure ttyd to let a HTTP reverse proxy handle authentication
    -u, --uid               User id to run with
    -g, --gid               Group id to run with
    -s, --signal            Signal to send to the command when exit it (default: 1, SIGHUP)
    -w, --cwd               Working directory to be set for the child program
    -a, --url-arg           Allow client to send command line arguments in URL (eg: http://localhost:7681?arg=foo&arg=bar)
    -W, --writable          Allow clients to write to the TTY (readonly by default)
    -t, --client-option     Send option to client (format: key=value), repeat to add more options
    -T, --terminal-type     Terminal type to report, default: xterm-256color
    -O, --check-origin      Do not allow websocket connection from different origin
    -m, --max-clients       Maximum clients to support (default: 0, no limit)
    -o, --once              Accept only one client and exit on disconnection
    -q, --exit-no-conn      Exit on all clients disconnection
    -B, --browser           Open terminal with the default system browser
    -I, --index             Custom index.html path
    -b, --base-path         Expected base path for requests coming from a reverse proxy (eg: /mounted/here, max length: 128)
    -P, --ping-interval     Websocket ping interval(sec) (default: 5)
    -6, --ipv6              Enable IPv6 support
    -S, --ssl               Enable SSL
    -C, --ssl-cert          SSL certificate file path
    -K, --ssl-key           SSL key file path
    -A, --ssl-ca            SSL CA file path for client certificate verification
    -d, --debug             Set log level (default: 7)
    -v, --version           Print the version and exit
    -h, --help              Print this text and exit
```

Read the example usage on the [wiki](https://github.com/tsl0922/ttyd/wiki/Example-Usage).

## Browser Support

Modern browsers, See [Browser Support](https://github.com/xtermjs/xterm.js#browser-support).

## Alternatives

* [Wetty](https://github.com/krishnasrinivas/wetty): [Node](https://nodejs.org) based web terminal (SSH/login)
* [GoTTY](https://github.com/yudai/gotty): [Go](https://golang.org) based web terminal
