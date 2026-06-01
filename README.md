# Self-Improving Agent Skill

A small Codex/Claude Code skill for turning agent mistakes into durable instruction fixes.

Use it when an agent followed the wrong tool, stale workflow, misleading agent name, or contradictory documentation. The skill guides the agent to find the active instruction sources, patch the smallest durable route, verify stale references are gone, and commit only the scoped fix.

The skill is the root [`SKILL.md`](SKILL.md). Copy this repo into your skills directory or install it with any Codex skill installer that accepts a GitHub repo path.

Example:

```text
Use self-improving-agent. The QA worker used Playwright even though our repo says browser-harness. Find why, fix the durable context, rename anything misleading, and verify no active routing source still points to Playwright.
```
