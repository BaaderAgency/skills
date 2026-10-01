# Effort Loop

Build on low, verify on high. A Claude Code skill that splits a build into a cheap builder
(a subagent on low effort) and a careful, separate verifier (a fresh subagent on high effort)
that tests the work against cases the builder never saw.

Write-up: https://baaderagency.com/blog/effort-loop-build-on-low-verify-on-high

## Install

1. Put the folder where Claude Code finds skills:
   ```
   cp -R effort-loop ~/.claude/skills/
   ```
2. In each project you use it in, copy the two helper agents so Claude Code loads them:
   ```
   mkdir -p .claude/agents && cp ~/.claude/skills/effort-loop/agents/*.md .claude/agents/
   ```
3. Start a build with `/effort-loop` or say "build on low, verify on high".

## What's inside

- `SKILL.md`: the five steps and the five hard rules.
- `agents/effort-builder.md`: the builder, `effort: low`.
- `agents/effort-verifier.md`: the verifier, `effort: high`.

Needs a Claude Code version that supports `effort` in subagent frontmatter and Opus access.
The testing method is adapted from Anthropic's "Automating eval design and hillclimbing with Claude".
