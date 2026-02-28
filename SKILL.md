---
name: moom-settings
description: |
  Create, edit, read, and understand Moom window management settings files (.moom) for macOS. Moom is a popular macOS app for managing window positions, sizes, and layouts using keyboard shortcuts, snap areas, and saved configurations. Use this skill whenever the user mentions Moom, .moom files, window layout configurations, window snapping setups, Moom keyboard shortcuts, Moom snapshots, Moom chains, or wants to programmatically create or modify window management presets. Also use when the user has a .moom or plist file related to window management, or wants to set up screen layouts, tiling configurations, or multi-display window arrangements for Moom. This skill can also suggest common window configurations and keyboard shortcut conventions, and interpret plain language descriptions of window positions (e.g., "left half", "top-right corner", "60/40 split", "main content with sidebar") to generate the correct Moom settings.
---

# Moom Settings Editor

This skill enables you to create and edit Moom settings files (.moom). These are XML plist files that Moom can import to configure window management actions — things like "snap this window to the left half of the screen" or "arrange these three apps in a specific layout."

## Quick Start

A .moom file is an XML plist containing a single top-level dictionary with one key: `Custom Controls (4001)`, whose value is an array of action dictionaries.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Custom Controls (4001)</key>
	<array>
		<!-- Action dictionaries go here -->
	</array>
</dict>
</plist>
```

Every action dictionary **must** have an `Action` key (integer code) and an `Identifier` key (UUID string). The action code determines the type and which additional keys are valid.

## Core Concepts

### Action Types at a Glance

| Code | Type | What it does |
|------|------|-------------|
| 19 | Move & Resize | Positions a window to a relative frame on screen |
| 1001 | Snapshot/Layout | Saves and restores a full window arrangement |
| 10001 | Group / Folder / Chain | Container: folder (organizer), combined action, or cycling chain |
| 21 | Center | Centers a window on screen |
| 31 | Move by Delta | Nudges a window by a pixel amount |
| 33 | Move to Edge/Corner | Snaps a window to a screen edge or corner |
| 41 | Move to Other Display | Sends a window to another monitor |
| 51 | Resize | Resizes a window to specific dimensions |
| 54 | Revert | Reverts a window to its previous size/position |
| 61 | Separator Line | Visual separator in the Moom menu |
| 0 | Spacer | Empty space in the Moom menu |
| -101 | Section Header | Text label/header in the Moom menu |

For the full specification of each action type and all their properties, read `references/action-types.md` in the skill directory.

### Identifiers

Every action needs a unique UUID. Generate standard v4 UUIDs in uppercase with hyphens:
```
<key>Identifier</key>
<string>A1B2C3D4-E5F6-7890-ABCD-EF1234567890</string>
```

When an action has a Hot Key, the Hot Key's Identifier should match the action's Identifier.

### Frame Strings

Moom uses a specific string format for rectangles, borrowed from macOS's `NSStringFromRect`:
```
{{x, y}, {width, height}}
```

- **Relative Frames** (Action 19): Values are 0.0 to 1.0, representing proportions of the screen. Origin is **bottom-left**, with y increasing upward (matching macOS's Quartz coordinate system). `{{0, 0}, {0.5, 1}}` means "left half of screen." `{{0, 0.5}, {0.5, 0.5}}` means "top-left quarter."
- **Absolute Frames** (Snapshots): Values are pixel coordinates. `{{0, 50}, {1920, 1030}}` means "starting at pixel (0, 50), 1920px wide, 1030px tall."

Common relative frames (remember: y=0 is the **bottom** of the screen, y=1 is the top):
| Position | Frame |
|----------|-------|
| Left half | `{{0, 0}, {0.5, 1}}` |
| Right half | `{{0.5, 0}, {0.5, 1}}` |
| Top half | `{{0, 0.5}, {1, 0.5}}` |
| Bottom half | `{{0, 0}, {1, 0.5}}` |
| Left third | `{{0, 0}, {0.33333333333333331, 1}}` |
| Middle third | `{{0.33333333333333331, 0}, {0.33333333333333331, 1}}` |
| Right third | `{{0.66666666666666663, 0}, {0.33333333333333337, 1}}` |
| Left two-thirds | `{{0, 0}, {0.66666666666666663, 1}}` |
| Right two-thirds | `{{0.33333333333333331, 0}, {0.66666666666666674, 1}}` |
| Full screen | `{{0, 0}, {1, 1}}` |
| Top-left quarter | `{{0, 0.5}, {0.5, 0.5}}` |
| Top-right quarter | `{{0.5, 0.5}, {0.5, 0.5}}` |
| Bottom-left quarter | `{{0, 0}, {0.5, 0.5}}` |
| Bottom-right quarter | `{{0.5, 0}, {0.5, 0.5}}` |

Moom uses full double-precision values (e.g., `0.33333333333333331` not `0.333`). Always use full precision for thirds and other fractional values to match Moom's expectations.

### Hot Keys

Hot keys are optional dictionaries that bind a keyboard shortcut to an action. See `references/action-types.md` for the complete key code and modifier flags lookup tables.

```xml
<key>Hot Key</key>
<dict>
	<key>Identifier</key>
	<string>SAME-UUID-AS-ACTION</string>
	<key>Key Code</key>
	<integer>38</integer>
	<key>Modifier Flags</key>
	<integer>786721</integer>
	<key>Visual Representation</key>
	<string>⌃⌥J</string>
