# CrossPoint Reader — Durable Context

Keep this file focused on repo-specific gotchas that are worth reusing in future sessions.

## Tinker Workflow

- This fork is a personal tinker playground; the workflow (loads, flashing, safety) is in `TINKER.md`. Log every firmware-affecting change as a new load there.

## Building In Claude Cloud Sessions

- `git submodule update --init --recursive` first; the fresh clone has `freeink-sdk` unpopulated.
- `pip install platformio` (PyPI 6.1.19). The pioarduino-core GitHub *archive* URL is 403-blocked by the egress proxy; GitHub release *downloads* are allowed.
- The pioarduino platform's penv bootstrap fails on that same archive URL. Fix: pre-install `platformio==6.1.19` plus the `python_deps` list from `~/.platformio/platforms/espressif32/builder/penv_setup.py` into the penv via `uv pip install --python=/root/.platformio/penv/bin/python ...` so the blocked URL is skipped.
- Python `requests` ignores `SSL_CERT_FILE`; framework downloads fail TLS until the proxy CA is appended to certifi: `cat /root/.ccr/ca-bundle.crt >> $(python3 -c 'import certifi; print(certifi.where())')` (and the penv's certifi).

## Simulator

- Simulator patches belong in the adjacent `crosspoint-simulator` repo.
- The valid local simulator env in this repo is `simulator`, and `pio run -e simulator` currently builds cleanly.
- The simulator `PNGdec` stub in `crosspoint-simulator/src/PNGdec.h` needs to mirror the real API shape used by app code, including `hasAlpha()` and `getTransparentColor()`, even though decode still fails intentionally.
- Known simulator limits:
  - No image rendering: `platformio.ini` ignores `hal`, `PNGdec`, and `JPEGDEC`, so image decoders are intentionally absent.
  - JPEGDEC stub always fails; `JPEGDEC fallback: open failed (err=-1)` is expected in simulator.
  - `esp_deep_sleep_start()` is a no-op in simulator.
  - `HalStorage` uses POSIX file access under `./fs_` and allows multiple readers, unlike real hardware.

## Real Hardware / Storage

- SdFat on hardware allows only one open reader per file path at a time. If a fallback needs to reopen the same file, close the first handle before reopening.

## Rendering / Reader Pipeline

- `lib/Epub/Epub/Page.cpp`: images must render only in `GfxRenderer::BW`; grayscale passes are text anti-aliasing passes only.
- Kindle EPUBs may contain paired high-res and old-Kindle fallback images. `ChapterHtmlSlimParser` should skip `<img>` nodes with `data-AmznRemoved-M8` to avoid duplicate stacked images.
- After image/layout pipeline changes that affect cached EPUB output, clear the affected `.crosspoint/epub_<hash>/` cache if behavior looks stale.

## Misc Repo Gotchas

- POSIX TZ signs are inverted from ISO 8601 in `TimeStore::applyTimezone()`: `"UTC-1"` means UTC+1.
- `LyraTheme::drawHeader()` does not call `BaseTheme::drawHeader()`, so header changes in the base theme must be duplicated in Lyra if needed.
