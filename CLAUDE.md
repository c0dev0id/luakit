# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test

Build system is a plain Makefile (BSD users: use `gmake`). Dependencies are detected via `pkg-config`.

```sh
make                                  # build luakit + luakit.so + apidoc + man page
make DEVELOPMENT_PATHS=1               # build for running from source dir (sets -ggdb, loads ./lib + ./config)
make USE_LUAJIT=0                      # build against Lua 5.1 instead of LuaJIT (default is LuaJIT)
make clean
make run-tests                         # full test suite (needs xvfb + luacheck + luassert)
make PREFIX=/usr install
```

Run a single test file by passing its path to the test runner:
```sh
./tests/run_test.lua tests/async/test_basic.lua
./tests/run_test.lua tests/style/test_luacheck.lua
```

CI runs across Ubuntu/Debian/Arch/Alpine/FreeBSD/OpenBSD; lint job runs `cppcheck` (`--enable=warning,performance,portability` over `*.c *.h common/ clib/ widgets/ extension/`) and `shellcheck`. The matching local invocations live in `.github/workflows/`.

After modifying generated/cached headers (`buildopts.h`, `common/tokenize.[ch]`), or after changing `PREFIX`, run `make clean` before rebuilding.

## Architecture

Luakit is a **two-process** browser: a UI process (`luakit`) hosts GTK and Lua, while a WebKit **web extension** (`luakit.so`) is dlopen'd into each WebKit web process to give Lua access to the DOM. The same source tree builds both.

```
*.c, clib/, widgets/, common/[clib/]   →   luakit       (UI process, links GTK+WebKitGTK)
extension/, extension/clib/, common/   →   luakit.so    (web extension, -DLUAKIT_WEB_EXTENSION -fPIC -shared)
```

Each side has its own `common_t common` and its own Lua state. Files under `common/` must compile cleanly into both targets and **must not reference `globalconf`** (enforced by `tests/style/test_common.lua`).

### IPC

The two processes exchange messages over a Unix socket. Message types live in the `IPC_TYPES` X-macro in `common/ipc.h`; adding one means extending that macro plus matching `ipc_recv_<name>` handlers on each side. `common/clib/ipc.h` exposes the `ipc_channel` Lua class used for module-to-module IPC.

### Lua module layout

- `clib/` — UI-side C → Lua bindings (download, request, soup, sqlite3, stylesheet, unique, widget, xdg, web_module, msg). Registered from `luah.c`.
- `extension/clib/` — Web-side C → Lua bindings (dom_document, dom_element, page, soup, msg). Registered from `extension/extension.c`.
- `common/clib/` — Bindings available in **both** Lua states (ipc, timer, regex, utf8, luakit).
- `widgets/` — GTK widget wrappers (`box`, `entry`, `notebook`, `overlay`, `stack`, `webview`, `window`, …). `widgets/webview/` contains WebView feature splits (auth, downloads, history, inspector, javascript, scroll, stylesheets, find_controller).
- `lib/lousy/` — Core Lua utility library: `bind`, `mode`, `signal`, `theme`, `util`, `uri`, `widget`, `pickle`, `load`. Used by everything above.
- `lib/` — User-facing built-in modules. Pure Lua. **Files named `*_wm.lua` run in the web extension process** (loaded via `require_web_module("name_wm")`), so they see `dom_document`/`dom_element`/`page` instead of UI globals like `widget`/`sqlite3`. The `lib.test_luacheck` test enforces this split via per-file globals.
- `config/rc.lua`, `config/theme.lua` — Default user config, installed to `XDGPREFIX/luakit/`. User overrides go in `~/.config/luakit/userconf.lua` (or a full `rc.lua` copy).

### Generated sources

`common/tokenize.list` is compiled by `build-utils/gentokens.lua` into `common/tokenize.[ch]`, producing interned C strings (`L_TK_*`) plus a `lua_class_t`-style hash lookup used across the C code. Add new tokens by editing the list — the Makefile regenerates both files.

`buildopts.h` is generated from `buildopts.h.in` and bakes the install paths (`LUAKIT_INSTALL_PATH`, `LUAKIT_CONFIG_PATH`, etc.) into the binary based on `PREFIX`.

### Modes, binds, settings

Bindings are managed by the `modes` module (`lib/modes.lua`), not `binds`. `binds.add_binds`/`add_cmds` are deprecated shims. The default modes (normal, insert, command, passthrough, lua, …) and their default keybinds live in `lib/binds.lua`, but extensions should call `modes.add_binds` / `modes.add_cmds` directly. The `settings` module (`lib/settings.lua`) provides the centralized, namespaced, optionally per-domain settings store; persisted values live in `$XDG_DATA_HOME/luakit/settings` (a `lousy.pickle` blob).

### Logging

C side: `debug()`, `verbose()`, `info()`, `warn()`, `fatal()` macros from `log.h`. Lua side: `msg.debug/verbose/info/warn/error`. Module-scoped verbosity from the CLI: `--log=lua/adblock=debug` or `--log=all=debug`.

## Source style (enforced by `tests/style/`)

- C files: header block must read `/* * <filename> - <short desc>\n * \n * Copyright © …`, contain the full GPL paragraph, end with `// vim: ft=c:et:sw=4:ts=8:sts=4:tw=80`, and headers need `LUAKIT_FILENAME` include guards (path with `/` and `.` replaced by `_`, uppercased).
- Lua files: start with `--- <summary line ending in a period>`, declare `-- @module <path matching filename>` (or `@submodule`), include `local _M = {}` after the header, document every `_M.foo`/`function _M.foo` export with a `--- ` block preceded by a blank line, end with `-- vim: et:sw=4:ts=8:sts=4:tw=80`.
- No tabs in indentation, no trailing whitespace, in any `.c`/`.h`/`.lua` file.
- `lib/markdown.lua` is the only exempt Lua file (vendored).

## Versioning

Version comes from `git describe` via `build-utils/getversion.sh`. Tagged releases follow `MAJOR.MINOR.PATCH`; untagged builds report `0.0.0-<sha>[-dirty]`.

## Repo-specific git workflow

Upstream PRs target the `develop` branch (per `.github/workflows/run-tests.yml`). The locally checked-out working branch (`develop-next`) is the personal integration branch — rebase it onto upstream `develop` (or local `main`/`master` if mirrored) before opening a PR.
