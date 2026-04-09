# Moonlander "Udon" — Input System Reference

An input orchestration stack on macOS that coordinates a custom keyboard, a tiling window manager, system-wide Vim motions, and several utility apps — all tied together through Karabiner-Elements.

## Moving parts

**Hardware:** ZSA Moonlander running QMK firmware. Five layers (English base, Russian, symbols/numpad, mouse/media, one-shot arrows). Tap dances and combos on the big keys and thumb cluster trigger app shortcuts. Dual-function keys throughout (tap for character, hold for modifier or layer).

**Orchestration:** Karabiner-Elements is the central hub. It intercepts shortcuts from the keyboard, switches the macOS input source, sets state variables, and shells out to `kontroll` (a CLI that talks to the Moonlander over USB to change layers from the OS side).

**Window management:** AeroSpace, a tiling window manager with Vim-style Alt+HJKL navigation and a service mode for advanced operations (join, fullscreen, reset layout).

**System-wide Vim:** KindaVim provides normal/insert/visual modes across all macOS apps. Mode state is propagated via Watchman to Karabiner and BetterTouchTool, so other tools can react to the current Vim mode.

**Apps triggered from the keyboard:** Things (task manager), Alfred (launcher, clipboard), 1Password, Mouseless (keyboard-driven UI navigation), AquaVoice (voice dictation), Tana (knowledge base capture), Contexts (window switcher).

## Keyboard

Moonlander has TapDances, Combos.

## Deterministic language switching

Moonlander and OS has two "layouts" for Russian - Moonlander implement it as a layer to have deterministic layout switching. It means that they should be in sync when something is changed on OS layer or in Moonlander.

The fix: the firmware sends a direction-specific shortcut on layer change (one for Russian, one for English). Karabiner intercepts each and calls `select_input_source` to force the correct language — never a toggle. It also sets variables (`kv_current_input_ru`, `kv_current_input_en`) that other Karabiner rules can condition on.

Shortcuts containing letter keycodes (e.g. for Mouseless or AquaVoice) break when the OS input source is Russian — the physical key sends a different keycode. Karabiner handles this by intercepting the shortcut, running `kontroll set-layer --index 0` to force the English firmware layer, then re-sending the shortcut so it arrives with the correct keycode.

## AeroSpace — window management

AeroSpace uses Alt as its modifier key with Vim-style directions.

**Main mode:** Alt+HJKL to focus windows, Alt+Shift+HJKL to move them. Alt+/ and Alt+, to switch between tiling layouts. Alt+Shift+; enters service mode.

**Service mode:** Single-key actions (R to reset layout, F to toggle floating, Backspace to close other windows, Esc to reload config). Alt+Shift+HJKL joins windows in a direction. Alt+Shift+M for fullscreen. Every action returns to main mode automatically.

## KindaVim — system-wide Vim

KindaVim writes its current mode to `~/Library/Application Support/kindaVim/environment.json`. A Watchman trigger watches this file and runs `~/kindaVimModeWatcher.sh` on every change. The watcher script reads the mode and propagates it to:

1. **Karabiner** — calls `karabiner_cli --set-variables '{"kindaVimState":"<mode>"}'`, making the mode available as a variable in any Karabiner rule
2. **BetterTouchTool** — sets a persistent string variable `kindaVimState` via AppleScript, enabling mode-aware triggers

This lets the rest of the stack behave differently depending on whether Vim normal, insert, or visual mode is active. The Watchman trigger is configured by running `~/setup-watchman-triggers.sh`.

```bash
cat ~/setup-watchman-triggers.sh

#!/bin/bash

# Set up Watchman watch
/opt/homebrew/bin/watchman --logfile=/var/folders/by/3cypzwdx14q1fjswl_xz4hwm0000gn/T//alexanderdoroshenko-state/log watch "/Users/alexanderdoroshenko/Library/Application Support/kindaVim/"

# Set up Watchman trigger
watchman -j <<EOT
["trigger", "/Users/alexanderdoroshenko/Library/Application Support/kindaVim/", {
  "name": "kindaVimState",
  "expression": ["match", "environment.json"],
  "command": ["/bin/bash", "/Users/alexanderdoroshenko/kindaVimModeWatcher.sh"],
  "stdin": ["name"]}]
EOT
```

```bash
cat /Users/alexanderdoroshenko/kindaVimModeWatcher.sh

#!/bin/bash

mode=$(jq -r .mode '/Users/alexanderdoroshenko/Library/Application Support/kindaVim/environment.json')
/Library/Application\ Support/org.pqrs/Karabiner-Elements/bin/karabiner_cli --set-variables "{\"kindaVimState\":\"$mode\"}"
#open "btt://set_persistent_string_variable/?variableName=kindaVimState&to=$mode"
osascript -e "tell application \"BetterTouchTool\" to set_persistent_string_variable \"kindaVimState\" to \"$mode\""
```

## Neovim

Minimal setup with lazy.nvim. Leader is Space. Telescope for fuzzy finding: `<leader>ff` files, `<leader>fg` live grep, `<leader>fr` recent files, `<leader>fb` buffers, `<leader>pf` git files, `<leader>fR` resume last picker. Cmd+S saves, Cmd+V pastes from system clipboard (mapped across all modes).

Config: `~/.config/nvim/init.lua`

## Shell (zsh)

**Shortcuts:** Ctrl+X Ctrl+E opens the current command line in `$EDITOR` (vim). fzf provides Ctrl+R for history search, Ctrl+T for file finder, and Alt+C to cd into a directory.

**Tools:** `zoxide` for smart directory jumping (replaces `cd`). `wtree` shell function creates git worktrees with automatic dependency install, symlinks for docs/.cursor from the primary working tree, and opens them in Cursor.

**Aliases:** `python`/`pip` point to python3/pip3, `marked` opens files in Marked 2.

## Ghostty — terminal

Ghostty uses Cmd-based keybindings for split management, keeping them out of the way of Neovim bindings inside the terminal.

**Splits:** Cmd+Shift+V to split right, Cmd+Shift+S to split down, Cmd+Shift+C to close a split, Cmd+Shift+= to equalize splits. Navigate between splits with Cmd+HJKL.

## BetterTouchTool and Leader key for Telegram

Right Shift arms a Telegram leader mode; TH ("Home") / TP ("Pacani") / TN ("Neela") dispatch predefined Telegram shortcuts, then disarm the leader. It works by sending to Telegram shortcuts like  Cmd+2, then Opt+Cmd+3, based on folders and chats.

## Karabiner and BetterTouchTool integration

Karabiner-Elements serves three roles:

1. **Language switching** — intercepts firmware shortcuts and forces the correct macOS input source
2. **Shortcut routing** — intercepts Hyper+letter shortcuts, forces the English firmware layer via `kontroll`, and re-sends so apps receive the correct keycode regardless of current input source
3. **State hub** — maintains variables for current language and Vim mode that other rules can reference

BetterTouchTool receives the KindaVim mode state and can condition its own triggers on it. The two tools share state through Karabiner variables and BTT persistent string variables, both updated by the same watcher script.
