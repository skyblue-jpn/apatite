# Apatite

Unified theme applier for Linux desktops (GTK/Qt/KDE/fontconfig/xsettingsd) based on **Base16** palettes.
CLI/GUI are planned; templates and presets are ready.

## Features

- Base16 を正とした配色生成（light/dark 両対応）
- GTK2/GTK3/GTK4, Qt5/Qt6 (qtct), KDE 色スキーム, fontconfig, xsettingsd などの設定ファイルをテンプレートから生成
- プリセット/壁紙/カスタム Base16 でテーマ選択
- 差分なしなら書き換えをスキップする設計（実装予定）
- フォールバック重視：不備のあるプリセットや欠損はデフォルト (One Dark/One Light) に置換

## Status

- テンプレート・プリセット・設定スキーマは整備済み
- Rust 実装はこれから着手（`apaite-core` にローダとテンプレート適用を実装予定）

## Repository Layout

- `assets/config.toml` … サンプル設定 (Base16 スキーマ)
- `assets/templates/` … GTK/Qt/KDE/fontconfig/xsettingsd などのテンプレート（`{{base0X}}` 参照）
- `assets/presets/` … Base16 プリセット（TOML/YAML）。`default/` に One Dark/One Light
- `apatite-core/` … コアロジック用クレート（未実装）
- `cli/`, `gui/` … CLI/GUI ラッパー（未実装）

## Config Schema (Base16)

サンプル: `assets/config.toml`

```/dev/null/config.example.toml#L1-40
version = 1

[theme]
mode = "preset"         # preset | wallpaper | custom
preference = "dark"     # light | dark

[theme.preset]
dark = "default"
light = "default"

[theme.wallpaper]
path = "~/Pictures/wallpapers/current_wallpaper.jpg"

[theme.custom.dark]
base00 = "#282c34"
...
base0F = "#be5046"

[theme.custom.light]
base00 = "#fafafa"
...
base0F = "#986801"
```

### Modes

- `preset` : 内蔵 or ユーザーの Base16 プリセットを名前で指定
- `wallpaper` : 壁紙から Base16 (light/dark) を生成（実装予定）
- `custom` : `theme.custom.dark/light` に Base16 を直書き

### Preference

- `preference = "dark" | "light"` で適用セットを選択（選択済み `base00..base0F` をテンプレに渡す想定）

## Presets

- 内蔵: `assets/presets/default/default.toml` (One Dark/One Light)
- 有名どころ: Dracula, Nord, Solarized, Gruvbox, Catppuccin, Tokyo Night など（YAML）
- テンプレート: `assets/presets/templates/template.{toml,yaml}`
- ユーザープリセット置き場: `~/.config/apatite/presets/*.{toml,yaml,yml}`

### Lookup Priority

1. ユーザー `~/.config/apatite/presets/*.{toml,yaml,yml}`
2. 内蔵 `assets/presets/**/*.{toml,yaml,yml}`

同名があればユーザーを優先。

### YAML Structure (Base16)

```/dev/null/preset.yaml#L1-20
system: "base16"
name: "Dracula"
author: "..."
variant: "dark"          # or "light"
palette:
  base00: "#282a36"
  ...
  base0F: "#bd93f9"
```

TOML も同内容を `theme.preset.dark/light` に持つだけ。

## Targets & Templates

- GTK2: `assets/gtk2/.gtkrc-2.0`
- GTK3: `assets/gtk3/gtk.css`, `settings.ini`
- GTK4: `assets/gtk4/gtk.css`, `settings.ini`
- Qt5/Qt6 (qtct): `assets/qtct/qtct.conf`
- KDE color scheme: `assets/kcolorscheme/apatite.colors`
- fontconfig: `assets/fontconfig/fonts.conf`
- xsettingsd: `assets/xsettingsd/xsettingsd.conf`
- アイコン/カーソル: `assets/icons/index.theme`

すべて `{{base0X}}` に統一。影はテンプレ内の固定 RGBA。

## Planned CLI (sketch)

- `apatite apply [--config <path>] [--mode <preset|wallpaper|custom>] [--preference <dark|light>] [--targets gtk3,qt6,...] [--dry-run] [--force]`
- `--dry-run` で生成結果を標準出力
- 既存ファイルと差分が無ければスキップ、差分があればアトミック書き込み

## Build/Dev (予定)

- `cargo run -p cli -- apply --config assets/config.toml --dry-run`
- `apatite-core` に以下を実装予定:
  - `load_config` (TOML)
  - `load_preset` (TOML/YAML → Base16)
  - `palette_from_wallpaper` (ダミー実装から開始)
  - `select_palette` (preference に基づき light/dark を選択し flatten)
  - `render_templates` (tera, `{base00..base0F}` を渡す)
  - 出力先: XDG_CONFIG_HOME/XDG_DATA_HOME に従う
  - フォールバック: 読み込み失敗/欠落時は該当セットをデフォルトに置換し INFO ログ

## License & Notices

- このプロジェクトのライセンスは `LICENSE` を参照

## Contributing

1. `cargo fmt && cargo clippy`
2. テンプレート変更時は `{{base0X}}` 参照に統一
3. 新規プリセット追加時は YAML/TOML のいずれかで base00..base0F を埋める
4. 壊れたプリセットや欠損値はデフォルトでフォールバックする方針で実装・テスト
