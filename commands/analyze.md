---
name: analyze
description: Analyze current project and generate domain knowledge skills
arguments:
  - name: focus
    description: Optional area to focus on (e.g., "billing", "authentication")
    required: false
---

# Analyze: Generate Domain Knowledge Skills

You are executing the `/analyze` command to examine the current project and generate skills capturing its domain knowledge, business logic, and key concepts.

## Output Location

All generated skills go to `.claude/skills/` (project-level).

Skills are named: `{{domain}}/SKILL.md`

## Mode Selection

**Determine mode based on focus argument:**
- If `{{focus}}` is provided: Execute **Focused Analysis** (Phase 2)
- Otherwise: Execute **Full Discovery** (Phase 1)

---

## Phase 1: Full Discovery (No Focus)

### 1.1 Initial Codebase Scan

Explore the project structure to understand what exists:

1. **Read project context files:**
   - README.md, README, or similar
   - package.json, pyproject.toml, Cargo.toml, or equivalent
   - Any docs/ directory content

2. **Examine directory structure:**
   - Use `ls` and file globbing to understand layout
   - Identify major directories (src/, lib/, app/, etc.)
   - Note naming patterns and organization style

3. **Identify key files:**
   - Entry points (main.*, index.*, app.*)
   - Models, schemas, types
   - Controllers, handlers, routes
   - Services, utilities, helpers

### 1.2 Domain Identification

From the scan, identify distinct domain areas:

For each potential domain, ask:
- Does this represent a coherent area of business logic?
- Is it distinct from other identified domains?
- Are there enough files/concepts to warrant a skill?
- Can I identify specific trigger phrases for when users would work on this?

**What makes a good domain:**
- Clear responsibility boundary
- Multiple related concepts/files
- Distinct from other domains
- Meaningful business or technical function

**Examples of domains:**
- "authentication" - login, sessions, permissions
- "billing" - payments, invoices, subscriptions
- "notifications" - emails, alerts, messaging
- "data-import" - file parsing, validation, transformation

### 1.3 Deduplication

Before presenting, verify domains are distinct:

For each pair of domains:
- Do they cover the same concepts? → Merge them
- Would the same user task trigger both? → Clarify boundaries or merge
- Is one a subset of the other? → Absorb smaller into larger

Output reasoning:
```
Verified {{N}} domains are distinct:
- "auth" vs "users": Distinct - auth is login/sessions, users is profile/preferences
- "api" vs "routes": MERGED → "api" (routes is implementation detail of api)
```

### 1.4 Preview and Confirm

Present discovered domains to user:

```
📋 Discovered domains in this project:

1. {{domain-1}}: [one-line description]
2. {{domain-2}}: [one-line description]
3. ...

Proceed with generation? Options:
- "yes" or "y" → Generate all skills
- "skip 2,4" → Generate all except skills 2 and 4
- "only 1,3" → Generate only skills 1 and 3
- "no" → Cancel generation
```

Wait for user confirmation before proceeding.

### 1.5 Generate Skills

For each confirmed domain, proceed to **Phase 3: Skill Generation**.

---

## Phase 2: Focused Analysis (With Focus)

When user provides `{{focus}}`:

### 2.1 Locate the Domain

Search for files/directories matching "{{focus}}":

1. **Direct matches:**
   - Directories named `{{focus}}` or containing `{{focus}}`
   - Files with `{{focus}}` in the name

2. **Related patterns:**
   - Similar terms (e.g., "billing" → also check "payment", "invoice")
   - Plural/singular variants
   - Common abbreviations

3. **Content search:**
   - Grep for `{{focus}}` in file contents
   - Look for classes, functions, comments mentioning the term

### 2.2 Handle Not Found

If no matching files found:

```
❌ Could not find domain "{{focus}}" in this project.

Discovered domains:
- {{domain-1}}
- {{domain-2}}
- {{domain-3}}

Try: /analyze {{domain-1}}
```

Then stop execution.

### 2.3 Deep Dive

For the focused area, read files in detail:

1. **Read all relevant files** (not just skim structure)
2. **Trace relationships:**
   - What does this domain depend on?
   - What depends on this domain?
3. **Understand data flow:**
   - How does data enter this domain?
   - How is it transformed?
   - Where does it go?
4. **Identify business rules:**
   - Validation logic
   - Conditional behavior
   - Edge cases handled

### 2.4 Generate Detailed Skill

Proceed to **Phase 3: Skill Generation** with deeper detail:
- Target 2,000-3,000 words (more than discovery mode)
- Include specific code examples
- Document more edge cases and patterns

---

## Phase 3: Skill Generation

### 3.1 Create Skill Directory

```bash
mkdir -p .claude/skills/{{domain}}
```

### 3.2 Write SKILL.md

Create `.claude/skills/{{domain}}/SKILL.md`:

```markdown
---
name: {{domain}}
description: This skill should be used when the user [specific scenarios from code analysis]. Examples: "when working with {{domain}}", "modifying {{key-concept}}", "debugging {{common-issue}}".
analyzed: {{YYYY-MM-DD}}
source_files:
  - {{file-1}}
  - {{file-2}}
---

# {{Domain Title}}

## What This Domain Does

[1-2 paragraph explanation of this area's purpose and responsibility, derived from code analysis]

## Key Concepts

- **[Concept 1]**: [What it is, how it's represented in code, key files]
- **[Concept 2]**: [What it is, how it's represented in code, key files]
- **[Concept 3]**: [What it is, how it's represented in code, key files]

## How It Works

[Describe the main flows, data transformations, key relationships between components]

[Include specific details about:
- Entry points into this domain
- Core processing logic
- Output/side effects
- Key decision points]

## Important Files

- `path/to/file`: [What it contains, why it matters, key exports]
- `path/to/other`: [What it contains, why it matters, key exports]

## Working With This Domain

[Behavioral guidance for when Claude (or developers) work on this area:]

- When modifying X, ensure Y
- The relationship between A and B means changes to A require...
- Common gotchas: [specific things to watch out for]
- Testing considerations: [how to verify changes]
```

### 3.3 Skill Writing Guidelines

**Description Field (Critical for Triggering):**
- Use third person: "This skill should be used when the user..."
- Include specific trigger phrases derived from actual code concepts
- Be concrete: reference real classes, functions, file patterns from the project

**Content Quality:**
- Ground everything in actual code (cite files, functions, classes)
- Explain the "why" behind patterns found in the code
- Focus on what someone needs to know to work effectively in this area
- Include specific file paths that are entry points

**Target Size:**
- Full discovery mode: 1,000-2,000 words per skill
- Focused analysis mode: 2,000-3,000 words

---

## Phase 4: Report Results

After generation, report:

```
✓ Analyzed project and generated domain skills

  Location: .claude/skills/

  Skills created:
  - {{domain-1}}: [brief description]
  - {{domain-2}}: [brief description]
  - ...

  Total: {{N}} skills

  Restart Claude Code to load these skills. They will be
  automatically invoked when working on related areas.
```

If this was a focused analysis:
```
✓ Generated domain skill for "{{focus}}"

  Location: .claude/skills/{{focus}}/SKILL.md

  This skill captures:
  - [key concept 1]
  - [key concept 2]
  - [key concept 3]

  Restart Claude Code to load this skill.
```
