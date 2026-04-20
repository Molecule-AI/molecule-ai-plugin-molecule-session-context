# molecule-session-context — HITL Gate Rules

This is a placeholder for session-context-specific rules. The session-context plugin
does not implement HITL gates directly — it uses the `session-start-context` hook.

## If This File Grows

If future releases of `molecule-session-context` add HITL-like approval flows
(e.g., requiring human sign-off before loading external repo stats), add those
rules here.

## Current Scope

This plugin uses `session-start-context` only. No gate logic is implemented.
