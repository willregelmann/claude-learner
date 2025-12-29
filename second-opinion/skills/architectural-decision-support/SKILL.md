---
name: architectural-decision-support
description: This skill should be used when the user asks to "make an architectural decision", "choose between design approaches", "evaluate architecture options", "plan system architecture", "review refactoring strategy", or when facing non-trivial technical choices with multiple valid approaches. Invoke architectural-reviewer subagent for independent analysis.
version: 1.0.0
---

# Architectural Decision Support

## When to Seek Independent Review

Invoke the architectural-reviewer subagent when facing high-stakes architectural decisions:

**High-stakes architectural decisions:**
- Designing new system components or services
- Choosing between architectural patterns (monolith vs microservices, event-driven vs request-response, etc.)
- Planning major refactoring strategies
- Evaluating framework or library choices
- Making decisions that affect more than 50% of a system
- Selecting data storage or caching strategies

**Indicators that review would be valuable:**
- Multiple valid approaches exist with different trade-offs
- Decision has long-term implications (difficult to reverse later)
- Uncertain about trade-offs between options
- Approaching problem with strong assumptions that should be challenged
- Risk of over-engineering or premature optimization
- User explicitly requests second opinion or review

**When NOT to invoke:**
- Simple bug fixes or obvious corrections
- Obvious implementation choices with one clear best practice
- Time-critical changes where delay is costly
- Already-shipping production code (unless major refactoring planned)
- Minor refactoring affecting less than 10% of system
- Trivial configuration changes

## How to Invoke the Architectural Reviewer

Follow this five-step workflow:

### Step 1: Generate Options

Before invoking the reviewer, generate 2-3 distinct architectural approaches:
- Ensure options are meaningfully different (not just minor variations)
- Document key characteristics of each option
- Identify trade-offs already visible

**Example:**
```
Option A: Monolithic architecture with modular structure
Option B: Microservices with API gateway
Option C: Event-driven architecture with message bus
```

### Step 2: Prepare Context

Provide the reviewer with focused context:
- **Problem statement**: Clear description of what needs to be solved
- **Inputs and outputs**: What goes in, what comes out
- **Constraints**: Scale requirements, team size, existing systems, time constraints
- **The 2-3 options**: Brief description of each approach
- **Key trade-offs**: What you've already identified

**Critical**: Do NOT provide implementation details. Force fresh thinking by keeping it abstract.

**Good context:**
```
Problem: Cache API responses for 1000 req/sec system
Inputs: HTTP requests with user context
Outputs: JSON responses
Constraints: Need TTL support, key pattern invalidation
Options: Redis, in-memory LRU, multi-layer cache
```

**Bad context:**
```
Here's our existing 500-line cache implementation with Redis...
[includes detailed code and configuration]
```

### Step 3: Invoke Subagent

Use the Task tool to invoke the architectural-reviewer:

```
Task tool parameters:
  subagent_type: architectural-reviewer
  model: haiku
  description: "Review architecture options"

  prompt: "Evaluate these architectural options independently:

  Problem: [clear problem statement]
  Inputs: [what goes in]
  Outputs: [what comes out]
  Constraints: [scale, requirements, limitations]

  Option A: [approach description with key characteristics]
  Option B: [approach description with key characteristics]
  Option C: [approach description with key characteristics]

  Analyze from first principles. Which would you choose and why?
  What simpler alternatives might work?
  What assumptions should be challenged?"
```

### Step 4: Synthesize Feedback

Compare the reviewer's perspective with initial analysis:

**Key questions:**
- Where do perspectives align? (validation of approach)
- Where do perspectives diverge? (reveals assumptions or blind spots)
- What assumptions did the reviewer challenge? (may be invalid constraints)
- Did reviewer identify simpler approaches? (potential over-engineering)
- Are there new trade-offs to consider?

**If reviewer questions a design you think is correct:**
- Articulate WHY the original approach is right
- Document the reasoning (this is valuable even if reviewer is "wrong")
- Consider if the concern reveals edge cases or future issues

**If reviewer suggests simplifications:**
- Evaluate whether suggested simpler approach actually meets requirements
- Question whether added complexity in original design is necessary
- Document why complexity is justified (or simplify if not)

### Step 5: Make Decision

Use both perspectives to inform the final choice:

**Document the decision:**
- Which option was selected
- Key reasoning from both perspectives
- Assumptions that were challenged
- Why this choice over alternatives
- What was learned from the review process

**Communicate to user:**
- Present final recommendation with clear reasoning
- Mention that independent review was conducted
- Highlight any significant insights from review process
- Note areas of agreement and disagreement between perspectives

## Value of Fresh Perspective

The architectural-reviewer provides value through several mechanisms:

### Different Reasoning Paths