</dict>
```

Common modifier flag values for letter/number keys:

| Modifiers | Flags | Symbols |
|-----------|-------|---------|
| Control + Option | 786721 | ⌃⌥ |
| Control + Option + Shift | 917795 | ⌃⌥⇧ |
| Control + Option + Command | 1835305 | ⌃⌥⌘ |
| Control + Command | 1310985 | ⌃⌘ |

For arrow keys, the flags are different (they include Function and NumericPad bits):

| Modifiers | Flags | Symbols |
|-----------|-------|---------|
| Control + Option | 11272481 | ⌃⌥ |
| Control + Option + Shift | 11403555 | ⌃⌥⇧ |
| Control + Option + Command | 12321065 | ⌃⌥⌘ |

The Visual Representation uses macOS modifier symbols: ⌃ (Control), ⌥ (Option), ⇧ (Shift), ⌘ (Command).

## Common Patterns

### Simple Move & Resize (Action 19)

The most common action. Positions a window to a portion of the screen.

```xml
<dict>
	<key>Action</key>
	<integer>19</integer>
	<key>Configuration Grid</key>
	<dict>
		<key>Configuration Grid: Columns</key>
		<integer>12</integer>
		<key>Configuration Grid: Rows</key>
		<integer>12</integer>
	</dict>
	<key>Hot Key</key>
	<dict>
		<key>Identifier</key>
		<string>YOUR-UUID-HERE</string>
		<key>Key Code</key>
		<integer>123</integer>
		<key>Modifier Flags</key>
		<integer>11272481</integer>
		<key>Visual Representation</key>
		<string>⌃⌥←</string>
	</dict>
	<key>Identifier</key>
	<string>YOUR-UUID-HERE</string>
	<key>Relative Frame</key>
	<string>{{0, 0}, {0.49999999999999994, 1}}</string>
	<key>Title</key>
	<string>Left Half</string>
