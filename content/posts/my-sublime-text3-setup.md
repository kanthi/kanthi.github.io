+++
date = "2024-12-15T16:10:28+05:30"
title = "My Sublime Text 4 Setup: A Modern Developer's Configuration"
draft = false
tags = ["sublimetext", "setup", "editor", "development"]
categories = ["Sublime Text", "Setup", "Development Tools"]
bookToc = true
hideToc = false

+++

![Sublime Text 4 Setup](/img/my-sublimetext-setup/st31.png)

Sublime Text has been my go-to editor for years, and with the release of Sublime Text 4, it's gotten even better. While VSCode dominates the market with its extensive plugin ecosystem, Sublime Text 4 remains unmatched in speed, responsiveness, and elegance. Here's my current setup that balances functionality with performance.

## Why Sublime Text 4?

Sublime Text 4 brings significant improvements over ST3:
- Native support for TypeScript, JSX, and TSX
- Improved syntax highlighting engine
- Better performance with large files
- Enhanced auto-completion
- Built-in Git integration improvements
- Tab multi-select functionality

## Essential Packages

### Package Control
First things first - install Package Control if you haven't already. Press `Ctrl+Shift+P` (or `Cmd+Shift+P` on Mac), type "Install Package Control" and hit enter.

### Core Packages

**A File Icon**
Beautiful file icons in the sidebar that make navigation much easier.

**Material Theme**
A clean, modern theme that's easy on the eyes. I prefer the darker variants for long coding sessions.

**SideBarEnhancements**
Adds essential functionality to the sidebar like "New File", "New Folder", "Edit", "Open in Browser" and more.

**BracketHighlighter**
Highlights matching brackets, braces, and tags. Essential for any developer working with nested code.

**Emmet**
The essential toolkit for web developers. Write HTML and CSS faster with abbreviations.

## UI and Theme Configuration

### Theme Setup
I use the **Materialize** theme with these settings in my user preferences:

```json
{
    "theme": "Materialize.sublime-theme",
    "color_scheme": "Packages/Materialize/schemes/Material Darker.tmTheme",
    "font_face": "JetBrains Mono",
    "font_size": 13,
    "line_padding_bottom": 2,
    "line_padding_top": 2
}
```

### Font Recommendations
- **JetBrains Mono**: Excellent for coding with ligatures
- **Fira Code**: Another great option with programming ligatures
- **Source Code Pro**: Clean and readable

## Development Packages

### Code Quality and Linting

**SublimeLinter**
The foundation for all linting in Sublime Text. Install these additional linters:
- SublimeLinter-eslint (JavaScript/TypeScript)
- SublimeLinter-stylelint (CSS/SCSS)
- SublimeLinter-pylint (Python)
- SublimeLinter-golint (Go)

**Trailing Spaces**
Highlights and removes trailing whitespace automatically.

### Git Integration

**GitGutter**
Shows git diff information in the gutter next to line numbers.

**Git**
Provides Git commands through the command palette.

**Sublime Merge Integration**
If you use Sublime Merge, this provides seamless integration.

### Web Development

**HTML-CSS-JS Prettify**
Formats and beautifies your web code. Configure it to run on save:

```json
{
    "html": {
        "allowed_file_extensions": ["htm", "html", "xhtml", "shtml", "xml", "svg"],
        "format_on_save": true
    },
    "css": {
        "allowed_file_extensions": ["css", "scss", "sass", "less"],
        "format_on_save": true
    },
    "js": {
        "allowed_file_extensions": ["js", "jsx", "ts", "tsx", "json"],
        "format_on_save": true
    }
}
```

**ColorHighlighter**
Displays colors in CSS files as colored backgrounds or borders.

**AutoFileName**
Autocompletes file paths and names when typing.

**LiveReload**
Automatically refreshes your browser when files change.

### Markdown Support

**MarkdownEditing**
Enhanced Markdown editing with better syntax highlighting and shortcuts.

**MarkdownPreview**
Preview Markdown files in your browser.

### Language-Specific Packages

**TOML**
Syntax highlighting for TOML configuration files.

**Dockerfile Syntax Highlighting**
Essential if you work with Docker.

**SASS**
Better SCSS/Sass support with completions and snippets.