Even simpler models approach problems differently, which reveals:
- **Over-engineered solutions**: Reviewer asks "why is this complexity needed?"
- **Unstated assumptions**: Reviewer doesn't know about constraints you assume
- **Simpler alternatives**: Fresh perspective sees obvious solutions you overlooked
- **Edge cases from different angles**: Different reasoning catches different issues

### Forced Articulation

Explaining approach to the reviewer forces clarity:
- "Why am I choosing X over Y?" must be articulated explicitly
- "What assumptions am I making?" become visible when challenged
- "Is this complexity necessary?" must be justified with reasoning

This process improves design documentation even when reviewer agrees with original approach.

### False Positives Are Valuable

If reviewer questions a good design, articulating "why this is correct" has value:
- Strengthens confidence in the decision
- Creates better documentation for future maintainers
- Reveals edge cases that should be documented
- Tests whether reasoning is sound

The cost of a false positive (defending a good design) is low compared to the cost of missing a real issue.

## Integration with Planning Workflows

### Before Writing Detailed Plans

Use architectural-reviewer to validate approach before creating detailed implementation plans:
- Invoke reviewer during design phase
- Incorporate feedback before using writing-plans skill
- Document both perspectives in design doc
- Proceed with detailed planning once architecture validated

### During Brainstorming

Generate options during brainstorming phase:
- Use brainstorming skill to explore alternatives
- Once 2-3 solid options identified, invoke reviewer
- Let human make final call with both perspectives informing decision

### Before Major Refactoring

Present current architecture and proposed changes:
- Describe existing system (inputs, outputs, current approach)
- Present proposed refactoring
- Ask reviewer: "Given these requirements, would you design it this way?"
- Compare fresh design with existing/proposed to validate refactoring addresses root issues

## Measuring Value

Track these metrics to validate the reviewer is providing value:

### Usage Patterns
- How often is reviewer feedback incorporated into final decision?
- What percentage of reviews lead to design changes?
- Are false positives (questioning good designs) valuable for forcing articulation?

### Quality Indicators
- Did review catch over-engineering?
- Did it surface simpler alternatives?
- Did discussion improve final design?
- Were unstated assumptions revealed?

### Cost-Benefit Analysis
- Token cost per review (expect ~500-1000 tokens with Haiku)
- Time added to decision process (typically 10-30 seconds)
- Value: Better architectural decisions, avoided over-engineering, improved documentation

If metrics show low incorporation rate (<20%) or high token cost without clear value, reconsider usage patterns.

## Examples

### Example 1: Caching Layer Design

**Main agent generates options:**
```
Option A: Redis with write-through caching
  - Distributed cache, survives restarts
  - Adds Redis dependency and operational complexity

Option B: In-memory LRU with TTL
  - Simple, no external dependencies
  - Lost on restart, not shared across instances

Option C: Multi-layer (memory + Redis)
  - Best performance + persistence
  - Maximum complexity
```

**Invoke reviewer:**
```
"Given: 1000 req/sec, JSON responses, need TTL + invalidation
Scale: Single server initially, may scale to 2-3 servers later
Question: Which caching approach would you choose?"
```

**Reviewer response (Haiku):**
```
"Recommendation: Option B (in-memory LRU)

Reasoning:
- 1000 req/sec on single server doesn't need distributed cache
- 'May scale to 2-3 servers' is speculative - YAGNI
- Option C adds complexity without clear current benefit
- Consider: Do you actually need cache persistence?

If you need distributed caching later, migrate then.
Start simple."
```

**Synthesis:**
- Reviewer caught potential over-engineering (Option C)
- Challenged assumption that future scaling requires distributed cache now
- Forced question: "Do we need persistence for this use case?"
- **Decision**: Start with Option B, add Redis only if scaling proves necessary
- **Value**: Avoided premature complexity

### Example 2: API Authentication

**Main agent generates options:**
```
Option A: JWT with refresh tokens
  - Stateless, scalable
  - Complex token rotation logic

Option B: Session-based with Redis
  - Simple, well-understood
  - Requires Redis dependency

Option C: OAuth 2.0 with external provider
  - Offloads auth complexity
  - External dependency, vendor lock-in
```

**Invoke reviewer:**
```
"Internal API, 100 users, need stateless auth for mobile app
Current stack: Node.js, PostgreSQL
Question: Which authentication approach?"
```

**Reviewer response:**
```
"Recommendation: Question the 'must be stateless' constraint

You say 'internal API, 100 users' but require stateless for mobile.
If you control both client and server, why stateless?

Option B (sessions) is simpler for this scale:
- No token rotation complexity
- Redis is lightweight
- Easier to revoke sessions

Challenge: Why is stateless a requirement?"
```

