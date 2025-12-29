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
/learn <topic> [--global]
```

### Examples

- `/learn laravel 12` - Generate skills for Laravel 12 framework
- `/learn rocket physics` - Generate skills for rocket physics concepts
- `/learn kubernetes networking` - Generate skills for K8s networking
- `/learn rust axum --global` - Generate skills at user-level for all projects

### Flags

**--global**: Generate skills to `~/.claude/skills/` instead of `./.claude/skills/`
- Use when skills should be available across all projects
- Default (no flag) generates to `./.claude/skills/` (current project)
- Skills in `~/.claude/skills/` are available everywhere

## What Gets Generated

### Preview Before Generation

After initial research, you'll see a preview of proposed skills:

```
📋 Skill Plan for "kubernetes networking"

Proposed skills (6 total):
1. kubernetes-networking-cni-plugins: Container network interface basics
2. kubernetes-networking-services: Service discovery and DNS
3. kubernetes-networking-ingress: Ingress controllers and routing
...

Proceed with generation? (yes/skip 2,4/only 1,3/no)
```

This lets you review scope, exclude subtopics you don't need, or abort if research quality was poor.

### Output Location

Skills are created at:
- **Default**: `./.claude/skills/` (project-level, scoped to current project)
- **With --global**: `~/.claude/skills/` (user-level, available everywhere)

Skills are named with topic prefix: `{{topic-slug}}-{{subtopic-slug}}/SKILL.md`

```
./.claude/skills/           # default location (project-level)
├── laravel-12-routing/SKILL.md
├── laravel-12-eloquent/SKILL.md
├── laravel-12-blade-templates/SKILL.md
└── ...

~/.claude/skills/           # with --global flag (user-level)
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

## Regeneration

Re-running `/learn <topic>` replaces existing skills with fresh generation:

**What happens:**
1. Detects existing skills for the topic
2. Warns user that existing skills will be replaced
3. User can rename old skills to preserve them (add "-old" suffix)
4. Generates fresh skills from new research
5. Replaces old skill directories

**When to regenerate:**
- Topic has evolved (new version, new features)
- Initial generation missed important subtopics
- Want to refresh with latest best practices

**To preserve old versions:**
```bash
# Before regenerating, rename to keep old skills
mv ~/.claude/skills/laravel-12-routing ~/.claude/skills/laravel-12-routing-v1
```

This is simpler than merge-based updates - no complex diff logic, no risk of corrupting custom edits.

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
2. **Review generated skills** in `./.claude/skills/` or `~/.claude/skills/`
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

Or simply re-run `/learn <topic>` - it will replace existing skills automatically.

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
