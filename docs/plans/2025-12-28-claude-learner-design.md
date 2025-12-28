# Claude Learner Plugin Design

**Date:** 2025-12-28
**Status:** Approved

## Overview

A Claude Code plugin that provides a `/learn <topic>` command. It researches any topic via web search and generates a mini-plugin containing Claude-optimized skills.

**Examples:**
- `/learn laravel 12` - generates skills for Laravel 12 framework
- `/learn rocket physics` - generates skills for rocket physics domain

## Key Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Skill persistence | Persistent files | Reusable across sessions |
| Storage location | User-level (`~/.claude/plugins/learned/`) | Available across all projects |
| Research sources | Web search only | Simpler, covers most needs |
| Skills per topic | Multiple focused skills | Easier to invoke contextually |
| Organization | Nested directories (mini-plugins) | Extensible, shareable, clear boundaries |
| Generation mode | Fully automatic | Simple UX, no interaction needed |
| Subtopic discovery | Dynamic | Adapts to each topic's landscape |
| Skill content style | Claude-optimized instructions | Behavioral guidance, not documentation |
| Domain scope | General purpose | Works for any topic, not just programming |
| Re-run behavior | Update/merge | Preserves user edits, adds new content |
| Distribution | GitHub (plugin only) | Generated skills stay local |

## Directory Structure

### The Plugin (distributed via GitHub)

```
github.com/yourname/claude-learner/
├── plugin.json
├── README.md
├── commands/
│   └── learn.md
└── skills/
    └── using-learner/
        └── SKILL.md
```

### Generated Output (local)

```
~/.claude/plugins/learned/
└── <topic>/
    ├── plugin.json
    └── skills/
        ├── <subtopic-1>/
        │   └── SKILL.md
        ├── <subtopic-2>/
        │   └── SKILL.md
        └── <subtopic-n>/
            └── SKILL.md
```

Example for `/learn laravel 12`:

```
~/.claude/plugins/learned/
└── laravel-12/
    ├── plugin.json
    └── skills/
        ├── routing/
        │   └── SKILL.md
        ├── eloquent/
        │   └── SKILL.md
        ├── migrations/
        │   └── SKILL.md
        └── artisan-cli/
            └── SKILL.md
```

## Research Process

### Phase 1: Initial Discovery
- Web search for `<topic> overview`, `<topic> key concepts`, `<topic> documentation`
- Identify what this topic is (framework, science, methodology, etc.)
- Extract main official/authoritative sources

### Phase 2: Subtopic Identification
- Analyze search results to find major subtopics/components
- Aim for 4-8 distinct subtopics
- Examples:
  - "laravel 12" → routing, Eloquent, Blade, migrations, Artisan, etc.
  - "rocket physics" → orbital mechanics, propulsion, aerodynamics, staging, etc.

### Phase 3: Deep Dive per Subtopic
- Run focused searches for each identified subtopic
- Gather: core concepts, common patterns, typical mistakes, best practices
- Prioritize authoritative sources

### Phase 4: Skill Generation
- Transform research into Claude-optimized skill files
- Write as behavioral guidance: "When working with X, do Y, avoid Z"
- Include concrete examples where useful
- Keep each skill focused (300-500 words)

## Skill File Format

```markdown
---
name: <topic>-<subtopic>
description: Use when [triggering conditions for this skill]
generated: YYYY-MM-DD
sources:
  - https://example.com/source1
  - https://example.com/source2
---

# [Subtopic Title]

## When This Skill Applies
- [Condition 1]
- [Condition 2]

## Key Patterns

[Behavioral guidance with concrete patterns to follow]

## Common Mistakes to Avoid

[Anti-patterns and pitfalls]

## Examples

[Concrete examples relevant to the topic]
```

The `description` field is critical - it tells Claude when to invoke this skill.

## Update/Merge Behavior

When `/learn <topic>` runs for an existing topic:

### Detection
- Check if `~/.claude/plugins/learned/<topic>/` exists
- If yes, enter update mode

### Comparison
- Run same research phases as initial generation
- Compare discovered subtopics against existing skills
- Identify: new, obsolete, or changed subtopics

### Merge Strategy
- **New subtopics:** Generate new skill files
- **Existing subtopics:** Update if research shows significant new information
- **Obsolete subtopics:** Keep but mark as "not found in latest research"
- **User edits:** Preserve custom additions

### Metadata
- `plugin.json` stores `lastUpdated` timestamp
- Each skill stores `generated` date and `sources`

### Output
- Display summary: "Updated 3 skills, added 1 new skill, 2 skills unchanged"

## Plugin Manifest Format

### Main Plugin (`plugin.json`)

```json
{
  "name": "claude-learner",
  "description": "Research any topic and generate Claude-optimized skills",
  "version": "1.0.0",
  "author": {
    "name": "Your Name"
  },
  "repository": "https://github.com/yourname/claude-learner"
}
```

### Generated Mini-Plugin (`plugin.json`)

```json
{
  "name": "laravel-12",
  "description": "Skills for working with Laravel 12 framework",
  "generated": "2025-12-28",
  "lastUpdated": "2025-12-28",
  "topic": "laravel 12",
  "skillCount": 5
}
```

## Deferred Enhancements

Intentionally not included in v1:
- GitHub repository research (code examples, READMEs)
- Project-level skills (`--local` flag)
- Interactive subtopic selection
- Sharing learned topics with others via GitHub
- Configurable research sources

## Installation

```bash
claude plugins add yourname/claude-learner
```

## Usage

```bash
/learn laravel 12
/learn rocket physics
/learn kubernetes networking
```
