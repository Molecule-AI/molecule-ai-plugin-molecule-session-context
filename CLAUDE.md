# molecule-session-context — Molecule AI Plugin

Plugin that auto-loads recent cron-learnings and repo PR/issue counts at session start. Pairs with `molecule-cron-learnings`.

## Overview

Session-context plugin injects recent cron-learnings and repository activity summaries into the agent's context at session start. Designed for `claude_code` runtime.

## Build and Test

```bash
# Validate plugin structure
python3 .molecule-ci/scripts/validate-plugin.py

# Install dependencies
pip install -r .molecule-ci/scripts/requirements.txt
```

## Project Structure

```
.                       # Plugin root
plugin.yaml             # Plugin manifest
SKILL.md                # agentskills.io spec
.claude/                 # Agent settings
.molecule-ci/
  scripts/
    validate-plugin.py  # plugin.yaml + content validator
    requirements.txt    # Python deps
runbooks/
  local-dev-setup.md    # Local development guide
```

## Plugin Manifest

```yaml
name: molecule-session-context
version: 1.0.0
description: Auto-load recent cron-learnings + repo PR/issue counts at SessionStart
author: Molecule AI
runtimes:
  - claude_code
hooks:
  - session-start-context
```

## Pre-commit Checklist

```bash
# 1. Validate plugin.yaml structure
python3 .molecule-ci/scripts/validate-plugin.py

# 2. Check for credentials (Python-based — no literal secret patterns)
python3 -c "
import re, sys
with open('plugin.yaml') as f:
    content = f.read()
patterns = [r'sk.ant', r'ghp.', r'AKIA[A-Z0-9]']
if any(re.search(p, content) for p in patterns):
    print('FAIL: possible credentials found')
    sys.exit(1)
print('No credentials: OK')
"

# 3. Verify plugin.yaml is valid YAML
python3 -c "import yaml; yaml.safe_load(open('plugin.yaml'))"

echo "All checks passed"
```

## Release

Version bumps in `plugin.yaml` → tag and push → platform registry publishes.