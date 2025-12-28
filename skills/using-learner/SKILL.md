---
name: using-learner
description: Use when someone asks about the /learn command, how to generate skills, or how to use the claude-learner plugin
---

# Using Claude Learner

## Overview

The `/learn` command researches any topic and generates Claude-optimized skills that help you work more effectively in that domain.

## Usage

```
/learn <topic>
```

**Examples:**
- `/learn laravel 12` - Generate skills for Laravel 12 framework
- `/learn rocket physics` - Generate skills for rocket physics concepts
- `/learn kubernetes networking` - Generate skills for K8s networking

## What Gets Generated

For each topic, the command creates a mini-plugin at `~/.claude/plugins/learned/<topic>/` containing:

- `plugin.json` - Metadata about the generated skills
- `skills/` - Directory of 4-8 focused skills

Each skill contains:
- When to apply the skill (triggering conditions)
- Key patterns and best practices
- Common mistakes to avoid
- Concrete examples

## Re-running for Updates

Running `/learn <topic>` again on an existing topic will:
- Research for new information
- Update existing skills with new findings
- Add skills for newly discovered subtopics
- Preserve any custom edits you've made

## Tips for Effective Learning

1. **Be specific**: `/learn laravel 12` is better than `/learn laravel`
2. **Domain agnostic**: Works for any topic, not just programming
3. **Wait for completion**: Research takes time, let it finish
4. **Check generated skills**: Review and customize as needed

## Troubleshooting

**Skills not appearing?**
- Check `~/.claude/plugins/learned/` for the generated directory
- Restart Claude Code to pick up new plugins

**Want to regenerate from scratch?**
- Delete `~/.claude/plugins/learned/<topic>/`
- Run `/learn <topic>` again
