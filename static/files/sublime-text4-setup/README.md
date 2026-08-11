# Sublime Text 4 — Zed-style setup (config bundle)

Copy these files into:

```text
~/Library/Application Support/Sublime Text/Packages/User/   # macOS
~/.config/sublime-text/Packages/User/                       # Linux
```

## Before use

1. Replace every `HOME_DIR` with your home directory absolute path  
   (e.g. `/Users/you` or `/home/you`).
2. Adjust tool paths if you do not use **mise** / **Homebrew** the same way:
   - Go / Python / Bun: `HOME_DIR/.local/share/mise/installs/...`
   - Odin / OLS: `/opt/homebrew/bin` (Apple Silicon Homebrew)
3. Install packages via Package Control (see the blog post).
4. Install **CaskaydiaCove Nerd Font Mono** (or another Nerd Font Mono) for Terminus.
5. Keep backups **outside** `Packages/` so Sublime does not load them as plugins.

## Key files

| File | Purpose |
|------|---------|
| `Preferences.sublime-settings` | Font, theme auto light/dark, excludes |
| `Zed Dark/Light.sublime-theme` | UI chrome (sidebar, tabs) |
| `Zed One Dark/Light.sublime-color-scheme` | Syntax colors |
| `LSP*.sublime-settings` | gopls, pyright, ruff, typescript, OLS client |
| `*.sublime-build` | ⌘B run for Go / Python / JS / TS / Odin |
| `Odin.sublime-syntax` | Odin highlighting |
| `Go Module.sublime-syntax` | go.mod highlighting + Go icon |
| `file_type_*.tmPreferences` | Sidebar icons for FileIcons |
| `Terminus*.sublime-settings` | Integrated terminal + Nerd Font |
| `Default (OSX).sublime-keymap` | Shortcuts (macOS) |

See the full walkthrough in the blog post: *Sublime Text 4 as a Zed-like polyglot IDE*.
