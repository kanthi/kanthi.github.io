+++
title = "My Favorite Rust CLI Tools for macOS: A 2026 Guide"
date = "2026-01-03T11:24:01+05:30"
author = "Kanthi"
authorTwitter = ""
cover = ""
tags = ["rust", "cli", "tools", "productivity", "macos", "homebrew"]
keywords = ["rust", "cli", "macos", "shell", "brew", "dev-tools"]
description = "I swapped my standard terminal tools for modern Rust equivalents, and I'm never going back. Here's my setup."
showFullContent = false
readingTime = true
hideComments = false
+++

I practically live in the terminal. For years, I was content with the standard BSD utilities that come with macOS—`ls`, `grep`, `find`, `cat`. They work, they're everywhere, and I have the muscle memory for them.

But recently, I decided to overhaul my setup with modern alternatives, most of which happen to be written in **Rust**. I wasn't looking for change for the sake of change, but these tools are legitimately faster, smarter, and just nicer to look at. They usually have sensible defaults (like ignoring `.git` directories) that save me from typing the same flags over and over.

If you're on a Mac and want to freshen up your terminal workflow, here are the tools I'm using right now.

## 1. ripgrep (rg) > grep

I can't imagine working without `rg` anymore. It's shockingly fast. The best part is that it respects my `.gitignore` automatically. I don't have to tell it to skip `node_modules` or `target` folders; it just knows.

*   **Install**: `brew install ripgrep`
*   **Try it**: `rg "search_term"`

## 2. bat > cat

`bat` is essentially `cat` with superpowers. It adds syntax highlighting and line numbers, and remarkably, it integrates with git to show added/modified lines in the margin. It's great for quickly peeking at code.

*   **Install**: `brew install bat`
*   **Try it**: `bat src/main.rs`

## 3. eza > ls

I used to use `exa`, but `eza` is the maintained community fork. It makes directory listings actually readable with colors and icons. The tree view is fantastic for quickly visualizing project structure.

*   **Install**: `brew install eza`
*   **Try it**: `eza -la --icons`

## 4. fd > find

I never could remember the exact syntax for `find`. `fd` is simple: `fd pattern`. It ignores hidden files and gitignored patterns by default, which is almost always what I want.

*   **Install**: `brew install fd`
*   **Try it**: `fd "config"`

## 5. zoxide > cd

`zoxide` changed how I navigate. It "learns" which directories I visit most. Instead of typing `cd ~/Workspace/projects/my-app`, I just type `z my-app` and I'm there. It feels like magic.

*   **Install**: `brew install zoxide`
*   **Try it**: `z app` (after using it a bit)

## 6. Starship (Prompt)

I used to spend hours tweaking my Zsh prompt. `Starship` works out of the box and gives me exactly what I need: git status, language versions (Node, Rust, Python), and error codes. It's fast and doesn't lag my terminal.

*   **Install**: `brew install starship`
*   **Setup**: Add `eval "$(starship init zsh)"` to your `~/.zshrc`

## 7. Delta (Git Diff)

Looking at standard git diffs can be painful. `Delta` gives me a clean, syntax-highlighted, side-by-side view. It makes code reviews in the CLI actually pleasant.

*   **Install**: `brew install git-delta`
*   **Setup**: Add it to your `~/.gitconfig` as the pager.

## 8. dust > du

When my disk space is low, `dust` saves me. It gives a visual graph of where all my space is going, so I can instantly spot that one massive `node_modules` folder I forgot to delete.

*   **Install**: `brew install dust`
*   **Try it**: `dust`

## 9. tokei > cloc

Curious how many lines of code are in your project? `tokei` is blazing fast and breaks it down by language.

*   **Install**: `brew install tokei`
*   **Try it**: `tokei .`

## 10. xh > curl

I love `curl`, but `xh` is just friendlier for quick API testing. It formats JSON responses automatically and the syntax for sending JSON data is much simpler.

*   **Install**: `brew install xh`
*   **Try it**: `xh httpbin.org/json`

## 11. hexyl > hexdump

If you ever need to inspect a binary file, `hexyl` makes it a lot less intimidating with color-coded bytes.

*   **Install**: `brew install hexyl`
*   **Try it**: `hexyl huge_image.png`

## 12. broot > tree

`tree` is fine until you run it on a huge directory. `broot` gives you a tree view that fits on your screen and lets you navigate and search inside it.

*   **Install**: `brew install broot`
*   **Try it**: `broot`

## 13. yazi (File Manager)

I'm usually a CLI-only person, but `yazi` is compelling. It's a terminal file manager that's incredibly snappy. It can even preview images right in the terminal, which is wild.

*   **Install**: `brew install yazi`
*   **Try it**: `yazi`

## 14. procs > ps

`procs` looks so much better than `ps`. It gives relevant info like TCP/UDP ports, Docker container names, and memory usage in readable units.

*   **Install**: `brew install procs`
*   **Try it**: `procs`

## 15. bottom > top

For system monitoring, `btm` (bottom) is my go-to. It looks like a dashboard from a sci-fi movie and is fully customizable.

*   **Install**: `brew install bottom`
*   **Try it**: `btm`

## 16. gping > ping

Why look at scrolling text when you can see a graph? `gping` visualizes latency, which helps me visualize when my connection is jittery.

*   **Install**: `brew install gping`
*   **Try it**: `gping google.com`

## 17. bandwhich

This tool is great for answering "what is using all my bandwidth?". It breaks down network usage by process.

*   **Install**: `brew install bandwhich`
*   **Try it**: `sudo bandwhich`

## 18. zellij > tmux

I've tried to learn `tmux` multiple times. `zellij` just clicked for me. It has a layout engine that makes sense and helpful shortcuts on screen so I don't have to memorize everything.

*   **Install**: `brew install zellij`
*   **Try it**: `zellij`

## 19. hyperfine

When I want to prove that `fd` is faster than `find`, I use `hyperfine`. It's a rigorous benchmarking tool that handles warmups and statistical outliers for you.

*   **Install**: `brew install hyperfine`
*   **Try it**: `hyperfine 'sleep 1'`

## 20. tealdeer > man

I rarely want the full `man` page. I just want to know how to use the command. `tealdeer` (a fast tldr client) gives me concise examples.

*   **Install**: `brew install tealdeer`
*   **Try it**: `tldr tar`

## 21. gitui

Sometimes I just want to stage specific chunks of code quickly. `gitui` is blazing fast and lets me do complex git operations with just the keyboard.

*   **Install**: `brew install gitui`
*   **Try it**: `gitui`

## 22. Cargo Tools

If you do any Rust dev, `cargo-update` is essential to keep your binaries (like all the ones on this list!) up to date. `cargo-edit` was so good its functionality is being merged into cargo itself, but checking for updates is still useful.

*   **Install**: `brew install cargo-update`
*   **Try it**: `cargo install-update -a`

## 23. so (Stack Overflow)

Because leaving the terminal to search for an error message breaks my flow. `so` lets me search Stack Overflow right from the CLI.

*   **Install**: `brew install so`
*   **Try it**: `so "how to reverse a string in rust"`

## 24. ddgr (DuckDuckGo)

Sometimes I just need a quick search. `ddgr` lets me query DuckDuckGo and open results quickly.

*   **Install**: `brew install ddgr`
*   **Try it**: `ddgr rust cli tools`

---

Switching to these tools has honestly made my time in the terminal more enjoyable. They aren't just "rewrites"—they rethink how these tools should work in 2026. Give a few of them a shot; `zoxide` and `ripgrep` alone are worth the install.
