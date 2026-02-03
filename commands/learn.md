---
name: learn
description: Research a topic and generate Claude-optimized skills
argument-hint: <topic> [--global]
---

# Learn: Research and Generate Skills

You are executing the `/learn` command. First, parse the arguments from: `$ARGUMENTS`

## Argument Parsing

1. **Extract the topic**: Everything in `$ARGUMENTS` except `--global` flag
   - Example: `laravel 12 --global` → topic is "laravel 12"
   - Example: `react hooks` → topic is "react hooks"

2. **Check for --global flag**: If `$ARGUMENTS` contains `--global`, set GLOBAL_MODE=true

3. **Validate**: If no topic provided (empty after removing --global), ask user what topic to research

Store the parsed topic as TOPIC for use throughout this command.

## Output Location

**Determine output directory based on --global flag:**
- If GLOBAL_MODE is true: Use `~/.claude/skills/` (user-level)
- Otherwise: Use `./.claude/skills/` (current project directory, default)

Store the chosen path as `SKILLS_DIR` for use throughout this command.

Both locations are auto-discovered by Claude Code - no plugin installation required.

## Phase 1: Check Existing & Initial Discovery

### 1.1 List Existing Skills

Before generating new skills, list ALL existing skills in `$SKILLS_DIR` to understand what's already available:

```bash
ls -la $SKILLS_DIR 2>/dev/null || echo "No skills directory yet"
```

Present the list to the user and identify any that might be related to TOPIC:
- Look for **exact matches**: Skills that clearly cover the same topic
- Look for **close matches**: Skills with similar names, overlapping concepts, or related domains
- Look for **potential conflicts**: Skills that might cover similar ground

```
📂 Existing skills in $SKILLS_DIR:
- kubernetes-networking (generated 2024-01-15)
- kubernetes-pods (generated 2024-01-15)
- react-hooks (generated 2024-02-01)
- ...

🔍 Potentially related to "TOPIC":
- [list any that seem related, even loosely]

If any existing skills should be replaced or updated, let me know.
Otherwise, I'll generate new skills alongside these.
```

This allows the user to see what exists and decide whether to replace, update, or add to their skill collection.

### 1.2 Initial Research

Use WebSearch to understand the topic:
- Search: "TOPIC overview"
- Search: "TOPIC key concepts"
- Search: "TOPIC official documentation"

From these results, determine:
- What type of topic this is (framework, library, science, methodology, etc.)
- The main authoritative sources (official docs, key references)
- A one-sentence description of the topic

**If web search returns poor or no results:**
- Try alternative search terms (e.g., full name vs acronym, different spellings)
- Search for related/parent topics to find context
- If the topic appears to be very niche or proprietary, inform the user that limited public information is available and ask if they can provide documentation or context
- If no useful information can be found after 3-4 search attempts, report this to the user rather than generating low-quality skills

## Phase 2: Subtopic Identification

Based on initial research, identify the major subtopics that deserve their own skills. Let the topic determine granularity naturally - don't force a fixed number.

For each potential subtopic, ask:
- Is this distinct enough to warrant its own skill?
- Would Claude benefit from specific behavioral guidance on this?
- Is there enough substance for 1,500-2,000 words of focused guidance?
- Can I identify concrete trigger phrases users would say?
- Does this overlap significantly with another subtopic? (If yes, merge them)

**Examples of natural granularity:**
- "laravel 12" → 8-12 subtopics (large framework with many distinct areas)
- "git branching strategies" → 2-4 subtopics (focused topic, fewer natural divisions)
- "kubernetes networking" → 5-7 subtopics (medium scope, distinct networking concepts)

**Each subtopic should have:**
- Clear triggering scenarios (when users work on this specific aspect)
- Distinct patterns and best practices
- Common mistakes specific to that subtopic
- Concrete examples demonstrating key concepts
- Minimal overlap with other subtopics

**Skill Naming:**
Choose clear, descriptive names for each skill. Names should be:
- Meaningful and self-documenting (e.g., `eloquent-relationships` not `laravel-12-eloquent-relationships`)
- Concise but specific enough to distinguish from related skills
- Lowercase with hyphens (e.g., `react-state-management`, `kubernetes-ingress`)

Don't mechanically prefix with the topic - use your judgment to pick the most useful name.

### Deduplication Verification

Before presenting to the user, verify subtopics are truly distinct:

For each pair of subtopics, ask:
- Do these cover the same concepts from different angles? → Merge them
- Would the same user question trigger both? → Merge or clarify boundaries
- Is one a subset of the other? → Absorb the smaller into the larger

