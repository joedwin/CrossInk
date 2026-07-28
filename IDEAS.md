# Jes's CrossInk Ideas Backlog

Personal fork: [joedwin/CrossInk](https://github.com/joedwin/CrossInk) · Device: **Xteink X3** · Upstream: [uxjulia/CrossInk](https://github.com/uxjulia/CrossInk)

Running list of customization ideas. Add freely; anything here can be picked up in a Claude Code session.
Status: 💡 idea · 🔬 scoping · 🚧 in progress · ✅ shipped · ❄️ parked

## STATUS — handoff (2026-07-27, Mac session)

**Done:**
- ✅ Fork created under `joedwin`, local clone at `~/Documents/code projects/crossink/CrossInk`, remotes: `origin` = joedwin/CrossInk, `upstream` = uxjulia/CrossInk. Working branch: `development`.
- ✅ Submodules initialized (`freeink-sdk`); Mac toolchain installed: PlatformIO 6.1.19, SDL2, cmake; gh CLI authed as `joedwin`.
- ⚠️ Old stray fork `joe-16396/CrossInk` (wrong account) still exists — delete via its repo Settings page in a browser when convenient.

**In flight — simulator build broken on this Mac (next task):**
- `pio run -e simulator-X3` fails: `lib/Epub/Epub/css/CssParser.cpp` `tryParseNumber<float>` calls floating-point `std::from_chars`, which Apple clang 17 / CommandLineTools libc++ has **deleted** (integral overloads only). Hardware builds (`-e default`, RISC-V GCC) are unaffected.
- Fix approach (partially applied in the Mac working tree, **uncommitted**): guard with `#if !defined(__cpp_lib_to_chars)` + `if constexpr (std::is_floating_point_v<T>)` and fall back to `strtof` on a bounded null-terminated copy (string_view isn't null-terminated; reject leading whitespace and require full consumption to match from_chars semantics).
- **Known remaining bug in that partial fix:** the `std::from_chars` call must move into the `else` branch of the `if constexpr` — statements after an `if constexpr` are still instantiated for every `T`, so leaving it after the block re-triggers the same error.
- Verify with `pio run -e simulator-X3`, then run `.pio/build/simulator-X3/program` (put an EPUB in `./fs_/books/` first).
- Once green, consider offering the fix upstream (`fix/` branch → PR to uxjulia) since it's portable and hardware-neutral.

## Now

- 🚧 **Dev loop bring-up** — fix the CssParser build error above, build `simulator-X3`, run it, confirm the X3-sized window works.
- 💡 **Flash CrossInk 1.4.1 to the X3** — via web installer (inky.crossink.dev → Flash Tools), SD-card update, or `pio run -e default -t upload`. Note: some third-party-store X3s are USB-locked → use SD-card method or the Xteink Unlocker.

## Fonts & typography

- 💡 **Personal font pack** — convert favorite TTF/OTF faces (e.g. Atkinson Hyperlegible, iA Writer Duo) to `.cpfont`, install to `/.fonts/` on SD. See `docs/sd-card-fonts.md` and the repo `custom-fonts` skill.
- 💡 **Tune default font + size** — pick the default reading font/size baked into the build.

## Reading experience

- 💡 **Bionic Reading intensity setting** — light/medium/heavy bolding instead of one fixed mode.
- 💡 **Guide Dots variants** — alternate guide styles (line highlight, underline sweep).
- 💡 **Per-book layout memory** — remember font/size/indents per title instead of globally.
- 💡 **RSVP speed-reading mode** — one word at a time at a set WPM (ambitious; reuse Auto Page Turn timing).

## UI & screens

- 💡 **Custom sleep screens** — book-cover text, quote of the day, progress bar templates.
- 💡 **Reading-goal widget** — daily/weekly minutes goal on the Dashboard theme, from existing stats.
- 💡 **Button-map profiles** — swappable layouts (left-handed, one-button) beyond current remapping.

## Data & sync

- 💡 **Highlights/annotations export** — dump bookmarks/notes to a text file on SD.
- 💡 **Stats sync target** — the KOReader sync protocol is already supported; explore syncing stats somewhere I can see them off-device.

## Ambitious / someday

- ❄️ **Offline dictionary lookup** — see `docs/dictionary-development.md`; RAM-constrained (~380 KB usable).
- ❄️ **Tilt gestures beyond page turn** — shake-to-bookmark etc., building on the gravity-sensor code.

## Notes

- X3 and X4 share one firmware binary (`env:default`, device type `x3-x4`); the X3's 528×792 screen and tilt sensor are runtime-detected. Always sanity-check layouts in `simulator-X3`. Sleep-screen images for X3 must be 528×792 BMPs.
- ESP32-C3: ~380 KB usable RAM, no PSRAM. Stability beats features (see `AGENTS.md`).
- Keep `development` synced with upstream; each idea gets a `feat/…` branch.
