---
name: self-improving-agent
description: Use when a user asks to improve agent behavior, fix stale or conflicting instructions, explain why an agent chose the wrong tool or workflow, rename/update agent or skill routing, or turn a correction into durable guidance across manifests, skills, agents, commands, memory, and docs.
---

# Self-Improving Agent

## Goal

Turn a real agent failure into a durable behavior fix. Do not just explain what went wrong. Find the instruction, tool, skill, agent, command, memory, or documentation that caused the behavior, patch the smallest current sources, and verify the bad route is gone.

## Workflow

1. State the observed failure as a concrete behavior.
   - Example: "The agent used Playwright MCP even though the repo says to use browser-harness."
   - Separate the symptom from the root cause. The symptom is what happened; the root cause is usually stale or more-specific instructions.

2. Search for the instruction sources.
   - Use fast text search first: `rg -n "old-tool|old-agent|wrong phrase|new-tool|correct phrase" .`
   - Check root manifests, skill files, agent definitions, command files, docs, memory, and generated routing references.
   - If the repo has multiple agent homes, inspect all active homes. Common examples: `.agents/`, `.claude/`, `.cursor/`, `.codex/`.

3. Determine precedence.
   - More specific worker or skill instructions usually beat broad root guidance.
   - A stale agent name, command template, or subagent type often beats a correct general policy.
   - If the same behavior is described in both current docs and old memory, current runtime files win, but stale memory still needs a correction if agents read it.

4. Patch the durable route.
   - Rename misleading agents, skills, or commands when the name itself causes drift.
   - Update every active call site that spawns, invokes, or links the renamed thing.
   - Mirror shared guidance across equivalent runtimes when the project uses more than one.
   - Keep unrelated edits out of scope. In a dirty worktree, stage explicit paths only.

5. Preserve useful constraints.
   - Keep the real quality bar from the old instruction if it was useful.
   - Replace only the stale route, tool, or name.
   - If a required proof changed from "video" to "screenshots", say what proof is now mandatory and what is optional.

6. Verify with negative and positive checks.
   - Negative search: `rg -n "old-agent|old-tool|stale route" <active paths>` should return only intentional historical notes or no hits.
   - Positive search: `rg -n "new-agent|new-tool|correct route" <active paths>` should show the active manifests, skills, agents, and commands.
   - Parse machine-readable files when possible, such as TOML/YAML/JSON.
   - Confirm renamed files exist and old files do not.

7. Document the cause and the fix.
   - Explain why the agent followed the wrong instruction.
   - Name the highest-impact files changed.
   - Mention verification performed.
   - If pushing, commit only the scoped fix.

## Guardrails

- Do not overwrite unrelated user changes.
- Do not use broad cleanup commands such as `git reset --hard` or checkout resets.
- Do not silently delete historical memory unless it is an active routing source or the user asked to clean it.
- Do not leave "legacy-named" labels in place after renaming; they become the next source of confusion.
- Do not rely on root instructions alone if a more-specific worker, skill, or command contradicts them.
- Do not call the work done until the stale trigger no longer appears in active routing files.

## Useful Commands

```bash
rg -n "old-name|old-tool|new-name|new-tool" .agents .claude docs memory AGENTS.md
git diff --name-status
git diff --check
python3 - <<'PY'
import tomllib
from pathlib import Path
with open(".agents/agents/example.toml", "rb") as f:
    data = tomllib.load(f)
assert data["name"] == "expected-name"
assert not Path(".agents/agents/old-name.toml").exists()
PY
git add -- path1 path2 path3
git commit -m "Route agents through correct tool"
```

## Final Report

Keep the final report short:

- Root cause: the stale or more-specific instruction that caused the failure.
- Changes: renamed or updated routing sources.
- Verification: searches, parsers, and push/commit status.
- Remaining risk: any historical notes intentionally left alone.
