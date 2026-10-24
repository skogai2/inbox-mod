# q1 — Session started with no task. What next?

status: answered
act: ask
kind: choice
thread: session
agent: afc11eac-5d6c-4505-ad17-35b1229da92f
cwd: .
asked at: 2026-10-24T03:05:13.415470+00:00

## Context

At session start the cactus MCP server failed with ENOENT on
${CLAUDE_PLUGIN_ROOT}/server/cactus-mcp (.mcp.json). The CLI works.

If nobody answers within the hour, I stay idle.

## Files

- /home/skogix/.local/src/inbox-mod/.mcp.json

## Options

- mcp — find why the cactus MCP server fails to launch
+ server/cactus-mcp exists in the checkout
- the cause is probably ${CLAUDE_PLUGIN_ROOT} resolving to a plugin copy that has no server/
- idle — wait for a task you type in chat  (★◐)

## Recommendation

idle — med
no task given; MCP is optional for Claude Code

## Answer

mcp
answered at: 2026-10-24T10:44:09.417286+00:00