Output your reasoning:
```
Verified {{N}} subtopics are distinct:
- "routing" vs "middleware": Distinct - routing is URL→controller, middleware is request filtering
- "eloquent" vs "migrations": Distinct - ORM usage vs schema management
- "validation" vs "forms": MERGED → "form-validation" (too much overlap)
```

This catches overlap before expensive deep research, not after.

## Phase 2.5: Preview

Before proceeding with deep research, present the proposed skill plan to the user:

```
📋 Skill Plan for "TOPIC"

Generating skills ({{N}} total):
1. {{skill-name-1}}: [one-line description]
2. {{skill-name-2}}: [one-line description]
3. ...

Location: $SKILLS_DIR
```

Then proceed automatically with generation.

## Phase 3: Deep Research per Subtopic

For each identified subtopic:
1. Use WebSearch to find detailed, actionable information
2. Focus on: best practices, common patterns, typical mistakes, key concepts, gotchas
3. Prioritize authoritative sources (official docs, recognized experts)
4. Look for concrete examples and code patterns
5. Identify specific trigger phrases users would say for this subtopic

**Research Quality Criteria:**
- Find **behavioral guidance**: What to do, what to avoid, why
- Identify **common mistakes**: Real anti-patterns developers encounter
- Gather **concrete examples**: Working code, specific scenarios
- Note **trigger phrases**: How users describe tasks in this area

Gather enough information to write Claude-optimized skills (behavioral guidance, not reference documentation).
Target 1,500-2,000 words of focused, actionable content per skill.

## Phase 4: Generate Skills

### 4.1 Create Each Skill

For each subtopic, create `$SKILLS_DIR/{{skill-name}}/SKILL.md`:

```bash
mkdir -p $SKILLS_DIR/{{skill-name}}
```

Then write the SKILL.md file:

```markdown
---
name: {{skill-name}}
description: This skill should be used when the user [specific scenarios with exact phrases]. Examples: "when working with X", "implementing Y", "configuring Z". [Be concrete and specific with trigger phrases]
generated: {{YYYY-MM-DD}}
sources:
  - [URL 1]
  - [URL 2]
---

# {{Subtopic Title}}

## When This Skill Applies

- [Specific concrete scenario 1]
- [Specific concrete scenario 2]
- [Specific concrete scenario 3]

## Key Patterns

[Write behavioral guidance using imperative/infinitive form (verb-first instructions)]
[Example: "To accomplish X, do Y" not "You should do X"]
[Example: "Prefer A over B because..." not "You should prefer A"]
[Example: "Validate input before processing" not "You should validate input"]

[Include concrete, actionable patterns - not abstract theory]

## Common Mistakes to Avoid

- **[Mistake 1]**: [Why it's wrong] → [What to do instead]
- **[Mistake 2]**: [Why it's wrong] → [What to do instead]

[Use imperative form: "Don't do X" not "You shouldn't do X"]

## Examples

[Provide 1-2 concrete examples demonstrating the key patterns]
[Use code blocks, command examples, or specific scenarios]
```

**Skill Writing Guidelines (Following Claude Code Best Practices):**

**Description Field (Critical for Triggering):**
- Use **third person**: "This skill should be used when the user..."
- Include **specific trigger phrases**: Exact words/phrases users would say
- Be **concrete**: "when implementing X", "configuring Y", "debugging Z"
- Avoid vague: Don't write "provides guidance on X" - specify when to use it

**Writing Style:**
- Use **imperative/infinitive form**: "To do X, do Y" not "You should do X"
- Write **FOR Claude**: Behavioral guidance, not human documentation
- Be **objective**: "Validate input" not "You should validate input"
- Stay **actionable**: Concrete patterns, not abstract theory

**Content Organization:**
- Keep skills **lean**: Target 1,500-2,000 words, max 3,000 words
- **Focus**: One subtopic per skill, specific behavioral guidance
- **Structure**: When to use, key patterns, mistakes to avoid, examples
- **Source attribution**: Include research URLs for credibility

### 4.2 Handle Existing Skills

If existing skills were identified as related in Phase 1.1 and the user confirmed they should be replaced:
- Delete those specific skill directories before writing the replacements
- Only delete skills the user explicitly agreed to replace
- Skills not mentioned are left untouched

## Phase 5: Report Results

After generation, report:

```
✓ Generated skills for "TOPIC"
  Location: $SKILLS_DIR

  Skills created:
  - {{skill-name-1}}: [brief description]
  - {{skill-name-2}}: [brief description]
  - ...

  Total: {{N}} skills

  Restart Claude Code to load these skills. They will be automatically
  invoked when working on related tasks.
```

If existing skills were replaced, also report:
- Previous skills replaced: [list names]
- New skills generated: N
