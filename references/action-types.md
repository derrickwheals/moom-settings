# Moom Action Types Reference

Complete specification for all Moom action types, key codes, and modifier flags.

## Table of Contents

1. [Action 19 — Move & Resize](#action-19--move--resize)
2. [Action 1001 — Snapshot/Layout](#action-1001--snapshotlayout)
3. [Action 10001 — Group](#action-10001--group)
4. [Action 21 — Center](#action-21--center)
5. [Action 31 — Move by Delta](#action-31--move-by-delta)
6. [Action 33 — Move to Edge/Corner](#action-33--move-to-edgecorner)
7. [Action 41 — Move to Other Display](#action-41--move-to-other-display)
8. [Action 51 — Resize](#action-51--resize)
9. [Action 54 — Revert](#action-54--revert)
10. [Action 61 — Separator](#action-61--separator)
11. [Action 0 — Spacer](#action-0--spacer)
12. [Action -101 — Section Header](#action--101--section-header)
13. [Key Code Table](#key-code-table)
14. [Modifier Flags Table](#modifier-flags-table)

---

## Action 19 — Move & Resize

Moves and resizes the frontmost window to a proportional position on the current screen.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `19` |
| `Identifier` | string | UUID |
| `Relative Frame` | string | Target position as `{{x, y}, {width, height}}` with values 0.0–1.0 |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Title` | string | Display name in Moom menu |
| `Hot Key` | dict | Keyboard shortcut (see Hot Key format) |
| `Configuration Grid` | dict | Grid dimensions for Moom's visual editor |
| `Window Description` | dict | Records the window this was designed for (informational) |

### Configuration Grid Dict

```xml
<key>Configuration Grid</key>
<dict>
	<key>Configuration Grid: Columns</key>
	<integer>12</integer>
	<key>Configuration Grid: Rows</key>
	<integer>12</integer>
</dict>
```

Common grid sizes:
- `1×1` — full screen
- `2×1` — halves (horizontal)
- `1×2` — halves (vertical)
- `3×1` — thirds (horizontal)
- `1×3` — thirds (vertical, for portrait displays)
- `2×2` — quarters
- `12×12` — fine grid (most flexible, allows 1/12th increments)
- `12×6` — wide fine grid (landscape-oriented displays)
- `6×12` — tall fine grid (portrait-oriented displays)
- `18×12` — extra-wide fine grid

Each action has its own grid, so you can mix landscape (more columns) and portrait (more rows) grids within the same .moom file to target different display orientations.

### Window Description Dict

This is optional and purely informational — it records which window the action was originally designed for. Same structure as a Snapshot window entry:

```xml
<key>Window Description</key>
<dict>
	<key>Application Name</key>
	<string>Safari</string>
	<key>Available Screen Frame</key>
	<string>{{0, 69}, {3008, 1598}}</string>
	<key>Bundle Identifier</key>
	<string>com.apple.safari</string>
	<key>Screen Frame</key>
	<string>{{0, 0}, {3008, 1692}}</string>
	<key>Window Frame</key>
	<string>{{508, 210}, {1992, 1316}}</string>
	<key>Window Subrole</key>
	<string>AXStandardWindow</string>
	<key>Window Title</key>
	<string>Window Title</string>
</dict>
```

---

## Action 1001 — Snapshot/Layout

Saves and restores a complete window arrangement. Records the position of every window involved.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `1001` |
| `Identifier` | string | UUID |
| `Snapshot` | array | Array of window description dicts |
| `Snapshot Screens` | array | Array of screen frame strings |

### Standard Keys (always include these)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `Activate Applications` | boolean | `false` | Bring snapshot apps to front |
| `Apply to Active Display` | boolean | `false` | Apply only to the active display |
| `Apply to Overlapping Windows` | boolean | `true` | Also reposition overlapping windows |
| `Auto-Trigger` | boolean | `false` | Auto-trigger on display change |
| `Auto-Trigger Display Count` | integer | `1` | Number of displays to trigger on |
| `Auto-Trigger: Wake` | boolean | `false` | Also trigger on wake |
| `Generic` | boolean | varies | `true` = any window matches; `false` = app-specific matching |
| `Stubborn` | boolean | `false` | Force-match windows more aggressively |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Title` | string | Display name |
| `Hot Key` | dict | Keyboard shortcut |

### Snapshot Window Entry

Each entry in the `Snapshot` array:

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `Application Name` | string | Yes | Human-readable app name |
| `Bundle Identifier` | string | Yes | macOS bundle ID (e.g., `com.apple.safari`) |
| `Window Title` | string | Yes | Window title at capture time |
| `Window Frame` | string | Yes | Window position/size: `{{x, y}, {w, h}}` |
| `Screen Frame` | string | Yes | Full screen dimensions |
| `Available Screen Frame` | string | Yes | Usable screen area (minus menu bar, dock) |
| `Window Subrole` | string | Yes | Usually `AXStandardWindow`, sometimes `AXDialog` or `AXSystemDialog` |
| `PID` | string | No | Process ID (sometimes present in newer exports) |
| `Window Number` | string | No | Window number (sometimes present in newer exports) |

### Common Bundle Identifiers

| App | Bundle Identifier |
|-----|-------------------|
| Safari | `com.apple.safari` |
| Finder | `com.apple.finder` |
| Mail | `com.apple.mail` |
| Notes | `com.apple.notes` |
| Terminal | `com.apple.Terminal` |
| TextEdit | `com.apple.textedit` |
| Dictionary | `com.apple.dictionary` |
| Microsoft Edge | `com.microsoft.edgemac` |
| Microsoft Teams | `com.microsoft.teams2` |
| Microsoft Outlook | `com.microsoft.outlook` |
| Microsoft Word | `com.microsoft.Word` |
| Microsoft Excel | `com.microsoft.Excel` |
| Microsoft PowerPoint | `com.microsoft.Powerpoint` |
| Google Chrome | `com.google.Chrome` |
| Firefox | `org.mozilla.firefox` |
| Slack | `com.tinyspeck.slackmacgap` |
| Discord | `com.hnc.Discord` |
| Notion | `notion.id` |
| VS Code | `com.microsoft.VSCode` |
| Xcode | `com.apple.dt.Xcode` |
| iTerm2 | `com.googlecode.iterm2` |
| Warp | `dev.warp.Warp-Stable` |
| Things | `com.culturedcode.thingsmac` |
| Bear | `net.shinyfrog.bear` |
| Spotify | `com.spotify.client` |
| 1Password | `com.1password.1password` |
| Ghostty | `com.mitchellh.ghostty` |
| Typora | `abnerworks.typora` |
| CotEditor | `com.coteditor.coteditor` |
| Obsidian | `md.obsidian` |
| Arc | `company.thebrowser.Browser` |
| Cursor | `com.todesktop.230313mzl4w4u92` |

---

## Action 10001 — Group (Folder / Chain)

A container that holds child actions. Depending on configuration, it can act as a folder (organizer), a combined action (execute all at once), or a cycling chain (advance through children on repeated presses).

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `10001` |
| `Identifier` | string | UUID |
| `Chain Mode` | integer | `0` or `11` (see behavior table below) |
| `Children` | array | Array of child action dicts (can include other groups for nesting) |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Title` | string | Display name |
| `Chain` | boolean | `true` to enable chaining (combined action or cycling, depending on Chain Mode) |
| `Children Expanded` | boolean | `true` to show children expanded in Moom's UI |
| `Hot Key` | dict | Keyboard shortcut — behavior depends on group type (see below) |

### Behavior Matrix

The combination of `Chain` and `Chain Mode` determines how the group works:

| `Chain` | `Chain Mode` | Behavior | Hot Key Effect |
|---------|-------------|----------|----------------|
| absent/false | `0` | **Folder** — organizes actions in the Moom menu | Opens a sub-menu of children |
| `true` | `0` | **Combined action** — all children execute simultaneously | Triggers all children at once |
| `true` | `11` | **Cycle** — advances through children one at a time | Executes next child in sequence |

### Folder Behavior

Groups with Chain Mode 0 and no Chain key are displayed as **Folders** in Moom's UI. They can be nested arbitrarily deep (folders within folders). Assigning a hot key opens the folder as a sub-menu, letting the user pick a child action.

### Combined Action Behavior

When Chain is true and Chain Mode is 0, all children execute together in one step. Useful for composing multi-step operations — e.g., "move to next display" + "resize to left half" as a single shortcut.

### Cycle Behavior

When Chain is true and Chain Mode is 11, each hot key press advances to the next child in order, wrapping back to the first after the last. Moom tracks the current position in the cycle.

---

## Action 21 — Center

Centers the frontmost window on the current screen.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `21` |
| `Identifier` | string | UUID |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Center Mode` | integer | `1` = center on screen |
| `Hot Key` | dict | Keyboard shortcut |

---

## Action 31 — Move by Delta

Moves the frontmost window by a fixed pixel amount in a direction.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `31` |
| `Identifier` | string | UUID |
| `Move Delta` | integer | Pixels to move |
| `Move Direction` | integer | Direction code (see below) |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Move Delta Unit` | integer | `0` = pixels |
| `Confine to Display` | boolean | Keep window within display bounds |
| `Hot Key` | dict | Keyboard shortcut |

### Direction Codes

| Value | Direction |
|-------|-----------|
| `3` | Up |
| `5` | Right |
| `7` | Down |
| `9` | Left |

---

## Action 33 — Move to Edge/Corner

Moves the frontmost window to a screen edge or corner.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `33` |
| `Identifier` | string | UUID |
| `Move to Edge/Corner Direction` | integer | Direction code |

### Direction Codes

| Value | Direction |
|-------|-----------|
| `3` | Top |
| `5` | Right |
| `7` | Bottom |
| `9` | Left |

---

## Action 41 — Move to Other Display

Moves the frontmost window to another connected display.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `41` |
| `Identifier` | string | UUID |
| `Move Direction` | integer | `5` = next display (right), `9` = previous display (left) |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Loop Through Displays` | boolean | `true` to wrap around when reaching the last display |
| `Resize Proportionally` | boolean | `true` to scale window proportionally to the new display |
| `Hot Key` | dict | Keyboard shortcut |

---

## Action 51 — Resize

Resizes the frontmost window to specific pixel dimensions.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `51` |
| `Identifier` | string | UUID |
| `Resize Size` | string | Target size as `{width, height}` in pixels |

### Optional Keys

| Key | Type | Description |
|-----|------|-------------|
| `Resize Anchor` | integer | Which corner/edge stays fixed (see below) |
| `Resize Width Unit` | integer | `0` = pixels |
| `Resize Height Unit` | integer | `0` = pixels |
| `Hot Key` | dict | Keyboard shortcut |

### Resize Anchor Values

| Value | Anchor Position |
|-------|----------------|
| `10` | Top-left (default) |

---

## Action 54 — Revert

Reverts the frontmost window to its previous size and position (before the last Moom action).

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `54` |
| `Identifier` | string | UUID |

No other keys needed.

---

## Action 61 — Separator

A visual separator line in the Moom custom controls menu.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `61` |
| `Identifier` | string | UUID |

---

## Action 0 — Spacer

An empty space in the Moom custom controls menu.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `0` |
| `Identifier` | string | UUID |

---

## Action -101 — Section Header

A text label that acts as a section header in the Moom custom controls menu.

### Required Keys

| Key | Type | Description |
|-----|------|-------------|
| `Action` | integer | `-101` |
| `Identifier` | string | UUID |
| `Title` | string | Header text |

---

## Key Code Table

macOS virtual key codes used in the `Key Code` field of Hot Key dicts.

### Letters

| Key | Code | | Key | Code | | Key | Code |
|-----|------|-|-----|------|-|-----|------|
| A | 0 | | J | 38 | | S | 1 |
| B | 11 | | K | 40 | | T | 17 |
| C | 8 | | L | 37 | | U | 32 |
| D | 2 | | M | 46 | | V | 9 |
| E | 14 | | N | 45 | | W | 13 |
| F | 3 | | O | 31 | | X | 7 |
| G | 5 | | P | 35 | | Y | 16 |
| H | 4 | | Q | 12 | | Z | 6 |
| I | 34 | | R | 15 | | | |

### Numbers

| Key | Code | | Key | Code |
|-----|------|-|-----|------|
| 1 | 18 | | 6 | 22 |
| 2 | 19 | | 7 | 26 |
| 3 | 20 | | 8 | 28 |
| 4 | 21 | | 9 | 25 |
| 5 | 23 | | 0 | 29 |

### Special Keys

| Key | Code |
|-----|------|
| Return/Enter | 36 |
| Tab | 48 |
| Space | 49 |
| Delete/Backspace | 51 |
| Escape | 53 |
| Left Arrow | 123 |
| Right Arrow | 124 |
| Down Arrow | 125 |
| Up Arrow | 126 |

### Punctuation & Symbols

| Key | Code | | Key | Code |
|-----|------|-|-----|------|
| = | 24 | | - | 27 |
| ] | 30 | | [ | 33 |
| ' | 39 | | ; | 41 |
| \ | 42 | | , | 43 |
| / | 44 | | . | 47 |
| ` | 50 | | | |

---

## Modifier Flags Table

Modifier flags are integer values that encode which modifier keys are held. The values differ for regular keys vs. arrow keys because arrows include additional system flags (Function and NumericPad).

Values marked **confirmed** were directly observed in real Moom export files. Values marked **inferred** were calculated from the confirmed values by following the bit-pattern logic, but have not been verified against an actual Moom export. Inferred values should work correctly, but if you encounter issues, prefer the confirmed combinations.

### For Letter, Number, and Return Keys

| Modifiers | Symbol | Flags Value | Status |
|-----------|--------|-------------|--------|
| Control + Option | ⌃⌥ | 786721 | confirmed |
| Control + Option + Shift | ⌃⌥⇧ | 917795 | confirmed |
| Control + Option + Command | ⌃⌥⌘ | 1835305 | confirmed |
| Control + Command | ⌃⌘ | 1310985 | confirmed |
| Control + Option + Shift + Command | ⌃⌥⇧⌘ | 1966379 | inferred |
| Control + Shift | ⌃⇧ | 393507 | inferred |
| Option + Command | ⌥⌘ | 1573161 | inferred |
| Option + Shift | ⌥⇧ | 655395 | inferred |
| Command + Shift | ⌘⇧ | 1179915 | inferred |
| Control | ⌃ | 262433 | inferred |
| Option | ⌥ | 524585 | inferred |
| Command | ⌘ | 1048849 | inferred |
| Shift | ⇧ | 131331 | inferred |

### For Arrow Keys (Left, Right, Up, Down)

Arrow keys include the Function (Fn) and NumericPad flags automatically.

| Modifiers | Symbol | Flags Value | Status |
|-----------|--------|-------------|--------|
| Control + Option | ⌃⌥ | 11272481 | confirmed |
| Control + Option + Shift | ⌃⌥⇧ | 11403555 | confirmed |
| Control + Option + Command | ⌃⌥⌘ | 12321065 | confirmed |
| Control + Option + Shift + Command | ⌃⌥⇧⌘ | 12452139 | inferred |
| Control + Command | ⌃⌘ | 11796745 | inferred |

### Visual Representation Symbols

| Modifier | Symbol |
|----------|--------|
| Control | ⌃ |
| Option/Alt | ⌥ |
| Shift | ⇧ |
| Command | ⌘ |
| Return | ↵ |
| Left Arrow | ← |
| Right Arrow | → |
| Up Arrow | ↑ |
| Down Arrow | ↓ |

The Visual Representation string concatenates modifier symbols in order (⌃⌥⇧⌘) followed by the key character. Examples: `⌃⌥J`, `⌃⌥⇧←`, `⌃⌘↵`.
