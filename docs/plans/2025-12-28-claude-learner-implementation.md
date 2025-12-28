# Claude Learner Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a Claude Code plugin with a `/learn <topic>` command that researches any topic and generates Claude-optimized skills.

**Architecture:** A pure markdown/prompt plugin with one slash command (`/learn`) that orchestrates web research and file generation. Generated skills are stored as mini-plugins in `~/.claude/plugins/learned/`.

**Tech Stack:** Claude Code plugin system (markdown commands, YAML frontmatter), WebSearch tool, Write tool.

---

## Task 1: Create Plugin Manifest

**Files:**
- Create: `plugin.json`

**Step 1: Create the plugin.json file**

```json
{
  "name": "claude-learner",
  "description": "Research any topic and generate Claude-optimized skills",
  "version": "1.0.0",
  "author": {
    "name": "Will"
  },
  "repository": "https://github.com/willcodefortea/claude-learner",
  "license": "MIT",
  "keywords": ["learning", "research", "skills", "knowledge"]
}
```

**Step 2: Verify file exists**

Run: `cat plugin.json`
Expected: The JSON content above

**Step 3: Commit**

```bash
git add plugin.json
git commit -m "feat: add plugin manifest"
```

---

## Task 2: Create Commands Directory

**Files:**
- Create: `commands/` directory

**Step 1: Create the commands directory**

```bash
mkdir -p commands
```

**Step 2: Verify directory exists**

Run: `ls -la commands/`
Expected: Empty directory listing

**Step 3: Commit**

```bash
git add .
git commit -m "feat: add commands directory structure"
```

---

## Task 3: Create the /learn Command

**Files:**
- Create: `commands/learn.md`

**Step 1: Create the learn command file**

The command needs to:
1. Parse the topic argument
2. Check if topic already exists (update mode)
3. Execute 4-phase research process
4. Generate skill files
5. Report what was created/updated

```markdown
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
```

**Step 2: Verify file exists and has correct structure**

Run: `head -20 commands/learn.md`
Expected: YAML frontmatter with name, description, and arguments

**Step 3: Commit**

```bash
git add commands/learn.md
git commit -m "feat: add /learn command for topic research and skill generation"
```

---

## Task 4: Create Skills Directory

**Files:**
- Create: `skills/` directory

**Step 1: Create the skills directory**

```bash
mkdir -p skills/using-learner
```

**Step 2: Verify directory exists**

Run: `ls -la skills/`
Expected: Directory with using-learner subdirectory

**Step 3: Commit**

```bash
git add .
git commit -m "feat: add skills directory structure"
```

---

## Task 5: Create the Using-Learner Meta-Skill

**Files:**
- Create: `skills/using-learner/SKILL.md`

**Step 1: Create the meta-skill file**

```markdown
---
name: using-learner
description: Use when someone asks about the /learn command, how to generate skills, or how to use the claude-learner plugin
---

# Using Claude Learner

## Overview

The `/learn` command researches any topic and generates Claude-optimized skills that help you work more effectively in that domain.

## Usage

```
/learn <topic>
```

**Examples:**
- `/learn laravel 12` - Generate skills for Laravel 12 framework
- `/learn rocket physics` - Generate skills for rocket physics concepts
- `/learn kubernetes networking` - Generate skills for K8s networking

## What Gets Generated

For each topic, the command creates a mini-plugin at `~/.claude/plugins/learned/<topic>/` containing:

- `plugin.json` - Metadata about the generated skills
- `skills/` - Directory of 4-8 focused skills

Each skill contains:
- When to apply the skill (triggering conditions)
- Key patterns and best practices
- Common mistakes to avoid
- Concrete examples

## Re-running for Updates

Running `/learn <topic>` again on an existing topic will:
- Research for new information
- Update existing skills with new findings
- Add skills for newly discovered subtopics
- Preserve any custom edits you've made

## Tips for Effective Learning

1. **Be specific**: `/learn laravel 12` is better than `/learn laravel`
2. **Domain agnostic**: Works for any topic, not just programming
3. **Wait for completion**: Research takes time, let it finish
4. **Check generated skills**: Review and customize as needed

## Troubleshooting

**Skills not appearing?**
- Check `~/.claude/plugins/learned/` for the generated directory
- Restart Claude Code to pick up new plugins

**Want to regenerate from scratch?**
- Delete `~/.claude/plugins/learned/<topic>/`
- Run `/learn <topic>` again
```

**Step 2: Verify file exists**

Run: `cat skills/using-learner/SKILL.md`
Expected: The skill content above

**Step 3: Commit**

```bash
git add skills/using-learner/SKILL.md
git commit -m "feat: add using-learner meta-skill"
```

---

## Task 6: Create README

**Files:**
- Create: `README.md`

**Step 1: Create README file**

```markdown
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
```

**Step 2: Verify file exists**

Run: `head -20 README.md`
Expected: README content with installation instructions

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README with installation and usage instructions"
```

---

## Task 7: Manual Verification

**Files:**
- None (verification only)

**Step 1: Verify plugin structure**

Run: `find . -type f | grep -v ".git" | sort`

Expected output:
```
./commands/learn.md
./plugin.json
./README.md
./skills/using-learner/SKILL.md
```

**Step 2: Verify plugin.json is valid JSON**

Run: `python3 -c "import json; json.load(open('plugin.json'))"`
Expected: No output (success)

**Step 3: Verify command frontmatter**

Run: `head -10 commands/learn.md`
Expected: Valid YAML frontmatter with name, description, arguments

**Step 4: Verify skill frontmatter**

Run: `head -5 skills/using-learner/SKILL.md`
Expected: Valid YAML frontmatter with name and description

**Step 5: Final commit with verification**

```bash
git log --oneline
```

Expected: 5-6 commits showing incremental progress

---

## Task 8: Merge to Main

**Files:**
- None (git operations only)

**Step 1: Switch to main branch in main worktree**

```bash
cd /home/will/Projects/claude-learning
git checkout main
```

**Step 2: Merge feature branch**

```bash
git merge feature/implement-learner
```

**Step 3: Verify merge**

Run: `ls -la`
Expected: All plugin files present in main worktree

**Step 4: Clean up worktree**

```bash
git worktree remove .worktrees/implement-learner
git branch -d feature/implement-learner
```

---

## Summary

After completing all tasks, you will have:

1. A complete Claude Code plugin structure
2. A `/learn` command that orchestrates research and skill generation
3. A meta-skill documenting how to use the plugin
4. A README for GitHub distribution
5. Clean git history with incremental commits
