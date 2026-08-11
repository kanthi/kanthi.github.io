+++
date = "2026-08-11T20:00:00+05:30"
draft = true
title = "Sublime Text 4 as a Zed-like Polyglot IDE"
tags = ["sublimetext", "setup", "editor", "go", "python", "javascript", "odin", "lsp", "terminus", "macos"]
categories = ["Sublime Text", "Setup", "Development Tools"]
keywords = ["sublime text 4", "zed theme", "lsp", "gopls", "odin", "terminus", "file icons", "mise"]
description = "Turn Sublime Text 4 into a fast, Zed-inspired polyglot editor: custom themes, matched fonts, LSP for Go/Python/JS/TS/Odin, sidebar icons, ⌘B runners, and a Nerd Font terminal."
bookToc = true
hideToc = false
showFullContent = false
+++

Sublime Text is still one of the fastest editors you can run on a laptop. This post captures a full **Sublime Text 4** setup that *looks* a bit like [Zed](https://zed.dev) (One Dark surfaces, clean chrome) while keeping Sublime’s strengths: instant startup, solid multi-language builds, and a real terminal via **Terminus**.

It is written for **macOS** (Apple Silicon paths with Homebrew + [mise](https://mise.jdx.dev)), but the same ideas apply on Linux if you adjust paths.

**Config bundle (copy into `Packages/User/`):**  
[/files/sublime-text4-setup/](/files/sublime-text4-setup/)

---

## What you get

| Area | Result |
|------|--------|
| UI | Custom **Zed Dark / Zed Light** themes + One Dark / One Light color schemes |
| Fonts | **IBM Plex Mono** in editor + sidebar (same family, aligned size) |
| Languages | Go, Python, JavaScript, TypeScript, Odin (+ Zig/Swift/Kotlin syntax for icons) |
| LSP | gopls, pyright, ruff, typescript, OLS (Odin) |
| Run | **⌘B** runs the current file (`go run`, `python`, `node`, `bun`, `odin run -file`) |
| Sidebar | Language **FileIcons** (go, js, odin, zig, …) |
| Terminal | **Terminus** panel with **CaskaydiaCove Nerd Font Mono** |

---

## Prerequisites

1. [Sublime Text 4](https://www.sublimetext.com/)
2. [Package Control](https://packagecontrol.io/installation)
3. Tooling on `PATH` (examples used in this setup):
   - **mise** for Go, Python, Node/Bun
   - **Homebrew** for `odin`, `ols`, `odinfmt`
4. A **Nerd Font Mono** for the terminal (this setup uses **CaskaydiaCove Nerd Font Mono**)

Useful checks:

```bash
which go python3 node bun odin ols gopls
go version
odin version
```

---

## Packages (Package Control)

Install:

- **LSP**
- **LSP-gopls**
- **LSP-pyright**
- **LSP-ruff**
- **LSP-typescript**
- **FileIcons** (color variant — better language recognition than mono-only)
- **Terminus**
- **BracketHighlighter**, **EditorConfig**, **GitGutter** (optional quality-of-life)

Command Palette → `Package Control: Install Package`.

Extra syntax (when FileIcons need a real scope, not Plain Text):

| Language | How |
|----------|-----|
| Odin | `Odin.sublime-syntax` in this bundle (User package) |
| go.mod | `Go Module.sublime-syntax` in this bundle |
| Zig | e.g. [sublime-zig-language](https://github.com/ziglang/sublime-zig-language) under `Packages/Zig` |
| Swift | Small `Swift.sublime-syntax` (included in bundle if present) |
| Kotlin | Community `Kotlin.tmLanguage` package |

**Important:** Do **not** keep backups under `Packages/User/_backups/`. Sublime loads everything under `Packages/` as packages and will try to parse old `.tmLanguage` files. Keep backups outside `Packages/`, e.g.:

```text
~/Library/Application Support/Sublime Text/Backups/
```

---

## 1. Look and feel (Zed-style)

### Preferences

Theme auto-follows system light/dark. Editor font matches the sidebar face.

```json
{
  "theme": "auto",
  "light_theme": "Zed Light.sublime-theme",
  "dark_theme": "Zed Dark.sublime-theme",
  "color_scheme": "auto",
  "light_color_scheme": "Zed One Light.sublime-color-scheme",
  "dark_color_scheme": "Zed One Dark.sublime-color-scheme",
  "font_face": "IBM Plex Mono",
  "font_size": 18,
  "line_padding_top": 2,
  "line_padding_bottom": 2
}
```

Full file: [`Preferences.sublime-settings`](/files/sublime-text4-setup/Preferences.sublime-settings).

### Themes and schemes

These live in `Packages/User/`:

- [`Zed Dark.sublime-theme`](/files/sublime-text4-setup/Zed%20Dark.sublime-theme) / [`Zed Light.sublime-theme`](/files/sublime-text4-setup/Zed%20Light.sublime-theme)  
  - extend Adaptive  
  - sidebar background matches editor  
  - sidebar labels: IBM Plex Mono @ 18  
  - blue-tinted selection, accent folder icons  
- [`Zed One Dark.sublime-color-scheme`](/files/sublime-text4-setup/Zed%20One%20Dark.sublime-color-scheme) / Light  
  - One Dark-style scopes + explicit Odin rules  

Sidebar font is **not** controlled by `font_size` in Preferences — only by theme rules on `sidebar_label` (`font.face` / `font.size`).

---

## 2. macOS GUI apps and PATH

Dock-launched Sublime does **not** load your interactive shell `PATH`. mise-installed `go` / `python` / `gopls` will look “missing” until you set environment variables for LSP and builds.

Pattern used everywhere:

```text
HOME_DIR/.local/share/mise/installs/go/latest/bin
HOME_DIR/.local/share/mise/installs/python/latest/bin
HOME_DIR/.local/share/mise/installs/bun/latest/bin
/opt/homebrew/bin
```

In the downloadable files, `HOME_DIR` is a placeholder — replace it with your absolute home path (Sublime does not expand `$HOME` in JSON).

### LSP global env

See [`LSP.sublime-settings`](/files/sublime-text4-setup/LSP.sublime-settings): `environ.PATH`, `GOROOT`, `GOPATH`, `ODIN_ROOT`.

### gopls and “go binary not found”

**LSP-gopls** may try to *install* gopls with `go` using Sublime’s process PATH. If that fails:

1. Install gopls yourself: `go install golang.org/x/tools/gopls@latest`
2. Point the client at it and set `"manageGoplsBinary": false`

Example: [`LSP-gopls.sublime-settings`](/files/sublime-text4-setup/LSP-gopls.sublime-settings).

---

## 3. Language servers

| Language | Client | Notes |
|----------|--------|--------|
| Go | LSP-gopls | Absolute gopls path + PATH to `go` |
| Python | LSP-pyright + LSP-ruff | mise python + `ruff server` |
| JS/TS | LSP-typescript | Needs network once to fetch tsserver |
| Odin | LSP client `odin` → `/opt/homebrew/bin/ols` | `brew install ols odinfmt odin` |

Odin OLS client (in `LSP.sublime-settings`):

```json
"clients": {
  "odin": {
    "command": ["/opt/homebrew/bin/ols"],
    "enabled": true,
    "selector": "source.odin",
    "initializationOptions": {
      "odin_command": "/opt/homebrew/bin/odin",
      "odin_root_override": "/opt/homebrew/opt/odin/libexec",
      "enable_hover": true,
      "enable_format": true,
      "enable_document_symbols": true
    }
  }
}
```

Optional workspace file at the repo root: `ols.json` (collections + `odin_command`). Homebrew’s `ols` wrapper already sets `OLS_BUILTIN_FOLDER`.

---

## 4. ⌘B = run current file

Build systems in the bundle:

| File | Default command |
|------|-----------------|
| `Go.sublime-build` | `go run $file` |
| `Python.sublime-build` | `python3 -u $file` |
| `JavaScript.sublime-build` | `node $file` |
| `TypeScript.sublime-build` | `bun run $file` |
| `Odin Run.sublime-build` | `odin run $file -file` |

Keymap idea (macOS): for each selector, **⌘B** forces that build system; **⌘⇧B** opens variants (Build / Test / Check).

**Pitfall:** do **not** put a top-level `"name"` key in a `.sublime-build` — Sublime passes it to `exec` and you get:

```text
__init__() got an unexpected keyword argument 'name'
```

Variant entries may use `"name": "Run"`; the root build must not.

Odin single-file tutorials need `-file`:

```bash
odin run hello.odin -file
```

Package-style (video-style) remains a variant: `odin run .`.

---

## 5. Sidebar file icons

**FileIcons** maps **syntax scopes** → PNG icons. If a file is Plain Text, you get a blank document.

| Scope | Icon |
|-------|------|
| `source.go` / `source.go.mod` | Go |
| `source.js` / `source.ts` | JS / TS |
| `source.odin` | Odin |
| `source.zig` / `source.swift` | Zig / Swift |

Custom themes must ship FileIcons overlays named like the theme file:

```text
Packages/FileIcons/theme/Zed Dark.sublime-theme
Packages/FileIcons/theme/Zed Light.sublime-theme
```

with `icon_file_type` + `content_margin`. This bundle assumes that FileIcons package layout; install FileIcons and add the two theme overlays (or copy from a working machine).

User overrides in the bundle:

- `file_type_go.tmPreferences` (includes `source.go.mod`)
- `file_type_odin.tmPreferences`, `file_type_swift.tmPreferences`, …

`go.mod` needs a real syntax so the icon can attach — see `Go Module.sublime-syntax` (`scope: source.go.mod`).

---

## 6. Odin syntax (durable)

Keep the Odin grammar in **User** so it is not lost if a loose `Packages/Odin` folder is cleaned up:

```text
Packages/User/Odin.sublime-syntax   → scope: source.odin
```

File: [`Odin.sublime-syntax`](/files/sublime-text4-setup/Odin.sublime-syntax).

After install: open a `.odin` file → bottom-right should say **Odin**. If it says Plain Text: click → **Odin** → **Open all with current extension as…**.

---

## 7. Terminus terminal

### Settings

- Shell: `zsh -i -l` with the same PATH/ODIN_ROOT/GOPATH env  
- Theme: Zed-ish One Dark palette (`theme: user`)  
- Font: **CaskaydiaCove Nerd Font Mono** so powerline/nerd glyphs work  

Files:

- [`Terminus.sublime-settings`](/files/sublime-text4-setup/Terminus.sublime-settings)
- [`Terminus View.sublime-settings`](/files/sublime-text4-setup/Terminus%20View.sublime-settings)

Use a **Nerd Font Mono** (not Propo) in the terminal.

### Shortcuts (macOS, from the keymap)

| Keys | Action |
|------|--------|
| **⌃\`** or **⌘J** | Toggle terminal panel (cwd = file/project) |
| **⌃⇧\`** | Open panel in current folder |
| **⌃⌥T** | Terminal as tab |
| **⌃⌥⇧T** | Panel at project root |

Close and reopen the panel after changing fonts.

---

## 8. Suggested layout of `Packages/User`

```text
Packages/User/
  Preferences.sublime-settings
  Zed Dark.sublime-theme
  Zed Light.sublime-theme
  Zed One Dark.sublime-color-scheme
  Zed One Light.sublime-color-scheme
  LSP.sublime-settings
  LSP-gopls.sublime-settings
  LSP-pyright.sublime-settings
  LSP-ruff.sublime-settings
  LSP-typescript.sublime-settings
  Go.sublime-build
  Python.sublime-build
  JavaScript.sublime-build
  TypeScript.sublime-build
  Odin Run.sublime-build
  Odin.sublime-syntax
  Go Module.sublime-syntax
  file_type_*.tmPreferences
  Terminus.sublime-settings
  Terminus View.sublime-settings
  Default (OSX).sublime-keymap
```

Downloadable snapshot: **[/files/sublime-text4-setup/](/files/sublime-text4-setup/)**  
Bundle README: **[/files/sublime-text4-setup/README.md](/files/sublime-text4-setup/README.md)**

---

## Quick verification checklist

1. Sidebar + editor both use IBM Plex Mono and feel aligned  
2. Open `.go` / `.py` / `.js` / `.ts` / `.odin` → correct syntax + icons  
3. **⌘B** runs the file; bottom panel shows output  
4. LSP: hover / go-to-definition works (restart servers if needed)  
5. **⌃\`** opens Terminus; nerd icons in the prompt render  
6. Console has no errors about `_backups` or missing syntax files  

```bash
# From Terminus
go version && odin version && python3 --version && node -v
```

---

## Why not only VS Code / Zed?

For day-to-day polyglot tinkering (a multi-language `codeplay` tree, quick `odin run -file`, Go snippets), Sublime stays absurdly snappy. This setup keeps that speed while borrowing a cleaner UI language from Zed and a real shell from Terminus.

If you already have an older ST3 post on this blog, treat this as the ST4 / 2026 successor: LSP-first, explicit PATH for GUI apps, and configs you can copy from the static bundle.

---

## Related

- Older notes: [My Sublime Text 4 Setup](/posts/my-sublime-text3-setup/)  
- Tooling: [mise version manager](/posts/mise-version-manager/)  
- Editors: [My VS Code Setup](/posts/my-vscode-setup/)  

Happy hacking.
