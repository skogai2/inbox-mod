# q4 — MCP failure diagnosed: what fix?

status: answered
act: ask
kind: choice
agent: 32a13300-fb85-4fa4-a14f-edfb06bc2dc7
cwd: .
asked at: 2026-10-24T11:45:39.769905+00:00

## Context

The plugin server (plugin:cactus:cactus) is connected and fine. The failing 'cactus' entry is the repo-root .mcp.json loaded a second time as this project's own config, where CLAUDE_PLUGIN_ROOT is unset, so the path doesn't resolve (claude mcp list: 'Missing environment variables: CLAUDE_PLUGIN_ROOT'). It only happens in sessions opened inside this repo, and you've now disabled it for this project via /mcp. Default if no answer: leave it.

## Files

- /home/skogix/.local/src/inbox-mod/.mcp.json
- /home/skogix/.local/src/inbox-mod/.claude-plugin/plugin.json
- /home/skogix/.local/src/inbox-mod/scripts/package-plugin.sh

## Options

- leave — keep it disabled per project; no code change  (★◐)
- move — move it to server/mcp.json, point plugin.json mcpServers at it, update package-plugin.sh and docs. Risk: the Desktop .plugin bundle may only read the root .mcp.json

## Recommendation

leave — med

## Answer

leave
answered at: 2026-10-24T11:47:03.931003+00:00
