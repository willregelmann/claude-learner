# A/B Test: Does /learn Improve Task Output?

## Design
- 5 tasks across different domains
- Group A (Control): Agent executes task with no pre-loaded skill
- Group B (Treatment): Agent executes task with /learn-generated skill injected
- Both agents use the same model (sonnet), same tools, same task prompt
- Evaluator sees outputs with blinded labels, scores on rubric

## Tasks
1. Tailwind CSS v4 configuration with custom design system
2. React 19 useActionState with server actions
3. Drizzle ORM schema with relations and queries
4. Bun.$ shell API deployment script
5. Vitest tests for a Vue 3 composable

## Rubric (1-5 each, total 5-25)
- Correctness: Does the code work? Are APIs used correctly?
- Conventions: Does it follow idiomatic framework patterns?
- Gotcha Avoidance: Does it avoid known pitfalls and deprecations?
- Completeness: Edge cases, error handling, real-world considerations?
- Up-to-dateness: Uses current APIs, not deprecated ones?

## Randomization
See mapping-SEALED.json (created before task execution)