**Synthesis:**
- Reviewer challenged unstated assumption (must be stateless)
- Forced re-evaluation: Stateless chosen for "scalability" but 100 users doesn't need it
- Realized sessions actually work better for use case
- **Decision**: Option B with documented reasoning
- **Value**: Avoided complexity driven by premature optimization

### Example 3: Microservices Decision

**Main agent generates options:**
```
Option A: Keep monolith, improve modularity
  - Simple deployment, easier development
  - May become bottleneck if one module scales differently

Option B: Split into microservices now
  - Independent scaling, team autonomy
  - Operational complexity, network overhead
```

**Invoke reviewer:**
```
"E-commerce platform, 5 developers, current traffic manageable
Modules: auth, product catalog, orders, payments
Question: Microservices now or later?"
```

**Reviewer response:**
```
"Recommendation: Option A (monolith) with caveat

5 developers is too small for microservice overhead.
Network calls between services add latency you don't need yet.

But: Make modules truly decoupled INSIDE the monolith.
Then microservices extraction is easy if needed.

Red flag: 'May become bottleneck' is speculative.
When it becomes a problem, solve it then."
```

**Synthesis:**
- Reviewer validated monolith approach for team size
- Added insight: Focus on modular monolith design
- **Decision**: Option A with explicit module boundaries
- **Value**: Avoided premature distribution while maintaining future flexibility

## Common Pitfalls

### Pitfall 1: Providing Too Much Context

Don't give reviewer implementation details that bias analysis.

**Problem:**
```
"Here's our existing 500-line cache implementation with Redis.
Should we refactor it to use memcached instead?"
```

**Better:**
```
"Cache 1000 req/sec with TTL support.
Currently using Redis. Alternative: memcached.
Key difference: Redis has more features we're not using."
```

Keep it abstract. Force fresh thinking.

### Pitfall 2: Ignoring False Positives

If reviewer questions a design you think is correct, don't dismiss it immediately.

**Anti-pattern:**
"Reviewer suggested simpler approach but doesn't understand our constraints. Ignoring."

**Better approach:**
"Reviewer suggested simpler approach. Let me articulate WHY our constraints require the more complex solution. This documentation will help future maintainers."

Even wrong reviews have value in forcing explicit reasoning.

### Pitfall 3: Asking Too Late

Invoke reviewer BEFORE detailed implementation planning.

**Too late:**
```
After writing 1000 lines of code:
"Should we have used a different architecture?"
```

**Right time:**
```
During design phase:
"We're choosing between architectures A, B, C. Independent review?"
```

Architectural decisions are expensive to change. Get review early.

### Pitfall 4: Treating Reviewer as Oracle

Reviewer provides perspective, not absolute truth.

**Anti-pattern:**
"Reviewer said Option B, so we must choose Option B."

**Better:**
"Reviewer prefers Option B because of X.
I prefer Option A because of Y.
Considering both perspectives, I choose Option A because [documented reasoning]."

Synthesize both views. Human (or main agent) makes final decision.

### Pitfall 5: Not Documenting Reasoning

Capture insights from the review process.

**Anti-pattern:**
Get review, make decision, move on without documenting.

**Better:**
Document:
- What reviewer suggested
- Where perspectives aligned/diverged
- Why final choice was made
- What assumptions were challenged

This creates valuable design documentation.

## Success Criteria

A review is successful when it achieves at least one of:

**Primary success criteria:**
- Surfaces simpler alternatives that meet requirements
- Challenges unstated or invalid assumptions
- Forces clear articulation of trade-offs
- Catches over-engineering before implementation
- Improves final design decision

**Secondary success criteria (false positives):**
- Questions a good design, forcing articulation of why it's correct
- Reveals edge cases that should be documented
- Strengthens confidence through validation

**Review is NOT required to:**
- Always find issues (validation is valuable even with no issues found)
- Agree with main agent (disagreement reveals assumptions)
- Provide "correct" answer (often no single right answer exists)
- Make the decision (human or main agent decides, reviewer informs)

## Integration Example

Complete workflow showing integration with planning:

```
1. User: "Design caching system for our API"

2. Main agent: [brainstorms 3 approaches]

3. Main agent: [invokes architectural-reviewer via Task tool]
   - Provides problem statement, options, constraints
   - NO implementation details

4. Reviewer (Haiku): [analyzes independently]
   - Questions complexity assumptions
   - Suggests simpler alternative
   - Challenges "need distributed cache" assumption

5. Main agent: [synthesizes perspectives]
   - Realizes over-engineering
   - Decides on simpler approach
   - Documents reasoning from both perspectives

6. Main agent → user: "Based on independent review,
   recommending in-memory cache. Here's why:
   [reasoning incorporating both perspectives]"

7. If user approves: Proceed with detailed implementation plan
```

This workflow ensures architectural decisions benefit from diverse perspectives before committing to detailed implementation.
