---
name: learn
description: Research a topic and generate Claude-optimized skills
arguments:
  - name: topic
    description: The topic to research (e.g., "laravel 12", "rocket physics")
    required: true
  - name: project
    description: Generate skills in current project (.claude/skills/) instead of user-level (~/.claude/skills/)
    required: false
---

# Learn: Research and Generate Skills

You are executing the `/learn` command to research "{{topic}}" and generate Claude-optimized skills.

## Output Location

**Determine output directory based on --project flag:**
- If `{{project}}` is set: Use `./.claude/skills/` (current project directory)
- Otherwise: Use `~/.claude/skills/` (user-level, default)

Store the chosen path as `SKILLS_DIR` for use throughout this command.

Skills are named with the topic as a prefix: `{{topic-slug}}-{{subtopic-slug}}/SKILL.md`

Both locations are auto-discovered by Claude Code - no plugin installation required.

## Phase 1: Check Existing & Initial Discovery

### 1.0 Compute Topic Slug

Before proceeding, compute the TOPIC_SLUG that will be used throughout this command:

1. Take the topic: "{{topic}}"
2. Replace all spaces with hyphens
3. Convert to lowercase
4. Remove special characters (keep only letters, numbers, hyphens)
5. Store as TOPIC_SLUG

**Example:** "Laravel 12" → "laravel-12"

Use TOPIC_SLUG for all directory paths and file naming in subsequent phases.

### 1.1 Check for Existing Skills

Check if this topic already has generated skills:

```bash
ls $SKILLS_DIR | grep "^{{topic-slug}}-" 2>/dev/null
```

If matching directories exist, inform the user:
```
⚠️ Found existing skills for "{{topic}}":
- {{topic-slug}}-subtopic-1 (generated YYYY-MM-DD)
- {{topic-slug}}-subtopic-2 (generated YYYY-MM-DD)

These will be replaced with newly generated skills.
To keep old versions, rename them first (e.g., add "-old" suffix).
```

This is simpler than update mode - fresh generation each time avoids merge complexity.

### 1.2 Initial Research

Use WebSearch to understand the topic:
- Search: "{{topic}} overview"
- Search: "{{topic}} key concepts"
- Search: "{{topic}} official documentation"

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

## Phase 2.5: Preview and Confirm

Before proceeding with deep research, present the proposed skill plan to the user:

```
📋 Skill Plan for "{{topic}}"

Proposed skills ({{N}} total):
1. {{topic-slug}}-{{subtopic-1}}: [one-line description]
2. {{topic-slug}}-{{subtopic-2}}: [one-line description]
3. ...

Location: $SKILLS_DIR

Proceed with generation? Options:
- "yes" or "y" → Generate all skills
- "skip 2,4" → Generate all except skills 2 and 4
- "only 1,3" → Generate only skills 1 and 3
- "no" → Cancel generation
```

Wait for user confirmation before proceeding. This allows users to:
- Review scope before expensive API calls
- Exclude subtopics they don't need
- Abort if research quality was poor
- Understand what will be generated

If user provides no response within reasonable time, default to "yes" for non-interactive sessions.

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

For each subtopic, create `$SKILLS_DIR/{{topic-slug}}-{{subtopic-slug}}/SKILL.md`:

```bash
mkdir -p $SKILLS_DIR/{{topic-slug}}-{{subtopic-slug}}
```

Then write the SKILL.md file:

```markdown
---
name: {{topic-slug}}-{{subtopic-slug}}
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

### 4.2 Replace Existing Skills

If existing skills were found in Phase 1.1:
- Delete the old skill directories before writing new ones
- This ensures clean generation without merge complexity
- Users who want to preserve old versions should rename them beforehand

## Phase 5: Report Results

After generation, report:

```
✓ Generated skills for "{{topic}}"
  Location: $SKILLS_DIR

  Skills created:
  - {{topic-slug}}-{{subtopic-1}}: [brief description]
  - {{topic-slug}}-{{subtopic-2}}: [brief description]
  - ...

  Total: {{N}} skills

  Restart Claude Code to load these skills. They will be automatically
  invoked when working on related tasks.
```

If existing skills were replaced, also report:
- Previous skills removed: N
- New skills generated: N
