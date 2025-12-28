# Claude Learner

A Claude Code plugin that researches any topic and generates Claude-optimized skills.

## Installation

```bash
claude plugins add willcodefortea/claude-learner
```

## Usage

```bash
/learn <topic>
```

### Examples

```bash
/learn laravel 12        # Framework skills
/learn rocket physics    # Science domain skills
/learn kubernetes        # DevOps skills
/learn negotiation       # Soft skills
```

## What It Does

1. **Researches** the topic using web search
2. **Identifies** 4-8 key subtopics
3. **Generates** Claude-optimized skills for each subtopic
4. **Saves** skills to `~/.claude/plugins/learned/<topic>/`

Generated skills are automatically available in future Claude Code sessions.

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
