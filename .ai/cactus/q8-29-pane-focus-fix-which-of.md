# q8 — Answer from the prompt: which forms?

status: open
act: ask
kind: multi
thread: pane-vim
agent: 32a13300-fb85-4fa4-a14f-edfb06bc2dc7
cwd: .
asked at: 2026-10-24T11:51:46.699762+00:00

## Context

No problem with either. A prompt.submit hook can drop a composer line before it reaches me (it never becomes a turn), and a slash command gets its args. Both feed one parser acting on the pane's selected row:

  1..9 pick, y/n confirm, s skip, c clear/close, j/k move
  e TEXT elaborate, b break up, u undo, g send multi
  any other text: free-text answer/verdict
  q12 2: aim at a row by key instead of the selection

Feedback comes as a toast ("q8: answered 1"); the pane keeps showing the row. Your bare "1" just now reached me as chat; that is the one the bare form would catch.

Separately: ctrl+x tab landing in the Input (where the hotkeys are dead) is a bug I fix either way.

## Files

- /home/skogix/.local/src/inbox-mod/mods/cactus-pane/hooks/register.tsx

## Options

- colon — a composer line starting with : is a cactus command (:1, :s, :e why)
+ fastest; one key of prefix
- a real message starting with : gets eaten (:: escapes)
- slash — a /c slash command (/c 1, /c e why)
+ autocompletes, shows in /help, can never eat a message
- bare — a lone digit or y/n as the whole line answers while a row is open
+ zero prefix
- "1" meant for me (e.g. picking from my chat list) answers the row instead

## Rewrite

was: Answer from the prompt: which forms?

No problem with either. A prompt.submit hook can drop a composer line before it reaches me (it never becomes a turn), and a slash command gets its args. Both feed one parser acting on the pane's selected row:

  1..9 pick, y/n confirm, s skip, c clear/close, j/k move
  e TEXT elaborate, b break up, u undo, g send multi
  any other text: free-text answer/verdict
  q12 2: aim at a row by key instead of the selection

Feedback comes as a toast ("q8: answered 1"); the pane keeps showing the row. Your bare "1" just now reached me as chat; that is the one the bare form would catch.

Separately: ctrl+x tab landing in the Input (where the hotkeys are dead) is a bug I fix either way.

options were:
-  — prefix: a composer line starting with : is a cactus command (:1, :s, :e why)
+ fastest; one key of prefix
- a real message starting with : gets eaten (:: escapes)
- /c — a /c slash command (/c 1, /c e why)
+ autocompletes, shows in /help, can never eat a message
- bare — a lone digit or y/n as the whole line answers while a row is open
+ zero prefix
- "1" meant for me (e.g. picking from my chat list) answers the row instead

files were:
- /home/skogix/.local/src/inbox-mod/mods/cactus-pane/hooks/register.tsx
