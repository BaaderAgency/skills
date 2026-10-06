# HookHoard

Build, fill and break down mood boards and swipe files from your AI chat. This is the companion skill for the
free HookHoard Chrome extension: it teaches your AI app to create boards, add images from links, files or web
pages, label each item with why it works, and turn a board into a brief or shot list.

Needs the HookHoard Chrome extension with "Connect your AI" set up. Your boards stay in your own browser.

## Install

- **Claude Code:**
  ```
  cp -R hookhoard ~/.claude/skills/
  ```
- **Claude Desktop and claude.ai:** zip the `hookhoard` folder and upload it under Settings > Skills
  (or use the "Get the HookHoard skill" download in the extension).
- **Codex, Cursor, Antigravity and other agents:**
  ```
  npx skills add BaaderAgency/skills --skill hookhoard
  ```

Then ask something like: "make me a mood board for a summer skincare launch from these links".

## What's inside

- `SKILL.md`: when to use HookHoard, the board-from-a-brief flow, a practical ad breakdown method, how to write
  a specific "why it works" line, and hard rules (no invented links or facts, no claiming saves that failed).
