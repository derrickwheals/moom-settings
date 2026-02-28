# Moom Settings Editor

A Claude Code skill for creating, editing, and managing [Moom](https://manytricks.com/moom/) window management configuration files (.moom) on macOS.

> **Disclaimer:** This skill and its creator are in no way affiliated with, endorsed by, or associated with [Many Tricks](https://manytricks.com/), the developer of Moom. This is an independent, community-created tool built by reverse-engineering the .moom file format. Moom is a product of Many Tricks and all trademarks belong to their respective owners.

## What This Skill Does

- **Creates** new .moom configuration files from plain language descriptions (e.g., "editor on the left two-thirds, terminal on the right")
- **Reads** existing .moom files and explains what each action and keyboard shortcut does
- **Edits** existing .moom files to add, modify, or remove window configurations
- **Suggests** common window layouts and keyboard shortcut conventions based on popular window management patterns
- **Interprets** natural language position descriptions and converts them to valid Moom configuration
- **Supports Stage Manager** layouts with adjusted window positions that account for the sidebar

The skill understands Moom's full plist file format, including Move & Resize actions, Snapshots/Layouts (both app-specific and generic "any window"), Chain Groups, Folders, combined-action chains, multi-display setups, and all supported action types.

## Prerequisites

- [Moom](https://manytricks.com/moom/) installed on your Mac
- [Claude Code](https://claude.com/claude-code) CLI tool

## Installation

Copy the `moom-settings` skill folder into your Claude Code skills directory:

```bash
cp -r moom-settings ~/.claude/skills/moom-settings
```

The skill will be automatically available in your next Claude Code session.

## Usage

There are two main workflows for using this skill: creating a brand new set of configurations, or editing your existing Moom settings.

---

### Workflow 1: Create New Configurations

Use this when you're starting fresh or want to generate a standalone set of window shortcuts.

#### Step 1 — Describe what you want

Tell Claude what window configurations you need. You can be as specific or as general as you like:

```
Create a Moom config with left half, right half, and maximize shortcuts
```

```
Set up Moom for a coding workflow with my editor taking up the left
two-thirds and a terminal on the right third
```

```
I want quarter-screen shortcuts that cycle through all four corners
when I press the same key repeatedly
```

You can also ask for suggestions:

```
What's a good starter set of Moom shortcuts?
```

Claude will generate a `.moom` file with the appropriate actions, keyboard shortcuts, and settings.

#### Step 2 — Import into Moom

1. Open **Moom** preferences (click the Moom menu bar icon, then **Preferences**, or use the keyboard shortcut)
2. Go to the **Custom** tab
3. Click the **Import** button (or use **File > Import**)
4. Select the `.moom` file that was generated
5. Your new configurations will appear in Moom's custom controls list

#### Step 3 — Test your shortcuts

Try each keyboard shortcut to confirm the window positions match what you expected. If anything needs adjusting, ask Claude to modify the file and re-import.

---

### Workflow 2: Export, Edit, and Re-Import

Use this when you want to modify your existing Moom settings — for example, to add new shortcuts to your current setup, change keyboard bindings, or reorganise your custom controls.

#### Step 1 — Export your current settings from Moom

1. Open **Moom** preferences and go to the **Custom** tab
2. Select all the custom controls you want to export (or just the ones you want to edit)
3. Click **Export** (or use **File > Export**)
4. Save the `.moom` file somewhere accessible (e.g., your Desktop or project folder)

#### Step 2 — Ask Claude to edit the file

Point Claude at your exported file and describe the changes you want:

```
Read my Actions.moom file and add a new shortcut that puts the window
in the top-right quarter using Ctrl+Option+I
```

```
Add a chain group to Actions.moom that cycles between left half,
left third, and left two-thirds when I press Ctrl+Option+Shift+Left
```

```
What shortcuts do I have configured in Actions.moom? Show me a summary.
```

Claude will automatically create a date-stamped backup of your file before making any changes (e.g., `Actions-backup-2026-02-28-154512.moom`).

#### Step 3 — Delete existing settings in Moom before re-importing

> **Important:** Moom's import function **adds** configurations — it does not replace existing ones. If you import a modified version of a file you previously imported, you will end up with **duplicate entries** for every action that wasn't changed.

To avoid duplicates:

1. Open **Moom** preferences and go to the **Custom** tab
2. Select the custom controls that correspond to what you exported and edited
3. **Delete them** (press the Delete key or click the minus button)
4. Confirm the deletion

Only after removing the old configurations should you proceed to re-import.

#### Step 4 — Import the edited file

1. In the **Custom** tab, click **Import**
2. Select your edited `.moom` file
3. Verify that your configurations appear correctly and there are no duplicates
4. Test your shortcuts to confirm everything works

> **Tip:** If something goes wrong, you can import the backup file that Claude created in Step 2 to restore your previous settings.

---

## Examples

Here are some example prompts that work well with this skill:

| What you want | What to say |
|---------------|-------------|
| Basic halves + maximize | "Create a Moom config with left half, right half, and full screen shortcuts" |
| Coding layout | "Set up Moom with editor on the left two-thirds and terminal on the right" |
| Cycling shortcuts | "Make a chain that cycles left half, left third, left two-thirds on Ctrl+Option+Shift+Left" |
| Understand a file | "Read Actions.moom and tell me what all the shortcuts do" |
| Add to existing | "Add a center shortcut on Ctrl+Option+C to my Actions.moom" |
| Custom split | "Create a 70/30 split with the main window on the left" |
| Multi-display | "Add shortcuts to move windows between monitors" |
| Full starter kit | "What's a good set of Moom shortcuts for someone with two monitors?" |
| App-specific layout | "Create a layout that puts VS Code on the left, Ghostty bottom-right, and Safari top-right" |
| Organise with folders | "Group my shortcuts into folders: one for halves, one for thirds, one for layouts" |
| Combined action | "Make a shortcut that moves a window to the next display and maximizes it in one step" |
| Portrait display | "Set up shortcuts for my vertical monitor with a top/bottom split" |

## Supported Action Types

| Code | Type | Description |
|------|------|-------------|
| 19 | Move & Resize | Position a window to a proportional screen area |
| 1001 | Snapshot/Layout | Save and restore complete window arrangements |
| 10001 | Group / Folder / Chain | Container: folder (organizer), combined action, or cycling chain |
| 21 | Center | Center a window on screen |
| 31 | Move by Delta | Nudge a window by a pixel amount |
| 33 | Move to Edge/Corner | Snap a window to a screen edge |
| 41 | Move to Other Display | Send a window to another monitor |
| 51 | Resize | Resize a window to specific dimensions |
| 54 | Revert | Undo the last Moom action |
| 61 | Separator | Visual divider in the Moom menu |
| 0 | Spacer | Empty space in the Moom menu |
| -101 | Section Header | Text label in the Moom menu |

## Recently Added Features

### Folders and Nested Organisation

Groups (Action 10001) can act as **Folders** in Moom's UI when Chain Mode is 0. Folders organise your custom actions into collapsible sections, and can be nested arbitrarily deep (folders within folders). Assigning a hot key to a folder opens it as a sub-menu of its children.

### Combined-Action Chains

In addition to cycling chains (press repeatedly to advance through positions), the skill now supports **combined-action chains** — where all children execute at once in a single key press. This lets you compose multi-step operations, like moving a window to another display *and* resizing it, into one shortcut.

### Portrait Display Grid Orientation

The Configuration Grid can be set independently per action, so you can use wider grids (e.g., 12x6) for landscape displays and taller grids (e.g., 6x12) for portrait displays within the same .moom file. This is useful for setups with mixed display orientations.

## Skill Structure

```
moom-settings/
├── SKILL.md              # Main skill instructions and reference tables
└── references/
    └── action-types.md   # Detailed action type specs, key codes, modifier flags
```

## Limitations

- This skill works with .moom files (Moom's import/export format). It cannot directly modify Moom's live preferences or interact with the running Moom application.
- Snapshot/layout actions require specific screen dimensions and app bundle identifiers, which may not transfer perfectly between different display setups. The skill includes common Mac display dimensions, but your exact Available Screen Frame depends on dock position and size.
- The file format was reverse-engineered from exported .moom files. While tested and validated, undocumented edge cases may exist.
- Some modifier flag combinations are inferred from confirmed values using bit-pattern logic. If a shortcut doesn't register in Moom, try a different modifier combination.
