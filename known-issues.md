# Known Issues

> Living document for molecule-ai-plugin-molecule-session-context.

---

## 1. GitHub API rate limit causes silent stats drop

**Severity:** Medium
**Affected versions:** All stable
**Status:** Known; tracked in `#sess-3`

The context loader makes unauthenticated or lightly-authenticated GitHub API calls
to count PRs and issues. On a heavily-loaded org, this can 403 immediately. The
error is caught and silently swallowed; the session starts with no repo stats.

**Workaround:** Set `GITHUB_TOKEN` to a token with sufficient rate limit headroom.
For high-volume orgs, set `github_repos` to only the repos that matter.

---

## 2. Cron learnings window may include stale entries

**Severity:** Low
**Affected versions:** All stable
**Status:** Known; tracked in `#sess-9`

Cron learnings are not deduplicated by content — if the same learning was written
multiple times within the window, all copies appear in the context block. This
can produce a verbose context with redundant items.

**Workaround:** Set `max_entries: 5` to cap the total entries shown. A
deduplication step is planned.

---

## 3. Context block is not truncated at token budget

**Severity:** Medium
**Affected versions:** All stable
**Status:** Known; tracked in `#sess-15`

The context block is injected without checking whether it would exceed the model's
context budget. On sessions with many cron learnings and many repos, the injected
block can be several thousand tokens, consuming budget intended for user input.

**Workaround:** Set `max_entries` conservatively (`≤ 5`) and limit `github_repos`
to ≤ 3 repos.

---

## 4. `GITHUB_TOKEN` absence is not logged as a warning

**Severity:** Low (observability)
**Affected versions:** All stable
**Status:** Known; tracked in `#sess-21`

When `GITHUB_TOKEN` is not set, repo stats are silently skipped. The runtime
logs no warning, making it hard to diagnose why the agent is missing repo context.

**Workaround:** Check your environment config if repo stats are absent. A startup
warning log is tracked in `#sess-21`.

---

## 5. Session context is loaded before preflight checks run

**Severity:** Low
**Affected versions:** `< 1.1.0`
**Status:** Known; tracked in `#sess-28`

The `session-start-context` hook fires before `builtin_tools/preflight.py`
runs. If preflight fails and the session is rejected, the context load still
occurred (wasted GitHub API calls and cron-learning reads).

**Workaround:** This is a launch-order concern; no workaround needed for most
deployments. A hook ordering fix is tracked in `#sess-28`.
