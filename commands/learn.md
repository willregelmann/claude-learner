---
name: learn
description: Research a topic and generate a Claude-optimized skill
argument-hint: <topic> [--global]
---

# Learn: Research and Generate a Skill

You are executing the `/learn` command. First, parse the arguments from: `$ARGUMENTS`

## Argument Parsing

1. **Extract the topic**: Everything in `$ARGUMENTS` except `--global` flag
2. **Check for --global flag**: If `$ARGUMENTS` contains `--global`, set GLOBAL_MODE=true
3. **Validate**: If no topic provided, ask user what topic to research

Store the parsed topic as TOPIC.

## Output Location

- If GLOBAL_MODE is true: Use `~/.claude/skills/` (user-level)
- Otherwise: Use `./.claude/skills/` (project directory, default)

Store as `SKILLS_DIR`.

## Phase 1: Research

### 1.1 Compute Topic Slug

1. Take TOPIC, replace spaces with hyphens, lowercase, remove special characters
2. Store as TOPIC_SLUG (e.g., "Laravel 12" → "laravel-12")

### 1.2 Check for Existing Skill

```bash
ls $SKILLS_DIR | grep "^TOPIC_SLUG$" 2>/dev/null
```

If exists, inform user it will be replaced.

### 1.3 Research the Topic

Use WebSearch to gather comprehensive information:
- Search: "TOPIC best practices"
- Search: "TOPIC common mistakes"
- Search: "TOPIC tutorial"
- Search: "TOPIC official documentation"

Gather:
- Key patterns and best practices
- Common mistakes to avoid
- Concrete examples
- Trigger phrases users would say

## Phase 2: Generate ONE Skill

Create the skill directory and file:

```bash
mkdir -p $SKILLS_DIR/TOPIC_SLUG
```

Write `$SKILLS_DIR/TOPIC_SLUG/SKILL.md`:

```markdown
---
name: TOPIC_SLUG
description: [WHAT it does]. Use when [WHEN to use it]. Examples: "[trigger 1]", "[trigger 2]", "[trigger 3]".
generated: {{YYYY-MM-DD}}
sources:
  - [URL 1]
  - [URL 2]
---

# [Topic Title]

## When This Skill Applies

- [Scenario 1]
- [Scenario 2]
- [Scenario 3]

## Key Patterns

[Behavioral guidance organized by concept]
[Use imperative form: "To do X, do Y"]
[Include code examples where helpful]

## Common Mistakes to Avoid

- **[Mistake 1]**: [Why wrong] → [What to do instead]
- **[Mistake 2]**: [Why wrong] → [What to do instead]

## Examples

[1-2 concrete examples demonstrating key patterns]
```

**Writing Guidelines:**
- **Description**: Include WHAT + WHEN + trigger phrases
- **Content**: Behavioral guidance, not reference documentation
- **Length**: Keep under 500 lines; be concise
- **Style**: Imperative form ("Do X" not "You should do X")

## Phase 3: Report

```
✓ Generated skill for "TOPIC"
  Location: $SKILLS_DIR/TOPIC_SLUG/SKILL.md

  Restart Claude Code to load this skill.
```
