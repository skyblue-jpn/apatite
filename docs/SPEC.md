# Apatite SPEC (Base16 Theme Applier)

## Goal

Apatite is a Base16-based unified theme applier for Linux desktops (GTK2/3/4, Qt5/6 via qtct, KDE colorscheme, fontconfig, xsettingsd, icons/cursor). It reads a config, resolves a palette (preset/wallpaper/custom), renders templates, and writes config files with safe defaults and fallbacks.

## Scope (v0)

- Load configuration (TOML) and presets (TOML/YAML) with Base16 keys.
- Support three modes: `preset`, `wallpaper` (palette extraction stub initially), `custom` (Base16 direct).
- Preference (`dark`/`light`) selects which palette to apply.
- Render templates (`{{base00}}..{{base0F}}`) using tera.
- Targets: GTK2/3/4, Qt5/6 (qtct), KDE colorscheme, fontconfig, xsettingsd, icons/cursor.
- Fallback: per-variant (light/dark) failure → swap to default preset (One Dark/One Light). Log at INFO.
- Write outputs atomically; skip writes if no diff.
- CLI provides `apply`, `--config`, `--mode`, `--preference`, `--targets`, `--dry-run`, `--force`.

## Directory layout (relevant)

- `assets/config.toml` — sample config (Base16)
- `assets/presets/` — built-in presets (One Dark/One Light as `default`, plus popular Base16 YAML/TOML)
- `assets/templates/` — target templates (use `{{base0X}}`)
- `assets/*` — per-target templates (gtk2/3/4, qtct, kcolorscheme, fontconfig, xsettingsd, icons)
- `apatite-core/` — core library (to implement below)
- `cli/` — CLI frontend (to implement)
- `gui/` — GUI frontend (later)

## Config schema (summary)

- `version = 1`
- `[theme]`
  - `mode = "preset" | "wallpaper" | "custom"`
  - `preference = "dark" | "light"`
- `[theme.preset]`
  - `dark = "<name>"` (optional; fallback to default if missing)
  - `light = "<name>"`
- `[theme.wallpaper]`
  - `path = "<string>"`
- `[theme.custom.dark]` / `[theme.custom.light]`
  - `base00..base0F = "#RRGGBB"` (either side may be missing; fallback applies)
- Fonts/assets/targets: see `assets/config.toml` (fonts.\*; assets.icon/cursor; targets.gtk2/3/4, qt5/qt6, fontconfig, xsettingsd)

## Preset loading rules

- Search order: user `~/.config/apatite/presets/*.{toml,yaml,yml}` → built-in `assets/presets/**/*.{toml,yaml,yml}`.
- Extension decides parser (TOML or YAML). YAML structure:
  ```
  system: "base16"
  name: "<scheme>"
  author: "<author>"
  variant: "dark" | "light"
  palette:
    base00: "#..."
    ...
    base0F: "#..."
  ```
- TOML structure mirrors `theme.preset.dark/light.base00..base0F`.
- If variant missing for requested side, fallback to default preset for that side (log INFO).

## Palette selection & fallback

- Resolve mode:
  - `preset`: load named preset(s); merge light/dark with fallback.
  - `wallpaper`: generate Base16 (both variants) from wallpaper (stub initially; if fails → default).
  - `custom`: use provided Base16 blocks; missing side → default.
- After resolving DualPalette (light/dark), select one by `preference` and expose as flat `base00..base0F` to templates.
- Validation: any missing/invalid color in the selected variant triggers replacement of the whole variant with default; log INFO.

## Rendering

- Engine: tera.
- Context: `base00..base0F`, fonts._, assets.icon/cursor, targets._.
- Templates already use `{{base0X}}`; shadow is a fixed RGBA in templates.
- Output destinations follow XDG:
  - GTK2: `~/.gtkrc-2.0`
  - GTK3: `~/.config/gtk-3.0/`
  - GTK4: `~/.config/gtk-4.0/`
  - Qt5/6 (qtct): `~/.config/qt5ct/qt5ct.conf`, `~/.config/qt6ct/qt6ct.conf`
  - KDE colorscheme: `~/.local/share/color-schemes/apatite.colors`
  - fontconfig: `~/.config/fontconfig/fonts.conf`
  - xsettingsd: `~/.config/xsettingsd/xsettingsd.conf`
  - icons: `~/.icons/default/index.theme`
- Behavior: diff-only writes; atomic replace (write to tmp, rename).

## CLI (planned)

- `apatite apply [--config <path>] [--mode <preset|wallpaper|custom>] [--preference <dark|light>] [--targets gtk3,qt6,...] [--dry-run] [--force]`
- `--dry-run`: print rendered outputs to stdout (no write).
- `--targets`: comma/space list to limit output.
- Exit codes: nonzero on fatal errors (e.g., config unreadable), zero with INFO logs on fallback usage.

## Logging

- INFO for: using default preset due to missing/invalid variant; skipping write due to no diff.
- WARN/ERROR for: unreadable config/preset, template render failure, write failure.

## Testing (planned)

- Snapshot tests: config → rendered outputs → compare fixtures.
- Unit tests: preset loader (TOML/YAML), fallback logic, diff/atomic write helper (write to temp dir).

## License & notices

- Project license: see `LICENSE`.

## Open items (post-v0)

- Implement real `palette_from_wallpaper`.
- GUI wrapper.
- `--strict` mode to fail on fallback instead of auto defaulting.
- More granular target toggles and per-target paths for edge environments (Flatpak, etc.).
