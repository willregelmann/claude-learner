---
name: skill-generation
description: This skill should be used when the user asks to "modify the /learn command", "change how skills are generated", "update skill format", "fix skill generation", "add a new phase to /learn", or when working on commands/learn.md or the skill generation pipeline.
analyzed: 2025-12-28
source_files:
  - commands/learn.md
  - skills/using-learner/SKILL.md
---

# Skill Generation Domain

## What This Domain Does

The skill generation system researches any topic via web search and produces Claude-optimized skills. The `/learn` command orchestrates a multi-phase pipeline: initial research, subtopic identification, deduplication verification, user confirmation, deep research per subtopic, and skill file generation. Skills are written FOR Claude consumption using imperative form, with strong trigger descriptions that determine when Claude will invoke them.

The output location is configurable: user-level (`~/.claude/skills/`) for global availability, or project-level (`./.claude/skills/`) for scoped knowledge. Both locations are auto-discovered by Claude Code.

## Key Concepts

- **Skill Naming**: Names are chosen by the agent to be meaningful and self-documenting (e.g., `eloquent-relationships` not `laravel-12-eloquent-relationships`). No mechanical prefixing from the topic - use judgment to pick the most useful name.

- **Existing Skill Detection**: Before generating, list ALL existing skills and identify close matches - skills with similar names, overlapping concepts, or related domains. Let user decide what to replace vs. keep.

- **Subtopic Identification**: Breaking a broad topic into distinct, non-overlapping areas. Each subtopic should have clear trigger scenarios, distinct patterns, and enough substance for 1,500-2,000 words.

- **Deduplication Verification**: Before presenting to user, verify subtopics don't overlap. Merge if same concepts from different angles, same user question triggers both, or one is subset of another.

- **Preview and Confirm**: User sees proposed skills before expensive research. Can approve all, skip specific numbers, select only specific numbers, or cancel entirely.

- **Skill Writing Guidelines**: Third-person descriptions ("This skill should be used when the user..."), imperative form in body ("Validate input" not "You should validate"), specific trigger phrases, source attribution.

## How It Works

### Entry Point
`commands/learn.md` - The slash command definition with YAML frontmatter defining arguments (topic required, project optional).

### Phase Flow
1. **List Existing Skills** - Show all skills, identify related/close matches
2. **Initial Research** - WebSearch for overview, key concepts, official docs
3. **Subtopic Identification** - Break into distinct areas, choose meaningful names
4. **Deduplication** - Verify no overlap between subtopics
5. **Preview and Confirm** - Present plan, wait for user approval
6. **Deep Research** - WebSearch for detailed info per subtopic
7. **Generate Skills** - Write SKILL.md files with proper structure
8. **Report Results** - Summary of what was created

### Skill File Structure
```
$SKILLS_DIR/
├── {{skill-name-1}}/
│   └── SKILL.md
├── {{skill-name-2}}/
│   └── SKILL.md
```

### Data Flow
- Input: Topic string from user (e.g., "laravel 12")
- Transform: Web research → subtopic breakdown → detailed research → structured skills
- Output: Multiple SKILL.md files with frontmatter and behavioral guidance

## Important Files

- `commands/learn.md`: The main command definition. Contains all phase logic as markdown instructions that Claude interprets and executes. ~270 lines covering the complete pipeline.

- `skills/using-learner/SKILL.md`: Documentation skill that teaches users how to invoke `/learn`. Includes examples, flags, troubleshooting. This is a meta-skill about using the plugin.

## Working With This Domain

### When Modifying the Pipeline

- Phases are numbered and sequential - maintain order when adding steps
- User confirmation (Phase 2.5) is critical for user control - preserve it
- Deduplication happens BEFORE user preview to reduce noise
- Each phase has clear inputs and outputs - document when adding new phases

### Skill Quality Standards

- Description field is CRITICAL for triggering - must include specific phrases users would say
- Body uses imperative form throughout, not second person
- Target 1,500-2,000 words per skill (max 3,000)
- Include source URLs for credibility
- Structure: When to use → Key patterns → Mistakes to avoid → Examples

### Common Gotchas

- Skill names should be meaningful and concise - let the agent choose rather than mechanically deriving from topic
- Web search can fail - handle gracefully with alternative terms and user notification
- Only replace skills user explicitly confirms - don't auto-delete based on loose name matching
- Skills load at Claude Code startup - remind users to restart after generation

### Testing Considerations

- Test with various topic types: frameworks (laravel), science (physics), methodologies (scrum)
- Verify skill naming produces clear, useful names
- Check both output locations work: user-level and project-level
- Confirm preview shows accurate count and descriptions
