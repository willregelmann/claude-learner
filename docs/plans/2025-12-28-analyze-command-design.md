# /analyze Command Design

**Date:** 2025-12-28
**Status:** Approved

## Overview

The `/analyze` command examines the current project's codebase and generates domain knowledge skills that capture what the project does, its business logic, and key concepts.

## Command Interface

```yaml
---
name: analyze
description: Analyze current project and generate domain knowledge skills
arguments:
  - name: focus
    description: Optional area to focus on (e.g., "billing", "authentication")
    required: false
---
```

**Usage:**
- `/analyze` → Auto-discover all domains, generate skills for each
- `/analyze billing` → Focus only on the billing domain

**Output location:** Always `.claude/skills/` (project-level only)

**Skill naming:** `{{domain}}/SKILL.md` (e.g., `billing/SKILL.md`)

## Phase 1: Domain Discovery (No Focus)

When no focus is specified, discover domains autonomously:

### 1.1 Codebase Scan
- Read README, docs, and key config files for project context
- Examine directory structure for major modules/areas
- Look at key files: models, controllers, services, core classes
- Parse comments, docstrings, and naming patterns

### 1.2 Domain Identification
- Group related concepts into distinct domains
- Each domain should represent a coherent area of business logic
- Examples: "user management", "payments", "notifications", "reporting"

### 1.3 Deduplication
- Verify domains are distinct (not overlapping)
- Merge closely related areas
- Aim for focused, well-bounded domains

### 1.4 Preview & Confirm
```
📋 Discovered domains in this project:

1. skill-generation: How skills are created from research
2. command-execution: How slash commands work
3. ...

Proceed with generation? (yes/skip 2/only 1,3/no)
```

## Phase 2: Focused Analysis (With Focus)

When user provides a focus (e.g., `/analyze billing`):

### 2.1 Locate the Domain
- Search for files/directories matching the focus term
- Look for related naming patterns
- If not found, report with suggestions from discovered domains

### 2.2 Deep Dive
- Read all relevant files in detail (not just structure)
- Trace relationships and dependencies
- Understand data flow and business rules

### 2.3 Generate Detailed Skill
- More detailed than auto-discovered skills
- Target 2,000-3,000 words for deeper coverage
- Include more code examples and specific patterns

### Error Handling
```
❌ Could not find domain "billing" in this project.

Discovered domains:
- commands
- skills
- research

Try: /analyze commands
```

## Phase 3: Skill Generation

For each domain, generate a skill with this structure:

```markdown
---
name: {{domain}}
description: This skill should be used when the user [specific scenarios derived from code]. Examples: "when working with X", "modifying Y", "debugging Z".
analyzed: {{YYYY-MM-DD}}
source_files:
  - [key files that informed this skill]
---

# {{Domain Title}}

## What This Domain Does

[1-2 paragraph explanation of this area's purpose and responsibility]

## Key Concepts

- **[Concept 1]**: [What it is, how it's represented in code]
- **[Concept 2]**: [What it is, how it's represented in code]

## How It Works

[Describe the main flows, data transformations, key relationships]

## Important Files

- `path/to/file.py`: [What it contains, why it matters]
- `path/to/other.py`: [What it contains, why it matters]

## Working With This Domain

[Behavioral guidance: what to know when modifying this area]
```

**Target size:**
- Auto-discovered: 1,000-2,000 words per skill
- Focused analysis: 2,000-3,000 words

## Phase 4: Report Results

```
✓ Analyzed project and generated domain skills

  Location: .claude/skills/

  Skills created:
  - commands: Slash command execution and argument handling
  - skills: Skill file structure and discovery

  Total: 2 skills

  Restart Claude Code to load these skills.
```

## Key Design Decisions

1. **Domain knowledge only** - Not conventions/patterns/workflows (future expansion)
2. **Autonomous exploration** - No interactive questions during analysis
3. **Project-level only** - Skills stored in `.claude/skills/` with the project
4. **Optional focus** - Full scan by default, or narrow to specific area
5. **Simple naming** - `{{domain}}/SKILL.md` without prefixes
