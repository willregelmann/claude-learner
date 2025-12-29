# Claude Learner

A Claude Code plugin that researches any topic and generates Claude-optimized skills.

## Installation

In a Claude Code session, ask Claude to add the marketplace:

```
Add the marketplace: git@github.com:willregelmann/claude-learner.git
```

Or provide the marketplace information directly in your project's `.claude/config.toml`.

## Usage

```bash
/learn <topic>                    # Generate user-level skills
/learn <topic> --project         # Generate project-level skills
```

### Examples

```bash
/learn laravel 12                # Framework skills (user-level)
/learn rocket physics --project  # Science skills (project-level)
/learn kubernetes                # DevOps skills (user-level)
/learn negotiation               # Soft skills (user-level)
```

### Skill Locations

- **User-level (default)**: `~/.claude/skills/` - Available across all projects
- **Project-level (`--project`)**: `./.claude/skills/` - Scoped to current project only

## What It Does

1. **Researches** the topic using web search
2. **Identifies** 4-8 key subtopics
3. **Generates** Claude-optimized skills for each subtopic
4. **Saves** skills to the appropriate location (user or project-level)

**Restart Claude Code** to load the newly generated skills.

## Generated Skill Format

Each skill contains:
- **When to apply**: Specific triggering conditions
- **Key patterns**: Behavioral guidance for Claude
- **Mistakes to avoid**: Common anti-patterns
- **Examples**: Concrete demonstrations

## Updating Skills

Re-run `/learn <topic>` to:
- Update existing skills with new information
- Add skills for newly discovered subtopics
- Preserve your custom edits

## License

MIT