</dict>
```

The Configuration Grid tells Moom's visual grid editor how many columns and rows the grid has. For halves, a 2x1 or 12x12 grid works. For thirds, use 3x1. For quarters, use 2x2 or 12x12. A 12x12 grid is the most flexible and allows fine positioning.

Each action can have its own grid size, so you can mix and match. This is particularly useful when targeting both landscape and portrait displays — use a wider grid (e.g., 12x6) for landscape-oriented actions and a taller grid (e.g., 6x12) for portrait-oriented actions.

When no Configuration Grid is specified, Moom uses a default grid. It's fine to omit it for simple proportional frames.

### Groups, Folders, and Chains (Action 10001)

Groups are containers that hold child actions. They serve three distinct purposes depending on their configuration:

#### Folders (`Chain Mode: 0`, no hot key or hot key opens sub-menu)

When a group has `Chain Mode: 0`, it acts as a **Folder** in Moom's UI. Folders organize actions into collapsible sections in the Moom menu — useful when you have many custom actions and need to keep things tidy.

- Folders can be **nested** arbitrarily deep (folders within folders within folders)
- Assigning a hot key to a folder opens it as a **sub-menu** — press the shortcut and a list of the folder's children appears, letting you pick one
- Without a hot key, the folder just groups actions visually in the Moom custom controls list

```xml
<dict>
	<key>Action</key>
	<integer>10001</integer>
	<key>Chain Mode</key>
	<integer>0</integer>
	<key>Children</key>
	<array>
		<!-- Child action dicts here (can include other groups for nesting) -->
	</array>
	<key>Children Expanded</key>
	<true/>
	<key>Identifier</key>
	<string>FOLDER-UUID</string>
	<key>Title</key>
	<string>My Window Shortcuts</string>
</dict>
```

#### Chain — Cycle Mode (`Chain Mode: 11`)

When `Chain` is `true` and `Chain Mode` is `11`, pressing the hot key repeatedly **cycles** through the children in order. This is great for "press once for left half, again for left third, again for left two-thirds."

Moom tracks which child is "current" so each press advances to the next one, wrapping back to the first after the last.

```xml
<dict>
	<key>Action</key>
	<integer>10001</integer>
	<key>Chain</key>
	<true/>
	<key>Chain Mode</key>
	<integer>11</integer>
	<key>Children</key>
	<array>
		<!-- Child action dicts here -->
	</array>
	<key>Children Expanded</key>
	<true/>
	<key>Hot Key</key>
	<dict>
		<key>Identifier</key>
		<string>GROUP-UUID</string>
		<key>Key Code</key>
		<integer>123</integer>
		<key>Modifier Flags</key>
		<integer>11403555</integer>
		<key>Visual Representation</key>
		<string>⌃⌥⇧←</string>
	</dict>
	<key>Identifier</key>
	<string>GROUP-UUID</string>
	<key>Title</key>
	<string>Left</string>
</dict>
```

#### Chain — Combined Action Mode (`Chain Mode: 0` with `Chain: true`)

When `Chain` is `true` but `Chain Mode` is `0`, pressing the hot key **executes all children at once** as a single combined action. This lets you compose multi-step operations into one shortcut — for example, moving a window to another display *and* resizing it in one press.

```xml
<dict>
	<key>Action</key>
	<integer>10001</integer>
	<key>Chain</key>
	<true/>
	<key>Chain Mode</key>
	<integer>0</integer>
	<key>Children</key>
	<array>
		<!-- All children execute together in one step -->
	</array>
	<key>Children Expanded</key>
	<true/>
	<key>Hot Key</key>
	<dict>
		<key>Identifier</key>
		<string>GROUP-UUID</string>
		<key>Key Code</key>
		<integer>41</integer>
		<key>Modifier Flags</key>
		<integer>786721</integer>
		<key>Visual Representation</key>
		<string>⌃⌥;</string>
	</dict>
	<key>Identifier</key>
	<string>GROUP-UUID</string>
	<key>Title</key>
	<string>Move to External &amp; Maximize</string>
