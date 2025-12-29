# Second Opinion Plugin

Get unbiased architectural reviews from a fresh perspective using a Haiku subagent.

## Overview

This plugin provides independent architectural reviews to help make better design decisions. When facing architectural choices, the main Claude agent can invoke a fresh Haiku subagent that analyzes options from first principles without inheriting assumptions or biases.

## How It Works

**The Problem:**
- Confirmation bias (justifying initial approach)
- Over-engineering (adding unnecessary complexity)
- Blind spots (missing simpler alternatives)
- Unstated assumptions (constraints that don't actually exist)

**The Solution:**
Invoke a fresh Haiku subagent that:
- Analyzes from first principles
- Questions complexity naturally
- Challenges assumptions
- Provides independent perspective

## Components

### Skill: `architectural-decision-support`

Tells the main agent when and how to seek architectural reviews:
- High-stakes architectural decisions
- Multiple valid approaches exist
- Risk of over-engineering
- Uncertainty about trade-offs

### Subagent: `architectural-reviewer`

Haiku model that provides unbiased analysis:
- Fresh context, no inherited assumptions
- Optimized for questioning complexity
- Fast and cost-effective (~500-1000 tokens per review)
- Read-only tools (Read, Grep, Glob)

## Usage

The skill triggers automatically when the main agent faces architectural decisions. The workflow:

1. **Main agent generates 2-3 options** for an architectural decision
2. **Invokes architectural-reviewer** via Task tool with problem statement
3. **Reviewer analyzes independently** from first principles
4. **Main agent synthesizes** both perspectives
5. **Final decision** incorporates diverse viewpoints

## Example

```
User: "Design a caching system for our API"

Main agent:
- Option A: Redis (distributed, persistent)
- Option B: In-memory LRU (simple, fast)
- Option C: Multi-layer (complex, "best of both")

Reviewer (Haiku):
"Recommend Option B. Scale doesn't justify distributed cache.
Option C adds complexity without clear current benefit.
Question: Do you need cache persistence?"

Result: Simpler solution chosen, avoided over-engineering.
```

## Value Proposition

**Primary benefits:**
- Catch over-engineering early
- Surface simpler alternatives
- Validate or challenge architectural intuitions
- Force articulation of assumptions

**Cost:**
- ~500-1000 tokens per review (Haiku is cheap)
- 10-30 seconds added to decision process
- Minimal compared to cost of wrong architectural decision

**False positives are valuable:**
Even when reviewer is "wrong," explaining why the original approach is correct strengthens design documentation.

## When to Use

**Good candidates for review:**
- Designing new system components
- Choosing between architectural patterns
- Planning major refactoring
- Framework/library selection
- Decisions affecting >50% of system

**Skip review for:**
- Simple bug fixes
- Obvious implementation choices
- Time-critical changes
- Minor refactoring (<10% of system)

## Design Validation

This plugin's design was itself validated through meta-review:
1. First Haiku subagent critiqued the design
2. Second Haiku subagent critiqued the first critique
3. Both agreed on narrowing scope, disagreed on reasoning
4. The disagreement validated the core concept

See `docs/plans/2025-12-28-second-opinion-design.md` for full design rationale.

## License

MIT
