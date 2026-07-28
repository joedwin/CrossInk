# Jes's CrossInk Ideas Backlog

Personal fork: [joe-16396/CrossInk](https://github.com/joe-16396/CrossInk) · Device: **Xteink X3** · Upstream: [uxjulia/CrossInk](https://github.com/uxjulia/CrossInk)

Running list of customization ideas. Add freely; anything here can be picked up in a Claude Code session.
Status: 💡 idea · 🔬 scoping · 🚧 in progress · ✅ shipped · ❄️ parked

## Now

- 💡 **Dev loop bring-up** — build `simulator-X3` env, run it, confirm the X3-sized window works on this Mac.
- 💡 **Flash CrossInk 1.4.1 to the X3** — via web installer or `pio run -e default -t upload`.

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