</dict>
```

A practical example: combine "Move to Other Display" (Action 41) + "Move & Resize to left half" (Action 19) into one shortcut that sends a window to the next monitor and snaps it to the left half in a single key press.

#### Summary of Group Modes

| `Chain` | `Chain Mode` | Behavior |
|---------|-------------|----------|
| absent/false | `0` | **Folder** — organizer only, hot key opens sub-menu |
| `true` | `0` | **Combined action** — all children execute at once |
| `true` | `11` | **Cycle** — each press advances to the next child |

- `Children Expanded: true` = children are shown expanded in Moom's UI
- Groups/folders can contain any action type as children, including other groups (enabling nesting)

### Layouts (Action 1001 — Snapshots)

Layouts are one of Moom's most powerful features. They save a complete window arrangement — every window's app, position, and size — and restore it with one shortcut. Moom calls these "Snapshots" internally (Action 1001), but users know them as "Saved Layouts."

Unlike Move & Resize actions (which use relative 0.0–1.0 frames), layouts use **absolute pixel coordinates** because they record exact window positions on a specific screen.

#### Two Types of Layouts

**App-Specific Layouts** (`Generic: false`) — "Save Layouts of Windows in Apps"

These match windows by bundle identifier. When activated, Moom finds windows belonging to each listed app and moves them to the saved positions. This is the most common type.

- Set `Generic: false` and `Activate Applications: true`
- Moom matches each snapshot entry to a window from the same app
- The same app can have multiple windows in one layout (e.g., two editor windows side by side, or a main window plus a dialog)
- Best for workflows like "put VS Code on the left, terminal bottom-right, browser top-right"

**Generic / "Any Window" Layouts** (`Generic: true`) — "Save Layouts of Any Windows"

These ignore which app owns each window. When activated, Moom takes the N most recently used windows and arranges them into the saved positions — completely app-independent.

- Set `Generic: true` and `Activate Applications: false`
- The number of windows affected equals the number of entries in the Snapshot array
- Best for reusable arrangements like "tile my 4 most recent windows in a grid"

#### Screen Dimensions

Layouts need real screen dimensions. Every window entry includes `Screen Frame` (total display) and `Available Screen Frame` (usable area minus menu bar and dock). These must match the user's actual display.

Common Mac screen dimensions (at default scaled resolution):

| Display | Screen Frame | Available Screen Frame |
|---------|-------------|----------------------|
| MacBook Air 13" | `{{0, 0}, {1470, 956}}` | `{{0, 37}, {1470, 886}}` |
| MacBook Air 15" | `{{0, 0}, {1710, 1112}}` | `{{0, 37}, {1710, 1042}}` |
| MacBook Pro 14" | `{{0, 0}, {1512, 982}}` | `{{0, 53}, {1512, 896}}` |
| MacBook Pro 16" | `{{0, 0}, {1728, 1117}}` | `{{0, 53}, {1728, 1031}}` |
| 27" 4K / Studio Display | `{{0, 0}, {2560, 1440}}` | `{{0, 37}, {2560, 1370}}` |

The Available Screen Frame depends on dock position, dock size, and whether it auto-hides. The values above assume a dock at the bottom. If the user has previously exported a .moom file, read it to get their exact screen dimensions from an existing snapshot entry — this is always the most reliable approach.

The Available Screen Frame tells you: `{{origin_x, origin_y}, {width, height}}`. The origin_y value is the bottom edge of the usable area (above the dock in Quartz coordinates), and origin_y + height is the top edge (below the menu bar).

#### Calculating Window Positions

Convert the user's descriptions to pixel coordinates within the Available Screen Frame. Given available frame `{{ax, ay}, {aw, ah}}`:

| Position | Window Frame |
|----------|-------------|
| Full available area | `{{ax, ay}, {aw, ah}}` |
| Left half | `{{ax, ay}, {floor(aw/2), ah}}` |
| Right half | `{{ax + ceil(aw/2), ay}, {floor(aw/2), ah}}` |
| Top half | `{{ax, ay + ceil(ah/2)}, {aw, floor(ah/2)}}` |
| Bottom half | `{{ax, ay}, {aw, floor(ah/2)}}` |
| Top-left quarter | `{{ax, ay + ceil(ah/2)}, {floor(aw/2), floor(ah/2)}}` |
| Top-right quarter | `{{ax + ceil(aw/2), ay + ceil(ah/2)}, {floor(aw/2), floor(ah/2)}}` |
| Bottom-left quarter | `{{ax, ay}, {floor(aw/2), floor(ah/2)}}` |
| Bottom-right quarter | `{{ax + ceil(aw/2), ay}, {floor(aw/2), floor(ah/2)}}` |
| Left two-thirds | `{{ax, ay}, {floor(aw*2/3), ah}}` |
| Right third | `{{ax + floor(aw*2/3), ay}, {aw - floor(aw*2/3), ah}}` |

All values are integers (pixel coordinates). Use floor/ceil to avoid sub-pixel gaps. Remember: y=0 is the bottom of the screen (Quartz coordinates), so "top" means higher y values.

#### Building a Layout Step by Step

1. **Determine the type** — app-specific (`Generic: false`) or any-window (`Generic: true`)
2. **Get screen dimensions** — from the user or from an existing .moom export
3. **Calculate window positions** — convert descriptions to pixel coordinates using the table above
4. **Look up bundle identifiers** — for app-specific layouts, find each app's bundle ID in `references/action-types.md`
5. **Set the Window Subrole** — use `AXStandardWindow` for normal windows, `AXDialog` for dialog/utility windows
6. **Provide a Window Title** — a representative title for the window (e.g., "Untitled" or the app name)

#### Example: App-Specific Layout

This layout arranges three apps on a MacBook Pro 14" (1512x982): editor on the left half, terminal in the bottom-right quarter, and a reference browser in the top-right quarter.

```xml
<dict>
	<key>Action</key>
	<integer>1001</integer>
	<key>Activate Applications</key>
	<true/>
	<key>Apply to Active Display</key>
	<false/>
	<key>Apply to Overlapping Windows</key>
	<true/>
	<key>Auto-Trigger</key>
	<false/>
	<key>Auto-Trigger Display Count</key>
	<integer>1</integer>
	<key>Auto-Trigger: Wake</key>
	<false/>
	<key>Generic</key>
	<false/>
	<key>Identifier</key>
	<string>YOUR-UUID-HERE</string>
	<key>Snapshot</key>
	<array>
		<dict>
			<key>Application Name</key>
			<string>Visual Studio Code</string>
			<key>Available Screen Frame</key>
			<string>{{0, 53}, {1512, 896}}</string>
			<key>Bundle Identifier</key>
			<string>com.microsoft.VSCode</string>
			<key>Screen Frame</key>
			<string>{{0, 0}, {1512, 982}}</string>
			<key>Window Frame</key>
			<string>{{0, 53}, {756, 896}}</string>
			<key>Window Subrole</key>
			<string>AXStandardWindow</string>
			<key>Window Title</key>
			<string>Untitled</string>
		</dict>
		<dict>
			<key>Application Name</key>
			<string>Ghostty</string>
			<key>Available Screen Frame</key>
			<string>{{0, 53}, {1512, 896}}</string>
			<key>Bundle Identifier</key>
			<string>com.mitchellh.ghostty</string>
			<key>Screen Frame</key>
			<string>{{0, 0}, {1512, 982}}</string>
			<key>Window Frame</key>
			<string>{{756, 53}, {756, 448}}</string>
			<key>Window Subrole</key>
			<string>AXStandardWindow</string>
			<key>Window Title</key>
			<string>Terminal</string>
		</dict>
		<dict>
			<key>Application Name</key>
			<string>Safari</string>
			<key>Available Screen Frame</key>
			<string>{{0, 53}, {1512, 896}}</string>
			<key>Bundle Identifier</key>
			<string>com.apple.safari</string>
			<key>Screen Frame</key>
			<string>{{0, 0}, {1512, 982}}</string>
			<key>Window Frame</key>
			<string>{{756, 501}, {756, 448}}</string>
			<key>Window Subrole</key>
			<string>AXStandardWindow</string>
			<key>Window Title</key>
			<string>Untitled</string>
		</dict>
	</array>
	<key>Snapshot Screens</key>
	<array>
		<string>{{0, 0}, {1512, 982}}</string>
	</array>
	<key>Stubborn</key>
	<false/>
	<key>Title</key>
	<string>Coding Layout</string>
