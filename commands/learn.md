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

Both locations are auto-discovered by Claude Code - no plugin installation required.

1. Take TOPIC, replace spaces with hyphens, lowercase, remove special characters
2. Store as TOPIC_SLUG (e.g., "Laravel 12" → "laravel-12")

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

Gather:
- Key patterns and best practices
- Common mistakes to avoid
- Concrete examples
- Trigger phrases users would say

## Phase 2: Generate ONE Skill

Create the skill directory and file:

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

Write `$SKILLS_DIR/TOPIC_SLUG/SKILL.md`:

```markdown
---
name: {{skill-name}}
description: This skill should be used when the user [specific scenarios with exact phrases]. Examples: "when working with X", "implementing Y", "configuring Z". [Be concrete and specific with trigger phrases]
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

## Critical Gotchas

[The 3-5 highest-impact mistakes that WILL break things if missed. These are the non-obvious errors that even experienced developers make. Each should be something that causes silent failures, broken functionality, or hard-to-debug issues. Prioritize by: how common the mistake is × how bad the consequence is.]

- **[Gotcha 1]**: [What goes wrong] → [The fix, with code if needed]
- **[Gotcha 2]**: [What goes wrong] → [The fix, with code if needed]
- **[Gotcha 3]**: [What goes wrong] → [The fix, with code if needed]

## Key Patterns

[Behavioral guidance organized by concept]
[Use imperative form: "To do X, do Y"]
[Include code examples where helpful]

## Common Mistakes to Avoid

[Additional mistakes beyond the critical gotchas. Limit to 5-7 items, ordered by impact. If you have more than 7, merge related ones or drop the least impactful.]

- **[Mistake 1]**: [Why wrong] → [What to do instead]
- **[Mistake 2]**: [Why wrong] → [What to do instead]

## Examples

[1-2 concrete examples demonstrating key patterns]
```

**Writing Guidelines:**
- **Description**: Include WHAT + WHEN + trigger phrases
- **Content**: Behavioral guidance, not reference documentation
- **Critical Gotchas section**: The 3-5 most impactful mistakes, placed early for maximum visibility. These are the items that A/B testing shows have the highest ROI — they prevent broken functionality, not just suboptimal code. Prioritize: missing required directives/config > wrong API usage > deprecated patterns > style issues
- **Common Mistakes section**: Cap at 5-7 items, ordered by impact. Merge related mistakes. Cut low-impact items rather than including everything found during research
- **Length**: Keep under 500 lines; be concise
- **Style**: Imperative form ("Do X" not "You should do X")
- **Structure**: When to use → Critical gotchas → Key patterns → Common mistakes → Examples

## Phase 3: Report

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
✓ Generated skill for "TOPIC"
  Location: $SKILLS_DIR/TOPIC_SLUG/SKILL.md

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
