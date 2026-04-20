# molecule-session-context — Rules

## Overview

This plugin loads contextual data (cron learnings, repo stats) into the agent's
initial prompt at session start. This document covers error handling, logging,
configuration, and the release process.

---

## Error Handling

### Missing Cron Learnings

If the cron-learnings namespace is empty or unreachable, the plugin logs a
warning and continues with an empty learnings section:

```
[WARN] session_context: no cron learnings found in namespace
[INFO] session_context: continuing with repo stats only
```

### GitHub API Errors

Errors from the GitHub API are caught and silently skipped (stats omitted from
context block). This prevents session start failures from blocking the agent,
but operators should monitor logs for repeated `session_context: github error`
entries as they indicate the `GITHUB_TOKEN` may be expired or rate-limited.

### Malformed Cron Learning Entries

If a cron learning entry cannot be parsed as JSON, the entry is skipped with
a warning and the remaining entries are processed:

```
[WARN] session_context: failed to parse cron learning <entry_id>: <reason>
```

---

## Logging

| Event | Fields |
|---|---|
| `session_context_loaded` | `workspace_id`, `learnings_count`, `repos_queried`, `context_bytes` |
| `session_context_github_error` | `workspace_id`, `repo`, `status_code`, `error` |
| `session_context_skipped` | `workspace_id`, `reason` |

---

## Configuration

```yaml
session_context:
  window_hours: 24              # how far back to read cron learnings
  max_entries: 10              # max cron learning entries to include
  github_repos: []             # list of "owner/repo" strings to query
  github_timeout_seconds: 10   # per-repo GitHub API timeout
  format: markdown             # markdown | plain
```

---

## Release Process

1. Update `plugin.yaml` version
2. Update `skills/session-context/skill.yaml` if the context block format changed
3. If the cron-learning schema changed, update the parsing logic and add migration
   notes to the CHANGELOG
4. Run tests:
   ```bash
   GITHUB_TOKEN=test_token pytest tests/ -v
   ```
5. Tag: `git tag vX.Y.Z && git push --tags`
6. Update org templates that pin this plugin
7. Validate by starting a new workspace and inspecting the agent's initial
   prompt for the context block