</dict>
```

The `Snapshot Screens` array lists the screen frames at capture time. Include one entry per display involved in the layout. This helps Moom adapt when displays change.

`PID` and `Window Number` keys appear in real Moom exports (captured from live windows) but are **not required** for importing — Moom ignores them during import. You do not need to include them when creating layouts from scratch.

#### Additional Layout Properties

- `Stubborn: false` (default) — standard window matching heuristics
- `Stubborn: true` — forces more aggressive matching (useful when Moom can't find the right window)
- `Apply to Overlapping Windows: true` — also repositions windows that overlap with snapshot positions
- `Apply to Active Display: true` — restricts the layout to only affect windows on the currently active display

#### Auto-Trigger Layouts

Layouts can automatically activate when the display configuration changes — ideal for docking/undocking a laptop:

- `Auto-Trigger: true` — enables auto-triggering
- `Auto-Trigger Display Count: N` — triggers when exactly N displays are connected
- `Auto-Trigger: Wake: true` — also triggers on wake from sleep

A typical setup: one layout auto-triggers with display count 1 (laptop only) and another with count 2 (laptop + external monitor), so windows rearrange automatically when you dock or undock.

## Working with Existing Files

When reading a .moom file:
1. Parse it as XML plist
2. The top-level array under `Custom Controls (4001)` contains all actions
3. Groups (Action 10001) contain nested actions in their `Children` array
4. Look at `Title` keys to understand what each action does
5. Look at `Hot Key` dicts to see keyboard shortcuts
6. Look at `Action` codes to understand the action type

### Backup Before Editing

Before making any changes to an existing .moom file, create a date-stamped backup copy in the same directory. This protects the user's working configuration in case an edit introduces problems — Moom settings can be complex and interdependent, and a bad edit could break an entire layout setup.

Use the format `<original-name>-backup-YYYY-MM-DD-HHMMSS.moom`. For example, editing `Actions.moom` on 2026-02-28 at 3:45:12 PM would produce:

```
Actions-backup-2026-02-28-154512.moom
```

Create the backup by copying the original file before making any modifications:

```bash
cp "Actions.moom" "Actions-backup-$(date +%Y-%m-%d-%H%M%S).moom"
```

This step is not optional — always create the backup before editing, even for small changes. Confirm the backup was created successfully before proceeding with edits. If the user explicitly says they don't want a backup, that's fine, but default to creating one.

### Editing Guidelines

When editing:
- Preserve existing UUIDs — changing them can break references
- Maintain proper XML plist formatting with tab indentation
- Keep the `<?xml ...?>` declaration and `<!DOCTYPE ...>` exactly as-is
- Use `&amp;` for ampersands in titles (standard XML escaping)

## Interpreting Plain Language Descriptions

Users will describe window positions in natural language. Map their descriptions to the correct Moom relative frames using this guide. Remember: y=0 is the **bottom** of the screen, y increases upward.

### Position Vocabulary

These phrases all mean the same thing — map them to the corresponding frame:

| User says | Moom Relative Frame |
|-----------|-------------------|
| "left half", "left side", "left 50%", "snap left" | `{{0, 0}, {0.5, 1}}` |
| "right half", "right side", "right 50%", "snap right" | `{{0.5, 0}, {0.5, 1}}` |
| "top half", "upper half", "snap up" | `{{0, 0.5}, {1, 0.5}}` |
| "bottom half", "lower half", "snap down" | `{{0, 0}, {1, 0.5}}` |
| "left third", "left 1/3" | `{{0, 0}, {0.33333333333333331, 1}}` |
| "center third", "middle third" | `{{0.33333333333333331, 0}, {0.33333333333333331, 1}}` |
| "right third", "right 1/3" | `{{0.66666666666666663, 0}, {0.33333333333333337, 1}}` |
| "left two-thirds", "left 2/3" | `{{0, 0}, {0.66666666666666663, 1}}` |
| "right two-thirds", "right 2/3" | `{{0.33333333333333331, 0}, {0.66666666666666674, 1}}` |
| "top-left quarter", "top-left corner", "upper-left" | `{{0, 0.5}, {0.5, 0.5}}` |
| "top-right quarter", "top-right corner", "upper-right" | `{{0.5, 0.5}, {0.5, 0.5}}` |
| "bottom-left quarter", "bottom-left corner", "lower-left" | `{{0, 0}, {0.5, 0.5}}` |
| "bottom-right quarter", "bottom-right corner", "lower-right" | `{{0.5, 0}, {0.5, 0.5}}` |
| "full screen", "maximize", "fill the screen" | `{{0, 0}, {1, 1}}` |
| "centered", "center of the screen" | Use Action 21 (Center) instead |
| "almost full", "nearly maximized" | `{{0.1111111111111111, 0}, {0.77777777777777812, 1}}` |
| "comfortable", "reading size" | `{{0.16666666666666666, 0.083333333333333329}, {0.66666666666666663, 0.83333333333333337}}` |

### Custom Fractions

When the user asks for non-standard splits (e.g., "60/40 split", "three-quarter width"), calculate the relative frame:
- "60% left": `{{0, 0}, {0.6, 1}}`
- "40% right": `{{0.6, 0}, {0.4, 1}}`
- "top 75%": `{{0, 0.25}, {1, 0.75}}`
- "small sidebar on the right": `{{0.75, 0}, {0.25, 1}}`

The formula is: `{{start_x, start_y}, {width, height}}` where all values are proportions from 0.0 to 1.0, and y=0 is the bottom.

### Multi-Window Descriptions

When users describe layouts involving multiple windows (e.g., "browser on the left, terminal on the right"), decide which approach fits:

- **Layout (Action 1001)** — if they want specific apps arranged in specific positions and restored as a group. Use an app-specific layout (`Generic: false`) when they name the apps, or a generic layout (`Generic: true`) when they just want "my recent windows arranged like this." Layouts use absolute pixel coordinates — see the "Layouts" section above for the full construction workflow.
- **Group of Move & Resize actions (Action 10001 + Action 19)** — if they want reusable keyboard shortcuts that position any frontmost window. This gives individual shortcuts per position rather than a single "restore everything" action.

If the user says something like "set up a coding layout with VS Code, terminal, and browser," that's a layout. If they say "I want shortcuts for left half and right half," that's Move & Resize actions.

## Suggested Configurations

When a user asks for help setting up Moom, or wants suggestions for a good starting configuration, offer these commonly used presets. These follow conventions popularised by window managers like Rectangle, Magnet, and Spectacle, so they'll feel familiar.

### Starter Pack — Essential Shortcuts

A minimal set that covers the most common needs:

| Action | Shortcut | Description |
|--------|----------|-------------|
| Left Half | ⌃⌥← | Snap window to left half |
| Right Half | ⌃⌥→ | Snap window to right half |
| Top Half | ⌃⌥↑ | Snap window to top half |
| Bottom Half | ⌃⌥↓ | Snap window to bottom half |
| Maximize | ⌃⌥↵ | Fill the entire screen |
| Center | ⌃⌥C | Center window without resizing |
| Next Display | ⌃⌥⌘→ | Move window to next monitor |
| Previous Display | ⌃⌥⌘← | Move window to previous monitor |

### Extended Pack — Thirds and Quarters

Builds on the starter pack for users who want finer control:

| Action | Shortcut | Description |
|--------|----------|-------------|
| Cycle Thirds | ⌃⌥⇧T | Press repeatedly: left third → center third → right third |
| Left Two-Thirds | ⌃⌥E | Main content area (pairs with right third sidebar) |
| Right Third | ⌃⌥N | Sidebar (pairs with left two-thirds) |
| Top-Left Quarter | ⌃⌥U | Upper-left quadrant |
| Top-Right Quarter | ⌃⌥I | Upper-right quadrant |
| Bottom-Left Quarter | ⌃⌥J | Lower-left quadrant |
| Bottom-Right Quarter | ⌃⌥K | Lower-right quadrant |

The quarter shortcuts use a spatial mnemonic: U/I are upper (top row of the right hand on QWERTY) and J/K are lower (bottom row), while the left-hand column (U/J) maps to the left side and the right-hand column (I/K) maps to the right side.

### Power User Pack — Chains with Cycling

For users who want one shortcut that adapts. Each chain cycles through related positions on repeated presses:

| Chain Group | Shortcut | Cycles through |
|-------------|----------|---------------|
| Left | ⌃⌥⇧← | Left half → Left third → Left two-thirds |
| Right | ⌃⌥⇧→ | Right half → Right third → Right two-thirds |
| Maximize | ⌃⌥⇧↵ | Maximize → Almost maximize → Comfortable |
| Quarters | ⌃⌥⇧Q | TL quarter → TR quarter → BL quarter → BR quarter |

### Productivity Presets

Common multi-window setups for specific workflows:

- **Side-by-side** (research, comparison): Two windows at left/right halves — ⌃⌥← and ⌃⌥→
- **Main + sidebar** (coding, writing): Left two-thirds for main content, right third for reference — ⌃⌥E and ⌃⌥N
- **Quad grid** (monitoring, dashboards): Four windows in each quarter — ⌃⌥U, ⌃⌥I, ⌃⌥J, ⌃⌥K
- **Stacked** (reading, email): Top half and bottom half — ⌃⌥↑ and ⌃⌥↓
- **Presentation mode**: Main content in left two-thirds at 75% height, leaving room for notes — custom relative frame

When suggesting configurations, ask the user about their typical workflow and display setup (single monitor, dual monitors, ultrawide) and whether they use Stage Manager, so you can tailor the recommendations and include Stage Manager variants where appropriate.

## Validation

After creating or editing any .moom file, always validate it before telling the user it's ready. Run:

```bash
plutil -lint "path/to/file.moom"
```

This checks that the file is well-formed XML plist. If it reports errors, fix them before proceeding. A file that fails `plutil -lint` will also fail to import in Moom, so this is a quick safety net that catches malformed XML, unclosed tags, encoding issues, and other structural problems. Do not skip this step.

## Stage Manager Support

macOS Stage Manager reserves space on the left edge of the screen for its app sidebar. Standard window positions that start at x=0 will sit behind the sidebar, which looks wrong. To support Stage Manager, offset the x position by 1/12 (0.083333...) and reduce the width accordingly.

Common Stage Manager-adjusted frames:

| Position | Standard Frame | Stage Manager Frame |
|----------|---------------|-------------------|
| Left half | `{{0, 0}, {0.5, 1}}` | `{{0.083333333333333329, 0}, {0.41666666666666663, 1}}` |
| Left two-thirds | `{{0, 0}, {0.66666666666666663, 1}}` | `{{0.083333333333333329, 0}, {0.58333333333333326, 1}}` |
| Full screen | `{{0, 0}, {1, 1}}` | `{{0.083333333333333329, 0}, {0.91666666666666663, 1}}` |

The pattern: take the standard frame, add `0.083333333333333329` to the x position, and subtract the same from the width. Height and y position stay the same. A 12x12 Configuration Grid works well for these since 1/12 aligns exactly to a grid cell.

A good approach is to create chain groups that cycle between the standard and Stage Manager variants of the same position, so the user can press the shortcut once for the standard layout and again for the Stage Manager version. For example:

```
⌃⌥↵ cycles: Maximize → Maximize (Stage Manager)
⌃⌥E cycles: Left two-thirds → Left two-thirds (Stage Manager)
```

When users mention Stage Manager, or when you know they use it, include these adjusted variants automatically.

## Formatting Rules

Moom expects well-formed XML plist files. Follow these rules:
- Use tabs for indentation, matching the nesting depth
- Boolean values are `<true/>` or `<false/>` (self-closing tags)
- Integer values use `<integer>N</integer>`
- String values use `<string>text</string>`
- The plist must start with the XML declaration and DOCTYPE
- Keys within a dict should follow Moom's expected ordering (Action first, then alphabetical for the rest, with Identifier near the end)

## File Extension

Moom settings files use the `.moom` extension. Users can import them via Moom's preferences (Custom tab > Import).
