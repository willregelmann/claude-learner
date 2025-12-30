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
/learn <topic>                    # Generate project-level skills (default)
/learn <topic> --global           # Generate user-level skills
/analyze                          # Analyze project, generate domain skills
/analyze <focus-area>             # Deep-dive into specific domain
```

### Examples

```bash
/learn laravel 12                 # Framework skills (project-level)
/learn kubernetes --global        # DevOps skills (user-level, all projects)
/learn negotiation                # Soft skills (project-level)
/analyze                          # Discover and document project domains
/analyze billing                  # Deep-dive into billing domain
```

### Skill Locations

- **Project-level (default)**: `./.claude/skills/` - Scoped to current project
- **User-level (`--global`)**: `~/.claude/skills/` - Available across all projects

## What It Does

### `/learn` - Research External Topics

1. **Researches** the topic using web search
2. **Identifies** key subtopics (typically 4-12, depending on scope)
3. **Generates** Claude-optimized skills for each subtopic
4. **Saves** skills to the appropriate location

### `/analyze` - Document Your Codebase

1. **Scans** project structure and key files
2. **Identifies** distinct domains (auth, billing, notifications, etc.)
3. **Generates** skills capturing domain knowledge and patterns
4. **Saves** skills to `.claude/skills/` (project-level only)

Use `/analyze <focus-area>` for deeper analysis of a specific domain.

**Restart Claude Code** to load newly generated skills.

## Generated Skill Format

Each skill contains:
- **When to apply**: Specific triggering conditions
- **Key patterns**: Behavioral guidance for Claude
- **Mistakes to avoid**: Common anti-patterns
- **Examples**: Concrete demonstrations

## Updating Skills

Re-run `/learn <topic>` or `/analyze` to regenerate skills. Existing skills for that topic/domain are replaced with fresh versions.

## License

MIT
