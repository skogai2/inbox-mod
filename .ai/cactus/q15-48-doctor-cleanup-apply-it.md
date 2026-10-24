# q15 — Doctor cleanup: apply it?

status: answered
act: ask
kind: choice
agent: b4111185-69a0-45a1-b4b3-2c7f882709d6
cwd: .
asked at: 2026-10-24T14:19:05.536174+00:00

## Context

Full report is in chat. Groups: (A) disable 7 unused plugins: agentic-harness, ai-instruct, pr-review-toolkit, agent-context-kit, agent-discussion (~4k est. tokens/session of listing), plus context7 (MCP, 0 calls ever) and typescript-lsp (0 uses, no TS here); (B) skillOverrides off for unused user skills modern-web-guidance, fnox, plus the user-scope worktrunk + wt-switch-create copies that duplicate the worktrunk plugin's; (C) move 238 lines of field/sky/garden invariants from CLAUDE.md into .claude/rules/field.md scoped to field/sky/garden/fieldproc + their tests (~4.2k est. tokens/session, CLAUDE.md 72k -> 55k chars); (D) mise uninstall claude@2.1.309 (stale copy). All reversible; C is a working-tree edit you review in git diff. Default if skipped: nothing changes.

## Files

- /home/skogix/.local/src/inbox-mod/CLAUDE.md
- /home/skogix/.claude/settings.json

## Options

- all — apply A-D  (★●)
- pick — tell me which letters
- none — keep everything

## Recommendation

all — high

## Answer

all
answered at: 2026-10-24T17:46:45.699857+00:00
