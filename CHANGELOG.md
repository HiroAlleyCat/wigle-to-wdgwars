# Changelog

All notable changes to wigle-to-wdgwars are documented here. Format
follows [Keep a Changelog](https://keepachangelog.com/) and the
project uses [Semantic Versioning](https://semver.org/).

## [1.1.1] - 2026-05-29 - Fix install path for users without git

v1.1.0 introduced a `requirements.txt` with the gungnir dependency
pinned via `gungnir @ git+https://github.com/...@v0.1.1`. That URL
form forces pip to shell out to `git clone`, which fails on any
machine without git on PATH (a common state for casual ZIP
downloaders, especially on Windows).

The README also still claimed "There's nothing to install. The script
is a single file, depends only on Python 3.10+, and uses `urllib`
from stdlib — no `pip install`." That was true in v1.0 but became
false the moment gungnir was added. New ZIP-download users following
the README hit `ModuleNotFoundError: No module named 'gungnir'`.

### Fixed

- **`requirements.txt` no longer requires `git` on the user's PATH.**
  The gungnir pin switched from
  `gungnir @ git+https://github.com/HiroAlleyCat/gungnir@v0.1.1`
  to
  `gungnir @ https://github.com/HiroAlleyCat/gungnir/archive/refs/tags/v0.1.1.tar.gz`
  — pip fetches the tarball over plain HTTPS with stdlib `urllib`.
  Same v0.1.1 tag, same gungnir bytes, same reproducibility.

### Documentation

- README's tagline and **Installing** section now correctly state that
  this is a one-dependency tool (gungnir), document `pip install -r
  requirements.txt`, and offer a no-git ZIP-download path alongside
  `git clone`.

- New **Updating** subsection explaining that release bumps may move
  the gungnir pin, so `git pull` (or re-extracting the ZIP) must be
  paired with `pip install --upgrade -r requirements.txt`.

## [1.1.0] — extract signed-JSON transport to gungnir

Structural refactor. **Multipart CSV upload (`/api/upload-csv`) and the
WiGLE-side fetch are unchanged** — they stay as local code because
gungnir doesn't handle multipart and doesn't speak the WiGLE API. The
**signed-JSON path (`/api/upload/`)** moves to
[gungnir](https://github.com/HiroAlleyCat/gungnir) 0.1.1, the same
library Muninn v2.0 sits on.

### Changed

- **`upload_aircraft_json()` default `batch` lowered 1000 → 500** per
  locosp's recommendation (100-500 per request). The CLI flag
  `--aircraft-batch` default follows suit.
- **`whoami()` output moved from stdout to structured stderr logs.**
  v1.0 printed raw JSON; v1.1 emits a `key OK — user=X` line plus a
  `wifi=… ble=… aircraft=… total=…` summary line. Cron jobs that
  parsed the raw JSON need to update.
- **`_post_signed()` is now a thin shim over `gungnir.transport
  .send_chunk()`**. Status return is `200` on success or `1` on any
  failure — the exact HTTP code for non-2xx is no longer surfaced
  (gungnir handles 5xx with retry + backoff internally).

### Added

- `__version__ = "1.1.0"` exposed at module top.
- New `requirements.txt` pinning gungnir to v0.1.1.

### Improved (free wins from gungnir 0.1.x)

- **Retry 5xx + network errors** with exponential backoff (3 attempts,
  2s/4s). v1.0 failed the upload on the first transient hiccup.
- **429 stops the batch and persists a cooldown.** v1.0 already had a
  cooldown file (the pattern gungnir borrowed) but kept POSTing after
  a 429 until the chunk loop finished. v1.1 stops immediately.
- **Silent-drop detection.** HTTP 200 ok:true with every counter zero
  now returns rc=1. v1.0 had no detection at all.
- **Inter-chunk cooldown of 1s** (was 2s hard-coded between aircraft
  chunks; the new default is shorter but configurable via the gungnir
  Client).
- **User-Agent now includes the repo URL** per RFC bot-UA convention:
  `wigle-to-wdgwars/1.1.0 (+https://github.com/HiroAlleyCat/wigle-to-wdgwars)`.

### Compatibility

- **API key file unchanged.** Local `load_key()` still reads from
  `~/.config/wigle-to-wdgwars/wdgwars.key`. The decision to keep this
  path *outside* gungnir's per-OS convention is deliberate — Windows
  users of v1.0 stored the key under `~/.config/` (Linux-style) and
  we don't want to break their install.
- **`cooldown.json` and `hwm.json` paths** now follow gungnir's per-OS
  convention. On POSIX this is byte-identical to v1.0
  (`~/.config/wigle-to-wdgwars/cooldown.json`). On Windows the files
  move to `%APPDATA%/wigle-to-wdgwars/`. Both files are ephemeral so
  the migration is harmless.
- **Wire-protocol unchanged.** gungnir's envelope is byte-identical to
  the v1.0 hand-rolled envelope (verified by gungnir's parity test
  against Muninn v1.11.1, which used the same code).

### Migration

```
pip install -r requirements.txt  # pulls gungnir from the pinned tag
```

No config-file changes. Cron stanzas keep working.

## [1.0.0] — initial release

### Added
- CLI tool to upload WiGLE 1.6 CSVs to the WDGoWars wardriving
  leaderboard. Supports both the multipart `/api/upload-csv` (for
  Wi-Fi + BLE observations) and the signed `/api/upload/` (for
  aircraft records).
- `--from-wigle` mode: pull your latest WiGLE upload via the WiGLE
  API and forward it to WDGoWars without staging a local file.
- API-key resolution via `--key`, `$WDGWARS_API_KEY`, or
  `~/.config/wigle-to-wdgwars/wdgwars.key`.
- Cooldown persistence (`cooldown.json`) so 429 deadlines survive
  cron restarts.
- HWM tracking (`hwm.json`) for external monitoring.
- Chunked CSV upload to dodge the Cloudflare 524 timeout on large
  imports.
