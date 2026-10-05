# Lens: Heuristic Sweep

Walk the scope against Nielsen's usability heuristics (NN/g). Cite each
finding by number and name, e.g. "Nielsen #9, error recovery."

Trimmed list: #4 (consistency) is the Design-system drift lens, #8
(minimalism) is the Task walkthrough's distraction check, and #10
(help/docs) is dropped.

## #1 Visibility of system status
- Every action gets a visible reaction: flash, Turbo Stream update, or
  button pending state. Silence reads as broken.
- Async content (lazy Turbo Frames, Stimulus fetches) shows a loading
  state without layout shift.
- Current location is shown: active nav item, breadcrumb, or step
  indicator.

## #2 Match between system and the real world
- The user's words, not model, column, or internal names
  ("Team member," not "Membership").
- Information appears in the order the user thinks about it.

## #3 User control and freedom
- Every modal can be closed; every multi-step flow can be cancelled or
  stepped back.
- Destructive actions offer undo where practical, otherwise a
  confirmation.
- Browser Back works as expected (Turbo history isn't broken).

## #5 Error prevention
- Inputs constrained where possible: select, radio, date picker over
  free text; invalid options disabled.
- Sensible defaults pre-filled.
- Confirmation before destructive or irreversible actions.
- Forgiving input (trims spaces, accepts common formats).

## #6 Recognition rather than recall
- Current values, selections, and active filters stay visible.
- The user never has to remember an ID or a choice from a previous
  screen.
- Fields have visible labels, not placeholder-only.

## #7 Flexibility and efficiency of use
- Repeated actions have a bulk option.
- Frequent actions are reachable in few steps.
- Lists that grow have search, filter, or sort.

## #9 Help users recognize, diagnose, and recover from errors
- Errors appear next to the field they're about, in plain language.
- They say what's wrong and how to fix it — no raw codes, no generic
  "Something went wrong," no bare "Invalid."
- Input is preserved on error.
- Error states offer a next step (retry, go back, contact).
