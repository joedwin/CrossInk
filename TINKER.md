# CrossInk Tinker Workflow

This fork is a personal playground for the Xteink X4 e-reader. The goal: making a
firmware change should be as easy as opening a Claude session and saying
"new load: change X".

## The Loop

1. **Ask.** Open a Claude Code session on this repo and describe the change
   ("make the font size change with the side buttons", "add a boot animation", ...).
2. **Claude builds it.** Claude edits the code, validates with `pio run -e default`
   (and `pio run -e simulator` for UI work), logs the load in the table below
   and the day's work in `DEVLOG.md`, commits, and pushes.
   **Before every commit that touches firmware code**, run two review agents in
   parallel and act on their findings:
   - a **security pass** (nothing malicious or unrelated smuggled in; no
     unsafe indexing of persisted values; no unexpected changes to
     network/OTA/flash-write paths), and
   - a **breakage pass** (persisted enum values only ever appended; all build
     variants still link; switch statements handle new values; RAM/flash
     budgets respected — check the build's `RAM:`/`Flash:` summary and the
     OTA-partition line).
3. **CI produces the firmware.** Every push to `main` or a `claude/**` /
   `tinker/**` branch triggers `.github/workflows/tinker-build.yml`, which builds
   the `default` environment and publishes a **`load-<n>` release** on this
   repo's GitHub Releases page with the `firmware-*.bin` attached.
4. **Flash it.** From any computer with Chrome/Edge:
   - Download the `.bin` from the release.
   - Plug the X4 in via USB-C and wake/unlock it.
   - Go to <https://crosspointreader.com/#flash-tools>, pick the device,
     choose **Custom .bin**, select the file, click **Flash**.
   - CLI alternative: `esptool.py --chip esp32c3 --port <port> --baud 921600 write_flash 0x10000 firmware.bin`

## Can I brick it?

Practically no, if we stick to this flow. The flash at address `0x10000` only
replaces the **application**; the ESP32-C3's first-stage bootloader is in mask
ROM and cannot be overwritten this way. If a load is broken (boot loop, frozen
screen, whatever):

1. Plug in via USB-C — the ROM download mode always works, even with a
   completely broken app.
2. Reflash either a known-good load from this repo's releases, or the official
   firmware from <https://crosspointreader.com/#flash-tools>.

Rules that keep it that way:

- Never flash to offsets other than `0x10000`, and never touch the bootloader
  or partition table unless we very deliberately decide to.
- Prefer testing risky changes behind a setting or on a branch load first.
- Reading data (books, progress) lives on the SD card and survives reflashes.

## Load History

| Load | Date | Commit | What changed |
|------|------|--------|--------------|
| 0 | 2026-07-28 | `b7f6708` | Baseline: stock CrossInk v1.4.0 |
| [1](https://github.com/joedwin/CrossInk/releases/tag/load-1) | 2026-07-28 | `454d2e9` | Stock v1.4.0 firmware, first CI-built load (workflow added, no firmware changes) |
| [2](https://github.com/joedwin/CrossInk/releases/tag/load-2) | 2026-07-28 | — | EB Garamond built-in font; Font Size Up/Down shortcut actions for Power/Menu/Back buttons |

## Idea Backlog

- Font size up/down mapped to buttons (per-book or global).
- Custom sleep-screen art / boot splash.
- Page-turn animation experiments (within e-ink refresh limits).
- On-device stats screen tweaks.
- **Render previews without hardware**: run the `simulator` build headless in a
  Claude session, capture the 800x480 framebuffer as an image, and share it in
  chat or as a small web page viewable on mobile/desktop — "here's how this load
  renders" before flashing anything. Could grow into a per-load screenshot
  gallery attached to each release.

## Flash Budget

The app must fit the 6,553,600-byte OTA partition (`check_firmware_size` fails
the build otherwise). As of Load 2 we're at **99.7% (6,208 bytes free)** — each
built-in font family costs ~1.05 MB. Before adding anything, free space first
(trim a font family's fallback glyphs or drop a family).

## Notes for Claude sessions

- **Over-engineering meter**: occasionally self-assess (a quick X/10 with a one-liner)
  how over-engineered the current approach is, especially before adding new
  infrastructure. Joe finds this fun and it keeps the project honest — this is
  a tinker repo, not a product.
- **Subagents welcome**: use subagents liberally where they genuinely help
  (parallel exploration, big searches, independent verification) and say what
  was delegated and why.
- Build validation: `pio run -e default` (firmware), `pio run -e simulator` (UI logic).
- Bump the load number and add a row to **Load History** with every firmware-affecting push.
- The on-device OTA updater currently points at `uxjulia/CrossInk` releases
  (`src/network/OtaUpdater.cpp`, `CROSSINK_OTA_RELEASE_URL`). If we ever want
  cable-free updates from this fork, repoint it here and match the
  `firmware-tiny-*.bin` asset naming.
- Cloud build environment gotchas are recorded in `.claude/CONTEXT.md`.
