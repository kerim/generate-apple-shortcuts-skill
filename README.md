# Shortcuts Generator Skill for Claude Code

> **Warning: Work in Progress**
>
> This skill does not currently produce valid code reliably. Generated shortcuts may fail to import or show "Unknown Action" errors. Use with caution and expect to debug output manually.

A Claude Code skill that enables AI-assisted generation of macOS/iOS Shortcuts. Create valid `.shortcut` plist files that can be signed and imported into Apple's Shortcuts app.

## Features

- Generate shortcuts from natural language descriptions
- Comprehensive coverage of 427 WF*Actions and 728 AppIntents
- Automatic signing with macOS `shortcuts` CLI
- Built-in validation and troubleshooting guidance

## Installation

### For Claude Code

1. Clone or download this repository
2. Copy the `skill/` folder to your Claude Code skills directory:
   ```bash
   cp -r skill ~/.claude/skills/generate-shortcuts-skill
   ```
3. Restart Claude Code to load the skill

### Alternative: Direct Download

1. Download just the `skill/` folder from this repository
2. Rename it to `generate-shortcuts-skill` and place it at `~/.claude/skills/generate-shortcuts-skill/`
3. Restart Claude Code

## Usage

Once installed, ask Claude Code to create a shortcut:

- "Create a shortcut that shows the current weather"
- "Build a shortcut that asks for text input and shows it"
- "Make a shortcut that opens Safari and navigates to a URL"

Claude will generate the plist XML, write it to a `.shortcut` file, and sign it so you can import it directly into the Shortcuts app.

## Documentation

| File | Description |
|------|-------------|
| `SKILL.md` | Skill definition with quick start guide |
| `ACTIONS.md` | All 427 WF*Action identifiers and parameters |
| `APPINTENTS.md` | All 728 AppIntent actions |
| `PARAMETER_TYPES.md` | Parameter value types and serialization formats |
| `PLIST_FORMAT.md` | Complete plist structure reference |
| `VARIABLES.md` | Variable reference system |
| `CONTROL_FLOW.md` | Repeat, Conditional, Menu patterns |
| `FILTERS.md` | Content filters for Find/Filter actions |
| `EXAMPLES.md` | Complete working examples |
| `VALIDATION.md` | Pre-output validation checklist |
| `TESTING.md` | Testing procedures for generated shortcuts |
| `TROUBLESHOOTING.md` | Common problems and solutions |

## Requirements

- macOS with the `shortcuts` CLI tool (included with macOS)
- Claude Code CLI

## License

MIT
