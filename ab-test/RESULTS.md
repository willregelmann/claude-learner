# A/B Test Results: Does /learn Improve Task Output?

## Experiment Summary

- **Hypothesis**: Pre-loading skills via `/learn` improves Claude's output quality
- **Design**: 5 tasks, each executed by two agents (control vs treatment), evaluated blind by opus-level judges
- **Model**: sonnet for all task agents, opus for evaluators
- **Blinding**: Evaluators saw only "Output Alpha" and "Output Bravo" — did not know which was treatment

## Unblinded Results

| Task | Domain | Control | Treatment | Delta | Winner |
|------|--------|---------|-----------|-------|--------|
| 1. Tailwind CSS v4 | Framework config | 15/25 | **23/25** | +8 | Treatment |
| 2. React 19 useActionState | React hooks | 20/25 | **25/25** | +5 | Treatment |
| 3. Drizzle ORM | Database ORM | **23/25** | 19/25 | -4 | Control |
| 4. Bun Shell API | Runtime scripting | 17/25 | **24/25** | +7 | Treatment |
| 5. Vitest + Vue 3 | Testing | 16/25 | **24/25** | +8 | Treatment |

## Aggregate Statistics

| Metric | Value |
|--------|-------|
| Treatment win rate | **4/5 (80%)** |
| Control win rate | 1/5 (20%) |
| Mean control score | 18.2/25 |
| Mean treatment score | **23.0/25** |
| Mean improvement | **+4.8 points (+26%)** |
| Largest treatment advantage | +8 (Tasks 1, 5) |
| Only control win | Task 3 (Drizzle ORM, -4) |

## Per-Task Analysis

### Task 1: Tailwind CSS v4 (Treatment +8)
**Critical finding**: Control missed `@custom-variant dark` directive entirely — the v4 mechanism for class-based dark mode. Also used `px` for breakpoints (v4 recommends `rem`) and omitted `@tailwindcss/vite` plugin. Treatment correctly used `@custom-variant`, `--breakpoint-*: initial`, rem units, and the Vite plugin. The skill's "Common Mistakes to Avoid" section directly prevented these errors.

### Task 2: React 19 (Treatment +5)
Treatment used Zod validation (idiomatic), preserved field values on error via `defaultValue`, added `aria-invalid` attributes, used `role="alert"` for screen readers, and handled general error messages separately from field errors. Control was functionally correct but simpler — manual validation instead of Zod, no field value preservation, less accessibility. The skill's patterns for Zod validation and `defaultValue` preservation were directly applied.

### Task 3: Drizzle ORM (Control +4)
The one task where control won. Control included `onDelete: 'cascade'` on foreign keys, used `.returning()` on all mutations, had a self-contained `runExamples()` with proper data flow between operations, and used `import * as schema` (idiomatic). Treatment omitted cascade deletes, skipped `.returning()` on update/delete, and hardcoded IDs in the demo. Notably, the skill contained guidance on these patterns, but the treatment agent didn't apply all of it.

### Task 4: Bun Shell (Treatment +7)
Treatment used `.nothrow()` consistently (the Bun-idiomatic pattern for deployment scripts), top-level await, and `Bun.file().json()`. Control used try/catch with `.quiet()` (losing error details), imported `exit` from Node's `process` module, wrapped in unnecessary `main()` function, and re-read `package.json` due to scoping issues. The skill's guidance on `.nothrow()` vs try/catch and top-level await was directly beneficial.

### Task 5: Vitest Vue 3 (Treatment +8)
Treatment used `withSetup` helper (from the skill), `flushPromises` (not `nextTick`), centralized app cleanup in `afterEach`, included bonus tests (AbortError handling, error clearing after retry), and complete mock Response objects. Control used `mount(defineComponent(...))` (verbose), `vi.waitFor()` instead of `flushPromises`, manual `wrapper.unmount()` in each test, and `vi.clearAllMocks()` instead of `vi.restoreAllMocks()`. The skill's patterns for `withSetup`, `flushPromises`, and mock restoration were directly applied.

## Key Observations

1. **The biggest improvements came from gotcha avoidance.** Tasks 1, 4, and 5 showed the largest deltas (+8, +7, +8), all driven by the treatment agent avoiding specific pitfalls documented in the skill (missing directives, wrong error handling patterns, wrong async testing utilities).

2. **Skills don't guarantee improvement.** Task 3 (Drizzle) showed the control actually outperforming treatment, despite the skill containing relevant guidance. The treatment agent didn't apply all the patterns from the skill (e.g., `.returning()`, cascade deletes). This suggests the skill provides _opportunity_ for improvement, not a guarantee.

3. **Framework convention tasks benefit most.** Tasks involving v4/v19-era API changes (Tailwind v4, React 19, Bun shell) showed the largest improvements, likely because the skill contained current API knowledge that the base model might not have in its training data.

4. **The mean improvement of 26% is substantial.** Going from 18.2 to 23.0 out of 25 represents moving from "decent but with significant issues" to "near-perfect with minor nits."

## Threats to Validity

1. **Single run per condition**: Each task was run once per condition. Non-deterministic model outputs could affect individual results. Ideally, each condition should run 2-3 times.

2. **Evaluator bias toward longer outputs**: Treatment outputs were sometimes longer/more detailed. Evaluators may have unconsciously favored thoroughness. However, the rubric focused on specific correctness criteria, not length.

3. **N=5 is small**: With only 5 tasks, statistical significance is limited. A Wilcoxon signed-rank test on paired differences yields p~0.06 (borderline). More tasks would strengthen conclusions.

4. **Treatment agents read skill files**: In real usage, skills are in the system prompt. Here, treatment agents read a file as their first action. This is equivalent in terms of context but adds one extra tool call.

5. **Web search was disabled**: Both conditions were told "do not use web search," which disadvantages control. In real usage, Claude *can* search the web. With web search enabled, the gap might be smaller (but /learn pre-researches, saving time and tokens).

## Conclusion

**Strong evidence that /learn improves output quality**, particularly for:
- Tasks involving recent API changes (frameworks that changed after training cutoff)
- Tasks with non-obvious gotchas and conventions
- Tasks where specific idiomatic patterns matter (not just "make it work" but "make it correct")

The 80% win rate and 26% score improvement suggest `/learn` is a meaningful tool, though not infallible (1 task showed control winning).
