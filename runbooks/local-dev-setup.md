# molecule-session-context — Local Dev Setup

## Prerequisites

- `git`
- Python 3.11+
- A local or staging Molecule platform
- A workspace with the `claude_code` runtime

## Clone and Validate

```bash
git clone https://github.com/Molecule-AI/molecule-ai-plugin-molecule-session-context
cd molecule-ai-plugin-molecule-session-context

python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"
python3 -c "import yaml; yaml.safe_load(open('skills/session-context/skill.yaml'))"
```

## Required Environment Variables

```bash
export GITHUB_TOKEN=ghp_your_test_token_here
export SESSION_CONTEXT_WINDOW_HOURS=1   # short window for faster testing
```

## Point Your Org Template at a Dev Branch

In `org.yaml`:

```yaml
plugins:
  - github://Molecule-AI/molecule-ai-plugin-molecule-session-context@<branch>
```

## Verify Context Loads

Start a new workspace session. Check the agent's system prompt (via transcript
or platform UI) for a block near the top that looks like:

```markdown
## Session Context (last 1 hour)

### Cron Learnings
- [cron-learner] ...learning text...
...

### Repo Activity
- owner/repo: 3 open PRs, 12 issues
...
```

If no context block appears, check the runtime logs for
`session_context_loaded` or `session_context_skipped` events.

## Seed Test Cron Learnings

```bash
# Write a test cron learning to the molecule memory store
# (adapt to your platform's memory API)
curl -X POST "$MOL_PLATFORM_URL/memory/cron-learnings" \
  -H "Authorization: Bearer $MOL_API_KEY" \
  -d '{"content": "[cron-learner] Test learning: always validate input", "timestamp": "2026-04-20T12:00:00Z"}'
```

## Common Issues

| Symptom | Cause | Fix |
|---|---|---|
| No context block in prompt | `GITHUB_TOKEN` not set, no learnings in store | Set env var; seed a cron learning |
| GitHub stats all zeros | Token lacks `repo` scope or org access | Use a token with `read:user,repo` |
| Context block huge | Too many learnings/repos in window | Reduce `max_entries` or `window_hours` in config |
| 403 from GitHub | Token expired or rate-limited | Refresh token; check `GITHUB_TOKEN` in env |