**TypeScript**
Enhanced TypeScript support (though ST4 has improved built-in support).

## General Configuration

Here's my complete user settings file:

```json
{
    "theme": "Materialize.sublime-theme",
    "color_scheme": "Packages/Materialize/schemes/Material Darker.tmTheme",
    "font_face": "JetBrains Mono",
    "font_size": 13,
    "line_padding_bottom": 2,
    "line_padding_top": 2,
    "highlight_line": true,
    "caret_style": "phase",
    "wide_caret": true,
    "caret_extra_width": 1,
    "bold_folder_labels": true,
    "highlight_modified_tabs": true,
    "show_encoding": true,
    "show_line_endings": true,
    "trim_trailing_white_space_on_save": true,
    "ensure_newline_at_eof_on_save": true,
    "translate_tabs_to_spaces": true,
    "tab_size": 2,
    "rulers": [80, 120],
    "word_wrap": false,
    "scroll_past_end": true,
    "fold_buttons": true,
    "fade_fold_buttons": false,
    "show_definitions": false,
    "auto_complete_triggers": [
        {"selector": "text.html", "characters": "<"},
        {"selector": "text.html", "characters": " "},
        {"selector": "source.css", "characters": ":"},
        {"selector": "source.css", "characters": " "}
    ]
}
```

## Language-Specific Configurations

### Python Development

**Anaconda**
Comprehensive Python package with linting, auto-completion, and more.

**SublimeREPL**
Run Python code directly in Sublime Text.

### Go Development

**GoSublime**
Complete Go development environment with formatting, building, and testing.

### C/C++ Development

For C development, I use a custom build system:

```json
{
    "shell_cmd": "gcc \"${file}\" -o \"/tmp/${file_base_name}\" && \"/tmp/${file_base_name}\"",
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "working_dir": "${file_path}",
    "selector": "source.c",
    "variants": [
        {
            "name": "Run in Terminal",
            "shell_cmd": "gcc \"${file}\" -o \"/tmp/${file_base_name}\" && open -a Terminal \"/tmp/${file_base_name}\""
        }
    ]
}
```

## Essential Keyboard Shortcuts

Here are the shortcuts I use most frequently:

- `Cmd+P` (Mac) / `Ctrl+P` (Windows/Linux): Quick file navigation
- `Cmd+Shift+P`: Command palette
- `Cmd+D`: Select next occurrence of current word
- `Cmd+Ctrl+G`: Select all occurrences
- `Cmd+L`: Select entire line
- `Cmd+Shift+D`: Duplicate line
- `Cmd+Shift+K`: Delete line
- `Cmd+/`: Toggle comment
- `Cmd+Shift+V`: Paste and indent
- `Cmd+R`: Go to symbol
- `Cmd+;`: Go to word
- `F12`: Go to definition

## Advanced Features

### Multiple Cursors
One of Sublime Text's killer features. Use `Cmd+Click` to place multiple cursors, or `Cmd+D` to select multiple instances of the same word.

### Vintage Mode
Enable Vim key bindings by adding `"ignored_packages": []` to your settings (removing "Vintage" from the ignored packages list).

### Project-Specific Settings
Create `.sublime-project` files for project-specific configurations:

```json
{
    "folders": [
        {
            "path": "."
        }
    ],
    "settings": {
        "tab_size": 4,
        "translate_tabs_to_spaces": false
    }
}
```

## Performance Tips

1. **Disable unused packages**: Keep only what you need
2. **Use .sublime-project files**: Better performance for large projects
3. **Exclude build folders**: Add `node_modules`, `dist`, etc. to folder exclude patterns
4. **Limit file indexing**: Use `"index_files": false` for very large projects

## Conclusion

Sublime Text 4 remains an excellent choice for developers who value speed and simplicity. While it may not have the extensive plugin ecosystem of VSCode, its core functionality is rock-solid, and the packages mentioned above cover most development needs.

The key is finding the right balance between functionality and performance. Start with the essential packages and add more as needed. Remember, one of Sublime Text's greatest strengths is its speed – don't compromise that with too many heavy plugins.

What's your Sublime Text setup like? Let me know in the comments if you have any favorite packages or configurations I should try!

---

*Last updated: December 2024 for Sublime Text 4*