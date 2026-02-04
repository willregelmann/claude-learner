---
name: project-analysis
description: This skill should be used when the user asks to "run /analyze", "analyze the codebase", "generate domain skills from code", "understand project structure", "create skills from this project", "scan for domains", "identify project domains", or when working on commands/analyze.md or the codebase analysis pipeline.
analyzed: 2025-12-29
source_files:
  - commands/analyze.md
---

# Project Analysis Domain

## What This Domain Does

The project analysis system scans a codebase to identify domain areas and generates skills capturing the project's domain knowledge, business logic, and key concepts. Unlike the `/learn` command which researches external topics via web search, `/analyze` examines the actual project files to produce skills grounded in the codebase itself.

The `/analyze` command operates in two modes: Full Discovery (no arguments) scans the entire project to identify all domain areas, while Focused Analysis (with a focus area argument) deep-dives into a specific domain. Output is always project-level at `.claude/skills/` since domain knowledge is project-specific.

## Key Concepts

- **Full Discovery Mode**: Triggered when no focus area is provided. Scans README, package files, directory structure to identify distinct domain areas. Results in multiple domain skills generated from a single analysis pass.

- **Focused Analysis Mode**: Triggered when user provides a focus area (e.g., `/analyze billing`). Performs deep-dive into specific domain: reads all relevant files, traces relationships, understands data flow, identifies business rules. Produces one detailed skill (2,000-3,000 words vs 1,000-2,000 for discovery).

- **Domain Identification**: Analyzing project structure to find coherent areas of business logic. Good domains have clear responsibility boundaries, multiple related files/concepts, distinct from other domains, and meaningful business or technical function.

- **Deduplication Verification**: Before presenting discovered domains, verify they don't overlap. Merge if they cover same concepts, same user task triggers both, or one is subset of another.

- **Source File Attribution**: Skills generated from analysis include `source_files` in frontmatter listing the actual project files analyzed. This grounds the skill in specific code.

## How It Works

### Entry Point
`commands/analyze.md` - The slash command definition that orchestrates the analysis pipeline.

### Mode Selection Flow
```
/analyze            → Full Discovery (Phase 1)
/analyze billing    → Focused Analysis (Phase 2)
```

### Full Discovery Pipeline (Phase 1)

1. **Initial Codebase Scan**
   - Read project context: README.md, package.json/pyproject.toml/etc.
   - Examine directory structure with `ls` and file globbing
   - Identify key files: entry points, models, controllers, services

2. **Domain Identification**
   - For each potential domain, evaluate:
     - Coherent area of business logic?
     - Distinct from other domains?
     - Enough files/concepts for a skill?
     - Clear trigger phrases for when users work on it?

3. **Deduplication**
   - Verify domains don't overlap
   - Merge if same concepts or same triggers
   - Absorb subsets into parent domains

4. **Preview**
   - Present discovered domains to user
   - Proceed automatically with generation

5. **Generate Skills**
   - Create skill for each confirmed domain

### Focused Analysis Pipeline (Phase 2)

1. **Locate the Domain**
   - Search for files/directories matching focus area
   - Check related patterns, plural/singular variants
   - Grep for focus area in file contents

2. **Handle Not Found**
   - If no matches, report discovered domains and suggest alternatives

3. **Deep Dive**
   - Read all relevant files in detail
   - Trace relationships: dependencies, dependents
   - Understand data flow: entry, transformation, output
   - Identify business rules: validation, conditionals, edge cases

4. **Generate Detailed Skill**
   - Target 2,000-3,000 words (vs 1,000-2,000 for discovery)
   - Include specific code examples
   - Document more edge cases and patterns

### Skill Generation (Phase 3)

For each domain, create `.claude/skills/{{domain}}/SKILL.md`:

```markdown
---
name: {{domain}}
description: This skill should be used when the user [specific scenarios]
analyzed: {{YYYY-MM-DD}}
source_files:
  - {{file-1}}
  - {{file-2}}
---

# {{Domain Title}}

## What This Domain Does
[1-2 paragraph explanation]

## Key Concepts
[Key domain concepts with file references]

## How It Works
[Main flows, data transformations, key relationships]

## Important Files
[File paths with descriptions]

## Working With This Domain
[Behavioral guidance, gotchas, testing considerations]
```

## Important Files

- `commands/analyze.md`: The main command definition (~290 lines). Contains all phase logic as markdown instructions for argument parsing, mode selection, discovery vs focused analysis, skill generation, and result reporting.

## Working With This Domain

### When Modifying the Analysis Pipeline

- Two distinct modes (discovery vs focused) - changes may affect one or both
- Discovery mode should produce lean skills (1,000-2,000 words)
- Focused mode should produce detailed skills (2,000-3,000 words)
- Always include `source_files` in generated skill frontmatter
- Handle "domain not found" gracefully with suggestions

### Key Differences from /learn

| Aspect | /analyze | /learn |
|--------|----------|--------|
| Data source | Local codebase | Web search |
| Output location | Project-level only | Project or user-level |
| Skill content | Code-grounded | Research-based |
| Frontmatter | `source_files` | `sources` (URLs) |

### Common Gotchas

- Domain identification requires reading actual code, not just file names
- Focused analysis needs a search strategy for finding related files (not just exact matches)
- Deduplication must happen before presenting to user to avoid noise
- Source files list should be accurate - include files actually read
- Skills load at Claude Code startup - remind users to restart

### Testing Considerations

- Test full discovery on projects with varying complexity
- Test focused analysis with exact matches, partial matches, and no matches
- Verify source_files accurately reflects analyzed files
- Check skill quality: descriptions should have concrete trigger phrases
- Ensure discovered domains are truly distinct (not overlapping)
