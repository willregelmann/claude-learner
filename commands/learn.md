---
name: learn
description: Research a topic and generate Claude-optimized skills
arguments:
  - name: topic
    description: The topic to research (e.g., "laravel 12", "rocket physics")
    required: true
---

# Learn: Research and Generate Skills

You are executing the `/learn` command to research "{{topic}}" and generate Claude-optimized skills.

## Output Location

All generated skills go to: `~/.claude/skills/`

Skills are named with the topic as a prefix: `{{topic-slug}}-{{subtopic-slug}}/SKILL.md`

This location is auto-discovered by Claude Code - no plugin installation required.

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

### 1.1 Check for Existing Topic

Check if this topic already has generated skills:

```bash
ls ~/.claude/skills/ | grep "^{{topic-slug}}-" 2>/dev/null
```

If matching directories exist, you are in **UPDATE MODE**. Read existing skills to understand what's already there before researching.

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

Based on initial research, identify 4-8 major subtopics that deserve their own skills.

For each potential subtopic, ask:
- Is this distinct enough to warrant its own skill?
- Would Claude benefit from specific guidance on this?
- Is there enough substance for 300-500 words of guidance?

Examples of good subtopic breakdown:
- "laravel 12" → routing, eloquent-orm, blade-templates, migrations, artisan-cli, authentication
- "rocket physics" → orbital-mechanics, propulsion-systems, aerodynamics, staging, trajectory-planning
- "kubernetes" → pods-deployments, services-networking, storage, configuration, troubleshooting

## Phase 3: Deep Research per Subtopic

For each identified subtopic:
1. Use WebSearch to find detailed information
2. Focus on: best practices, common patterns, typical mistakes, key concepts
3. Prioritize authoritative sources

Gather enough information to write Claude-optimized guidance (not documentation).

## Phase 4: Generate Skills

### 4.1 Create Each Skill

For each subtopic, create `~/.claude/skills/{{topic-slug}}-{{subtopic-slug}}/SKILL.md`:

```bash
mkdir -p ~/.claude/skills/{{topic-slug}}-{{subtopic-slug}}
```

Then write the SKILL.md file:

```markdown
---
name: {{topic-slug}}-{{subtopic-slug}}
description: Use when [specific triggering conditions - be precise about when this skill applies]
generated: {{YYYY-MM-DD}}
sources:
  - [URL 1]
  - [URL 2]
---

# {{Subtopic Title}}

## When This Skill Applies

- [Specific condition 1]
- [Specific condition 2]
- [Specific condition 3]

## Key Patterns

[Write behavioral guidance for Claude: "When doing X, always Y", "Prefer A over B because...", "The correct approach is..."]

[Include concrete patterns, not abstract explanations]

## Common Mistakes to Avoid

- [Mistake 1]: [Why it's wrong and what to do instead]
- [Mistake 2]: [Why it's wrong and what to do instead]

## Examples

[Provide 1-2 concrete examples that demonstrate the key patterns]
```

**Skill Writing Guidelines:**
- Write FOR Claude, not for humans reading documentation
- Focus on behavioral guidance: what to do, what to avoid
- Be specific and actionable, not theoretical
- Keep each skill focused (300-500 words)
- The `description` field is critical - it determines when Claude invokes the skill

### 4.2 Update Mode Handling

If in UPDATE MODE:
- Compare new subtopics with existing skills
- For existing subtopics: Update content if research shows significant new information
- For new subtopics: Create new skill files
- For obsolete subtopics: Add `obsolete: true` to frontmatter, keep file
- Preserve any custom user additions (look for content not matching original generated patterns)

## Phase 5: Report Results

After generation, report:

```
✓ Generated skills for "{{topic}}"
  Location: ~/.claude/skills/

  Skills created:
  - {{topic-slug}}-{{subtopic-1}}: [brief description]
  - {{topic-slug}}-{{subtopic-2}}: [brief description]
  - ...

  Total: {{N}} skills

  Restart Claude Code to load these skills. They will be automatically
  invoked when working on related tasks.
```

If UPDATE MODE, also report:
- Skills updated: N
- Skills added: N
- Skills unchanged: N
