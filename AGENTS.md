# AGENTS.md

NotGrid: party/raid frames addon for WoW Vanilla 1.12.1 (Turtle/OctoWoW). Plain Lua loaded by the game client — no build system, linter, or test suite. User-facing slash commands are documented in README.md.

## Environment & verification

- This repo IS the installed addon (`Interface/AddOns/NotGrid` inside the game client). Verify changes in-game via `/reload`; watch the error frame for Lua errors.
- The 1.12 client runs **Lua 5.0**. Do NOT validate with modern lua/luacheck/lua5.1+:
  - `for k,v in tbl do` without `pairs()` is valid here (options.lua, frames.lua) — don't "fix" it.
  - Frame scripts rely on implicit `this` (e.g. `this.unit` in OnClick handlers).
  - Modern tooling will flag correct code as errors; syntax-check only with a Lua 5.0-compatible parser, or not at all.

## Load order & architecture

- `notgrid.toc` defines load order; any new file/lib must be added there. Order matters: AceLibrary bootstrap + vendored libs first, then `localization` → `core` → `frames` → `mapsizes` → `proximity` → `options` → `menus`.
- Globals: `NotGrid` (Ace2 addon object), `NotGridOptions` (SavedVariablesPerCharacter). Flow: `core.lua` `OnInitialize` builds frames (`frames.lua`), `OnEnable` wires events and options (`self.o = NotGridOptions` only after saved vars load).
- `libs/` is vendored Ace2-era code registered through the global AceLibrary registry. Treat as frozen: no refactoring/modernizing. HealComm detects SuperWoW via `SetAutoloot`.

## Adding an option (spans multiple files)

1. `localization.lua`: add key under enUS with value `true` (enUS uses key-as-text; other locales like ruRU map keys to real strings).
2. `options.lua`: add a default to `DefaultOptions`.
3. `menus.lua`: add a menu entry referencing `L["key"]` (+ tooltip key if needed).
- If the change breaks previously saved settings, bump `DefaultOptions["version"]` (it tracks commits) and add a migration block keyed on the old version in `NotGrid:SetDefaultOptions()` (options.lua).

## Conventions

- Tabs for indentation.
- Default fonts point at client-installed files (`Fonts\custom\...`), not stock Blizzard paths — missing fonts fail silently to defaults on other installs.
