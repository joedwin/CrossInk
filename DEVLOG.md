# CrossInk Devlog

Dated log of what changed and why — the narrative companion to `TINKER.md`'s
load table and `CHANGELOG.md`'s user-facing entries. Newest day first. Every
session that changes the repo should add or extend a day entry here.

## 2026-07-28

**Set up the tinker pipeline (Loads 0–1).**
- Turned this fork into a self-serve firmware playground: `TINKER.md` playbook
  (the ask → build → flash loop, load history, anti-brick notes) and
  `.github/workflows/tinker-build.yml`, which builds `env:default` on every
  push to `main`/`claude/**`/`tinker/**` and publishes the `.bin` as a
  `load-<n>` GitHub release. No repo secrets needed.
- Tamed the Claude cloud sandbox so firmware builds work in-session
  (PlatformIO registry is proxy-blocked; workarounds recorded in
  `.claude/CONTEXT.md`). Local `pio run -e default` now takes ~5 min.
- Load 1 = stock v1.4.0, proving the pipeline end to end.

**Load 2: EB Garamond + font-size buttons.**
- Added EB Garamond (SIL OFL, from Google Fonts' variable font, statics
  instantiated at wght 400/700) as a third built-in reading font. Generated
  all 8 sizes × 4 styles for both `builtinFonts/` and `noemoji/` variants via
  `convert-builtin-fonts.sh`; wired through `fontIds.h`, `all.h`, `main.cpp`
  registration, `FONT_FAMILY` enum + mappings in `CrossPointSettings`,
  Font Selection UI, and i18n.
- Added `Font Size Up` / `Font Size Down` reader shortcut actions
  (`SHORT_PWRBTN` 22/23, `LONG_PRESS_MENU_ACTION` 21/22 — appended, never
  renumbered, since raw values are persisted). They reuse the existing
  `sdFontSystem.changeReaderFontSize()` + `reindexCurrentSection()` path and
  are assignable to short/long Power, long-press Menu, and long-press Back in
  Settings → Controls.
- **Flash reality check:** a built-in font family costs ~1.05 MB (glyph data
  incl. emoji/symbol/CJK fallbacks), not the ~0.5 MB first estimated. After
  Load 2 the app partition is 99.7% full (6,208 bytes free). Next load must
  free space (trim a family's fallbacks, or drop one family) before adding
  anything else.
- Introduced pre-commit review agents (security pass + breakage pass) as a
  standing workflow step; first run performed retroactively on Load 2's diff.
