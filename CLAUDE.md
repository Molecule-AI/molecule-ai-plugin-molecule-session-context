# molecule-ai-plugin-molecule-session-context

Session-start context loader for the Molecule AI platform. Automatically injects
relevant recent context — cron learnings, repo activity — into the agent's system
prompt when a new session begins.

## Purpose

`molecule-session-context` eliminates the need for the agent to manually query
external systems at session start. It reads recent cron-learning entries and repo
PR/issue counts and formats them into a context block appended to the agent's
initial prompt.

## Key Conventions

| Topic | Convention |
|---|---|
| **Hook** | `session-start-context` — fires once when a workspace session initialises |
| **Time window** | Default: last 24 hours of cron learnings; configurable via `session_context.window_hours` |
| **Cron learnings** | Stored in molecule-core memory store under the `cron-learnings` namespace |
| **Repo stats** | PR and issue counts fetched from GitHub API via `GITHUB_TOKEN` |
| **Output format** | Markdown context block prepended to the agent's initial prompt |
| **Runtime** | `claude_code` only |

## Project Structure

```
molecule-ai-plugin-molecule-session-context/
├── skills/
│   └── session-context/    # Skill manifest
│       └── skill.yaml
├── adapters/               # Runtime-specific context injection
├── rules/
│   └── session-context.md
├── runbooks/
│   └── local-dev-setup.md
└── plugin.yaml
```

## Dev Setup

```bash
git clone https://github.com/Molecule-AI/molecule-ai-plugin-molecule-session-context
cd molecule-ai-plugin-molecule-session-context

# Validate plugin.yaml
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"
```

### Required Environment Variables

| Variable | Required | Description |
|---|---|---|
| `GITHUB_TOKEN` | Yes | GitHub personal access token for PR/issue count queries |
| `SESSION_CONTEXT_WINDOW_HOURS` | No | Override default 24 h window (default: `24`) |

### Configuration

In `config.yaml`:

```yaml
session_context:
  window_hours: 24          # how far back to read cron learnings
  github_repos:              # repos to query for PR/issue stats
    - owner/repo-name
  max_entries: 10            # max cron learning entries to include
  format: markdown            # output format (markdown | plain)
```

## Testing

```bash
# Mock GITHUB_TOKEN for unit tests
GITHUB_TOKEN=test_token pytest tests/ -v
```

## Release Process

1. Bump `plugin.yaml` version
2. Tag: `git tag vX.Y.Z && git push --tags`

## Rules

See `rules/session-context.md`.

## Known Gotchas

- If `GITHUB_TOKEN` is absent, repo stats are silently omitted from the context
  block. The session still starts normally.
- Cron learnings must exist in the molecule-core memory store before session start;
  if no learnings are recorded, the context block contains only repo stats.
- The context block is prepended, not merged — it does not replace any
  user-provided initial prompt.
- Rate limiting from the GitHub API is not retried; a 403 from GitHub causes the
  stats section to be silently dropped.
