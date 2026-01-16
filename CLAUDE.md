# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code skill that enables AI-assisted generation of macOS/iOS Shortcuts. It provides comprehensive documentation on the Shortcuts plist format, enabling Claude to generate valid `.shortcut` files from natural language descriptions.

**This is a documentation-only project** - no build, test, or lint commands. The skill consists of markdown reference files that Claude loads when helping users create shortcuts.

## Directory Structure

All skill documentation lives in `src/`:

**Core reference:**
- `SKILL.md` - Skill definition with YAML frontmatter (required for Claude Code recognition)
- `ACTIONS.md` - 427 WF*Action identifiers and parameters
- `APPINTENTS.md` - 728 AppIntent actions (iOS 16+)
- `PARAMETER_TYPES.md` - Parameter serialization formats (WFTextTokenString, etc.)
- `VARIABLES.md` - UUID-based variable reference system
- `CONTROL_FLOW.md` - Repeat, Conditional, Menu patterns
- `FILTERS.md` - Content filters for Find/Filter actions
- `PLIST_FORMAT.md` - Complete plist structure
- `EXAMPLES.md` - Complete working examples

**Quality assurance:**
- `VALIDATION.md` - Pre-output checklist (MUST complete before outputting code)
- `TESTING.md` - Testing procedures for generated shortcuts
- `TROUBLESHOOTING.md` - Common problems and solutions

## Key Technical Concepts

When working with this skill:

1. **Shortcuts are binary plists** written as XML with `WFWorkflowActions` array containing action dictionaries
2. **Variable references** use UUIDs and the placeholder character `￼` (U+FFFC) in `attachmentsByRange`
3. **Control flow** uses `GroupingIdentifier` (UUID) and `WFControlFlowMode` integer (0=start, 1=middle, 2=end)
4. **UUIDs must be uppercase** format: `XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX`
5. **Signing is required** - generated shortcuts must be signed with `shortcuts sign` CLI before import

## Installation Location

When deploying this skill, files go to `~/.claude/skills/shortcuts-generator/` (or similar named directory).
