# Moonlander "Udon" — App Shortcuts Design

This document maps every app-level shortcut triggered from the Moonlander firmware, how it is triggered, what it sends to macOS, and what the receiving app does with it.

## Design principles

1. **Complex shortcut in the app, simple action on the keyboard.** App-side bindings use Hyper (Ctrl+Shift+Alt+Cmd) or Meh (Ctrl+Shift+Alt) combinations that no standard app would ever claim. The firmware makes them easy to hit.

2. **Two trigger mechanisms:**
   - **Tap dances** on the big keys and thumb cluster — for frequently used app launchers (Things, 1Password, Alfred). Single key, multiple actions via tap/double-tap/hold.
   - **Combos** using Meh (right big key) + right-hand home row — for app actions that need a quick chord without leaving the typing position.

3. **Language safety.** Shortcuts that contain letter keycodes (like Hyper+A) may fail when the OS input source is Russian. These are routed through Karabiner, which runs `kontroll set-layer --index 0` to force the English layer before re-sending the shortcut. Pure modifier combos (like Ctrl+Space) are language-independent and need no special handling.

## Shortcut reference

### Tap dances

These are physical keys that behave differently based on tap count and hold.

#### DANCE_0 — Things (left big key)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| Tap | Ctrl+Space | Quick add task |
| Double-tap | Hyper+N | Open Things window |

**Language safety:** Ctrl+Space is modifier-only, safe in any language. Hyper+N contains a letter — needs Karabiner routing if triggered from Russian layer.

**Physical position:** Left big key (top row, between 5 and the left half cluster edge).

#### DANCE_1 — Alfred (left thumb area, bottom row)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| Tap | Alt+Space | Open Alfred |
| Hold | Alt+Cmd+C | Clipboard history |
| Double-tap | Hyper+Space | *(unassigned — available slot)* |

**Language safety:** All three are modifier-only or modifier+space, safe in any language.

**Physical position:** Bottom row of the left half, right of `MO(3)`.

#### DANCE_2 — 1Password (right big key)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| Tap | Hyper+0 | Quick search and autofill |
| Double-tap | Hyper+9 | Open 1Password window |

**Language safety:** Both use number keys, safe in any language.

**Physical position:** Right big key (top row, between the right half cluster edge and 6).

#### DANCE_3 — Soft/Hard Sign (Russian layer only, M position)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| Tap | RU_SOFT | Types Ь (soft sign) |
| Double-tap | RU_HARD | Types Ъ (hard sign) |

**Note:** This is a typing aid, not an app shortcut. Only active on Layer 1 (Russian).

### Combos (chords)

All app combos start with the **Meh key** (right big key, held) or **Hyper key** plus one or more keys. 

#### Mouseless — keyboard-driven UI navigation

| Chord | Shortcut sent | App function |
|-------|---------------|--------------|
| Meh + H | Hyper+A | Show overlay |
| Meh + H + J | Hyper+S | Free mode |

**Language safety:** Both contain letters (A, S). Routed through Karabiner with `kontroll set-layer --index 0` to guarantee English input source before delivery.

**App-side config:** In Mouseless settings, set "Show overlay" to Cmd+Alt+Shift+Ctrl+A and "Free mode" to Cmd+Alt+Shift+Ctrl+S.

#### AquaVoice — dictation (new)

| Chord | Shortcut sent | App function |
|-------|---------------|--------------|
| Meh + J | Hyper+D | Activate |
| Meh + J + K | Hyper+F | Activate Hands Free |

**Language safety:** Both contain letters (D, F). Will need Karabiner routing if used from Russian layer.

**App-side config:** In AquaVoice settings, set "Activate" to Cmd+Alt+Shift+Ctrl+D and "Activate Hands Free" to Cmd+Alt+Shift+Ctrl+F.

#### Tana — knowledge base quick capture (new)

| Chord | Shortcut sent | App function |
|-------|---------------|--------------|
| Meh + K | Cmd+Shift+Space | Quick Add |

**Language safety:** Modifier+space only, safe in any language. No Karabiner routing needed.

**App-side config:** Tana uses Cmd+Shift+Space by default for Quick Add. No change needed in app settings.

#### Scratchpad — toggle quick notes panel

| Chord | Shortcut sent | App function |
|-------|---------------|--------------|
| Meh + L | Hyper+L | Toggle Scratchpad |

**Language safety:** Contains a letter (L). Will need Karabiner routing if used from Russian layer.

### Left thumb cluster apps

These are dedicated keys on the left thumb cluster that trigger app-level functionality.

#### Contexts — app window switcher (left thumb cluster)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| Hold + type | *TODO* | Search and switch to app window |

**Trigger:** Hold a left thumb key to activate Contexts, type to filter windows, release to switch.

*TODO: Add exact key position, shortcut details, and language safety notes.*

#### AeroSpace — tiling window manager (left thumb cluster)

| Action | Shortcut sent | App function |
|--------|---------------|--------------|
| *TODO* | *TODO* | *TODO* |

**Trigger:** Left thumb cluster key(s).

*TODO: Add keybindings for workspace switching, window tiling commands, etc.*

### Combo key layout on the right hand

Visual reference for how the combo chords map to physical keys:

```
Right big key = Meh (held for all combos)

Home row:  H        J        K        L
           │        │        │        └── + Meh = Scratchpad Toggle
           │        │        └── + Meh = Tana Quick Add
           │        │
           │        ├── + Meh = AquaVoice Activate
           │        └── + K + Meh = AquaVoice Hands Free
           │
           ├── + Meh = Mouseless Show Overlay
           └── + J + Meh = Mouseless Free Mode
```

## Existing combo (non-app)

| Chord | Output | Purpose |
|-------|--------|---------|
| 0 + - (number row) | Hyper | Experimental, not actively used |

## Reserved Hyper/Meh bindings

Summary of all Hyper and Meh key combinations currently in use, to avoid conflicts when adding new shortcuts.

| Shortcut | Used by | Status |
|----------|---------|--------|
| Hyper+0 | 1Password quick search | Active |
| Hyper+9 | 1Password open window | Active |
| Hyper+A | Mouseless show overlay | Active |
| Hyper+D | AquaVoice activate | Proposed |
| Hyper+F | AquaVoice hands free | Proposed |
| Hyper+L | Scratchpad toggle | Active |
| Hyper+N | Things open window | Active |
| Hyper+S | Mouseless free mode | Active |
| Hyper+Space | *(unassigned, on DANCE_1 double-tap)* | Available |

All other Hyper+key and Meh+key combinations are available for future use.

## Implementation checklist

When ready to implement the three new combos:

### Firmware changes (`keymap.c`)

- [ ] Add `combo3[]`, `combo4[]`, `combo5[]` PROGMEM arrays
- [ ] Add entries to `key_combos[]` array
- [ ] Update `COMBO_COUNT` in config (from 3 to 6)

### Karabiner changes (if using from Russian layer)

- [ ] Add Hyper+D routing rule with `kontroll set-layer --index 0` (same pattern as Hyper+A)
- [ ] Add Hyper+F routing rule with `kontroll set-layer --index 0`
- [ ] Tana (Cmd+Shift+Space) needs no Karabiner rule

### App settings

- [ ] AquaVoice: set "Activate" shortcut to Cmd+Alt+Shift+Ctrl+D
- [ ] AquaVoice: set "Activate Hands Free" shortcut to Cmd+Alt+Shift+Ctrl+F
- [ ] Tana: verify Cmd+Shift+Space is set for Quick Add (should be default)
