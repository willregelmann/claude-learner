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

All generated skills go to: `~/.claude/plugins/learned/{{topic-slug}}/`

Where `{{topic-slug}}` is the topic with spaces replaced by hyphens and lowercase (e.g., "Laravel 12" → "laravel-12").

## Phase 1: Check Existing & Initial Discovery

### 1.1 Check for Existing Topic

First, check if this topic already has generated skills:

```bash
ls ~/.claude/plugins/learned/{{topic-slug}}/ 2>/dev/null
```

If the directory exists, you are in **UPDATE MODE**. Read existing skills to understand what's already there before researching.

### 1.2 Initial Research

Use WebSearch to understand the topic:
- Search: "{{topic}} overview"
- Search: "{{topic}} key concepts"
- Search: "{{topic}} official documentation"

From these results, determine:
- What type of topic this is (framework, library, science, methodology, etc.)
- The main authoritative sources (official docs, key references)
- A one-sentence description of the topic

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

### 4.1 Create Directory Structure

```bash
mkdir -p ~/.claude/plugins/learned/{{topic-slug}}/skills
```

### 4.2 Create plugin.json

Write to `~/.claude/plugins/learned/{{topic-slug}}/plugin.json`:

```json
{
  "name": "{{topic-slug}}",
  "description": "Skills for working with {{topic}}",
  "generated": "{{YYYY-MM-DD}}",
  "lastUpdated": "{{YYYY-MM-DD}}",
  "topic": "{{topic}}",
  "skillCount": {{number of skills}}
}
```

### 4.3 Generate Each Skill

For each subtopic, create `~/.claude/plugins/learned/{{topic-slug}}/skills/{{subtopic-slug}}/SKILL.md`:

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

### 4.4 Update Mode Handling

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
  Location: ~/.claude/plugins/learned/{{topic-slug}}/

  Skills created:
  - {{subtopic-1}}: [brief description]
  - {{subtopic-2}}: [brief description]
  - ...

  Total: {{N}} skills

  These skills are now available in Claude Code. They will be automatically
  invoked when working on related tasks.
```

If UPDATE MODE, also report:
- Skills updated: N
- Skills added: N
- Skills unchanged: N
