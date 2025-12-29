# Second Opinion Plugin Design

**Date:** 2025-12-28
**Status:** Design Complete - Ready for Implementation

## Overview

A Claude Code plugin that provides independent architectural reviews through a Haiku subagent. Enables the main agent to get unbiased perspectives on architectural decisions without inheriting assumptions or biases.

## Problem Statement

When making architectural decisions, the main agent can fall into:
- Confirmation bias (justifying initial approach)
- Over-engineering (adding unnecessary complexity)
- Blind spots (missing simpler alternatives)
- Unstated assumptions (constraints that don't actually exist)

**Solution:** Invoke a fresh Haiku subagent that analyzes options from first principles and challenges assumptions.

## Core Insight

The value isn't Haiku being "smarter" than Opus/Sonnet - it's about **perspective diversity**:
- Fresh model approaches problem differently
- Simpler model questions complexity naturally
- Forces main agent to articulate assumptions
- False positives (questioning good designs) add value by forcing clarity

## Design Decisions

### Scope: Architectural Decision Support Only

**Focused on:** High-stakes architectural choices where multiple valid approaches exist

**NOT for:** Bug fixes, verification, code review, minor refactoring

**Why narrow scope:**
- Clear invocation triggers
- Natural context boundaries
- Human-in-the-loop decision support
- Measurable value (design improvement)

### Model Choice: Haiku

**Why Haiku:**
- Cheap enough to invoke liberally (~500-1000 tokens per review)
- Simpler perspective naturally questions over-engineering
- Fast response time
- Cost-efficiency enables frequent use

**Trade-off accepted:** Haiku may miss sophisticated issues, but that's not the use case. We're seeking fresh perspective, not deeper analysis.

### Architecture: Skill + Subagent

**Skill:** `architectural-decision-support`
- Tells main agent when to invoke reviewer
- Defines workflow (generate options, invoke reviewer, synthesize)
- Written in imperative form for agent consumption

**Subagent:** `architectural-reviewer`
- Haiku model for cost efficiency
- Read-only tools (Read, Grep, Glob)
- System prompt emphasizes first principles, simplicity, questioning assumptions

## Workflow

```
1. Main agent faces architectural decision
   ↓
2. Generate 2-3 distinct options
   ↓
3. Invoke architectural-reviewer subagent
   - Provide: problem statement, options, constraints
   - Withhold: implementation details (force fresh thinking)
   ↓
4. Reviewer analyzes independently
   - Evaluates each option
   - Questions assumptions
   - Suggests simplifications
   ↓
5. Main agent synthesizes both perspectives
   - Compare reasoning
   - Identify challenged assumptions
   - Articulate why final choice is correct
   ↓
6. Make decision (human or main agent)
   - Document reasoning
   - Include both perspectives
```

## Value Proposition

**Primary value:**
- Catch over-engineering early
- Surface simpler alternatives
- Force articulation of assumptions
- Validate or challenge architectural intuitions

**Secondary value (false positives):**
- Even when reviewer is "wrong," explaining why the original approach is correct strengthens design documentation

**Cost:**
- ~500-1000 tokens per review (Haiku)
- 10-30 seconds added to decision process
- Minimal compared to cost of wrong architectural decision

## Success Metrics

Track to validate value:

1. **Adoption rate** - How often is reviewer invoked?
2. **Incorporation rate** - % of reviews that lead to design changes
3. **False positive value** - Are "wrong" reviews still valuable for forcing articulation?
4. **Token cost** - Average tokens per review
5. **Quality improvement** - Does discussion improve final design?

## Components

### File Structure
```
second-opinion/
├── .claude-plugin/
│   └── plugin.json
├── skills/
│   └── architectural-decision-support/
│       └── SKILL.md
└── agents/
    └── architectural-reviewer.md
```

### plugin.json
```json
{
  "name": "second-opinion",
  "version": "1.0.0",
  "description": "Get unbiased architectural reviews from fresh perspective",
  "author": {
    "name": "Will"
  },
  "license": "MIT",
  "keywords": ["architecture", "review", "decision-support"]
}
```

## Skill Specification

**Name:** `architectural-decision-support`

**Description (third-person with trigger phrases):**
"This skill should be used when making architectural decisions, choosing between design approaches, planning system architecture, evaluating refactoring strategies, or facing non-trivial technical choices. Invoke architectural-reviewer subagent for independent analysis."

**Key Sections:**
1. When to Seek Independent Review (specific criteria)
2. How to Invoke (5-step workflow)
3. Value of Fresh Perspective (why this works)
4. Integration with Planning Workflows
5. Measuring Value (metrics to track)
6. Examples (concrete scenarios)
7. Common Pitfalls (what to avoid)
8. Success Criteria

**Word count target:** 1,500-2,000 words (lean, focused)

## Subagent Specification

**Name:** `architectural-reviewer`

**Model:** haiku (for cost efficiency)

**Color:** yellow (indicates caution/review mode)

**Tools:** Read, Grep, Glob (read-only, no modifications)

**System Prompt (simplified):**
```
You are an independent architectural reviewer providing fresh perspectives.

Analyze options from first principles without inheriting assumptions.

Core principles:
1. Question complexity - simpler is usually better
2. Think from first principles
3. Challenge assumptions
4. Be direct - state preference clearly
5. Consider trade-offs explicitly

Output: State recommendation, then explain reasoning.
```

**Description includes 3 examples** showing when to invoke with commentary.

## Design Validation Process

This design went through meta-review:

1. **First review (Haiku):** Questioned value proposition, recommended narrowing scope
2. **Second review (Haiku):** Challenged first review, defended value of fresh perspective
3. **Synthesis:** Both reviewers agreed narrowing scope is correct, disagreed on reasoning

**Key insight from meta-review:** The disagreement between reviewers validated the core concept - diverse perspectives reveal different insights.

## Implementation Plan

**Phase 1: Core Implementation**
1. Create plugin structure
2. Write skill (architectural-decision-support/SKILL.md)
3. Write subagent (architectural-reviewer.md)
4. Write plugin.json

**Phase 2: Testing**
1. Test invocation on real architectural decisions
2. Track metrics (adoption, incorporation, token cost)
3. Iterate based on usage patterns

**Phase 3: Documentation**
1. Update README with usage examples
2. Document success metrics
3. Add to claude-learner marketplace

## Integration with Claude Learner

This plugin complements claude-learner:
- claude-learner generates knowledge skills
- second-opinion provides decision support
- Both improve agent capabilities through different mechanisms

Could be distributed as:
- Separate plugin in same marketplace
- Additional component of claude-learner
- Standalone marketplace entry

## Open Questions

1. Should we support other review modes later (verification, first-principles) if metrics show value?
2. Should metrics tracking be built into the plugin or manual?
3. Should there be a user-facing command (`/review-architecture`) or skill-only?

## References

- Brainstorming session: 2025-12-28
- Meta-review validation: Two Haiku subagents with diverging perspectives
- Based on: superpowers plugin patterns, Claude Code best practices
