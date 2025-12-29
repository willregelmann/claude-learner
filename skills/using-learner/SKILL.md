---
name: using-learner
description: This skill should be used when the user asks to "learn a topic", "generate skills for X", "research and create skills", "use /learn command", "how do I learn about X", or mentions generating Claude skills from research. Provides guidance on using the claude-learner plugin's /learn command.
version: 1.0.0
---

# Using Claude Learner

## Overview

The `/learn` command researches any topic and generates Claude-optimized skills that provide behavioral guidance for working in that domain.

## How to Use

Invoke the command with a topic to research:

```bash
/learn <topic> [--project]
```

### Examples

- `/learn laravel 12` - Generate skills for Laravel 12 framework
- `/learn rocket physics` - Generate skills for rocket physics concepts
- `/learn kubernetes networking` - Generate skills for K8s networking
- `/learn rust axum --project` - Generate skills in current project only

### Flags

**--project**: Generate skills to `./.claude/skills/` instead of `~/.claude/skills/`
- Use for project-specific knowledge that shouldn't be shared globally
- Skills in `./.claude/skills/` are discovered when working in that project
- Skills in `~/.claude/skills/` are available across all projects

## What Gets Generated

### Output Location

Skills are created at:
- **Default**: `~/.claude/skills/` (user-level, available everywhere)
- **With --project**: `./.claude/skills/` (project-level, scoped to current project)

Skills are named with topic prefix: `{{topic-slug}}-{{subtopic-slug}}/SKILL.md`

```
~/.claude/skills/           # default location
├── laravel-12-routing/SKILL.md
├── laravel-12-eloquent/SKILL.md
├── laravel-12-blade-templates/SKILL.md
└── ...

./.claude/skills/           # with --project flag
├── rust-axum-routing/SKILL.md
├── rust-axum-handlers/SKILL.md
└── ...
```

### Skill Content

Each generated skill follows Claude Code skill development best practices:
- **Description**: Third-person with specific trigger phrases
- **When This Skill Applies**: Concrete triggering conditions
- **Key Patterns**: Behavioral guidance using imperative form
- **Common Mistakes to Avoid**: Anti-patterns with corrections
- **Examples**: Concrete demonstrations
- **Sources**: Attribution to research sources

Skills are designed for AI consumption using imperative/infinitive form, not second person.

### Auto-Discovery

Both `~/.claude/skills/` and `./.claude/skills/` are auto-discovered by Claude Code. No plugin installation or configuration required.

## Update Mode

Re-running `/learn <topic>` on an existing topic enters **UPDATE MODE**:

**What happens:**
1. Reads existing skills to understand current coverage
2. Researches topic for new information
3. Updates existing skills if significant new findings discovered
4. Adds new skills for newly discovered subtopics
5. Preserves custom user edits (content not matching original patterns)
6. Marks obsolete subtopics with `obsolete: true` in frontmatter

**When to update:**
- Topic has evolved (new version, new features)
- Initial generation missed important subtopics
- Want to refresh with latest best practices

**Preservation:**
Custom edits are detected and preserved during updates. The command identifies original generated patterns and keeps user modifications.

## Best Practices for Effective Learning

### Be Specific with Topics

**Good:**
- `/learn laravel 12` (includes version)
- `/learn kubernetes networking` (specific subdomain)
- `/learn rust async` (focused area)

**Too vague:**
- `/learn laravel` (which version?)
- `/learn kubernetes` (too broad, generates surface-level skills)
- `/learn programming` (way too broad)

### Topic Selection Guidelines

- **Include version numbers** for frameworks and libraries
- **Specify subdomain** for broad topics (e.g., "kubernetes networking" not just "kubernetes")
- **Works for any domain** - programming, science, business processes, methodologies
- **Public information required** - needs good web search results

### After Generation

1. **Restart Claude Code** to load new skills
2. **Review generated skills** in `~/.claude/skills/` or `./.claude/skills/`
3. **Test triggering** by working on related tasks
4. **Customize if needed** - add project-specific details, fix inaccuracies
5. **Update later** if topic evolves

## Troubleshooting

### Skills Not Appearing

**Check generation:**
```bash
ls ~/.claude/skills/ | grep "topic-slug"
```

**If skills exist but not loading:**
- Restart Claude Code (skills load at startup, not hot-reloaded)
- Check SKILL.md has valid YAML frontmatter
- Verify directory structure: `skill-name/SKILL.md`

### Regenerate from Scratch

To delete and regenerate:
```bash
rm -rf ~/.claude/skills/<topic-slug>-*
/learn <topic>
```

Update mode preserves edits. To force fresh generation, delete first.

### Poor Quality Skills

If generated skills are too generic or miss key concepts:
- Use more specific topic name
- Check if topic has good public documentation
- Run `/learn` again in update mode after topic information improves
- Manually enhance skills with project-specific knowledge

### No Web Results

If topic is proprietary or niche with limited public information:
- Provide documentation to Claude for manual skill creation
- Use `/learn` on related public topics as starting point
- Create skills manually following skill development best practices
