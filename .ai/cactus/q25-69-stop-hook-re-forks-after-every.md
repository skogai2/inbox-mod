# q25 — Stop hook re-forks after every 'done'/'idle' answer. Change it?

status: answered
act: ask
kind: choice
agent: b4111185-69a0-45a1-b4b3-2c7f882709d6
cwd: .
asked at: 2026-10-24T17:58:39.582189+00:00

## Context

Each mod wake reads as a human prompt, so stop_fork.py demands a new fork even when the answer was done/idle; this loop has posted q19-q24. Leave this row open and the hook stays quiet (it mutes while I have an open row).

## Files

- /home/skogix/.local/src/inbox-mod/hooks/stop_fork.py

## Options

- fix — don't hold a turn opened by a cactus-pane wake whose answers were done/idle-shaped  (★◐)
- leave — keep the hook as is; I'll leave this row open

## Recommendation

fix — med

## Answer

fix
answered at: 2026-10-24T17:59:00.867200+00:00
