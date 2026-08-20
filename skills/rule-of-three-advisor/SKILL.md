---
name: rule-of-three-advisor
description: "Finds likely Rule of Three refactoring opportunities, provides concrete code evidence, and suggests proportionate designs. Use when reviewing duplicated logic or deciding whether repeated code should become a shared abstraction."
user-invocable: true
---

# Rule of Three Advisor

Review code for the Rule of Three: when substantially similar logic appears in three or more places, treat it as evidence that a shared abstraction may improve maintainability. This is a heuristic, not an automatic refactoring mandate. Reference: [Rule of Three (computer_programming)](<https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)>).

## Operating Rules

- Search the relevant repository scope before forming a finding. In monorepos, identify the owning package or service and avoid unrelated workspace areas. Focus on executable or operational artifacts that affect system design and maintenance, including source code, configuration, scripts, tests, and deployment definitions. Respect `.gitignore` and equivalent ignore rules: do not inspect ignored files or directories unless the user explicitly includes them. Exclude generated, vendored, build-output, fixture, documentation-only, and agent customization content, including `SKILL.md`, `AGENT.md`, prompt, instruction, and similar agent-skill files. Follow local contribution, architecture, and testing conventions.
- Distinguish duplicated behavior from coincidental similarity. Compare intent, inputs, outputs, invariants, error handling, side effects, lifecycle, and expected rate of change.
- Require at least three concrete occurrences for a Rule of Three finding unless the user explicitly asks for two-instance duplication or a known design smell.
- Prefer evidence over assertion: provide file paths, line ranges or symbols, and a short description of the shared behavior. Quote only the smallest useful snippets.
- Do not recommend abstraction solely to reduce line count. Call out cases where duplication is intentional, clearer, independently evolving, generated, or separated by a meaningful boundary.
- Use `git log` or `git blame` when claiming change-frequency, bug-fix drift, or ownership evidence; otherwise label those factors unknown.
- Preserve behavior. Suggestions must identify how callers, tests, error semantics, and ownership would remain correct.
- Do not edit code unless the user explicitly requests implementation. When implementation is requested, propose the smallest coherent change and add or update focused tests when repository patterns support them.
- Never claim certainty from text matching alone. Label findings as confirmed, likely, or inconclusive and explain why.
- If no qualifying duplication exists, say so explicitly. Report the closest rejected candidates and why they do not meet the threshold when that context is useful.

## Workflow

1. **Establish scope.** Identify files, language, runtime, owning package or service, and review target. Apply repository ignore rules before searching. Include executable or operational artifacts such as source code, configuration, scripts, tests, and deployment definitions. Exclude ignored paths, generated, vendored, build-output, fixture, documentation-only, and agent customization files such as `SKILL.md` and `AGENT.md`, unless the user explicitly asks to include them.
2. **Collect candidates.** Search for repeated blocks, equivalent branches, repeated validation/serialization/error-handling flows, and near-duplicate helpers. Group occurrences by behavior, not merely names.
3. **Validate candidates.** Inspect each occurrence in context. Confirm the shared contract and list meaningful differences. Reject groups whose only commonality is superficial syntax.
4. **Assess the threshold.** Prioritize groups with three or more occurrences, high change frequency, bug-fix drift, or difficult consistency requirements. Treat the third meaningful occurrence as the normal refactoring trigger; do not wait for a fourth occurrence unless the evidence shows the third is intentionally independent.
5. **Report evidence.** For each finding, include:
   - **Finding:** concise description and confidence.
   - **Evidence:** three or more paths plus line ranges/symbols, and the behavior shared.
   - **Risk:** concrete maintenance or correctness cost, if present.
   - **Suggestion:** preferred abstraction boundary and alternatives, with tradeoffs.
   - **Validation:** for review-only analysis, characterization tests or focused checks that would confirm shared behavior; for implementation, the focused tests or checks to run before and after refactoring.
6. **Recommend proportionately.** Prefer, in order: a small local helper, a domain-level function/service, or a carefully designed object/module. Keep call sites readable and avoid abstractions that couple unrelated domains.
7. **Close uncertainty.** If line-level evidence or behavior cannot be established, report the missing evidence and stop short of a definitive recommendation.

## Report Template

```md
### <finding> — <confirmed|likely|inconclusive>

Evidence:

- `<path>:<lines or symbol>` — <shared behavior>
- `<path>:<lines or symbol>` — <shared behavior>
- `<path>:<lines or symbol>` — <shared behavior>

Risk: <maintenance or correctness impact, or none demonstrated>
Suggestion: <smallest suitable abstraction>; tradeoff: <key tradeoff>
Validation: <focused tests/checks>
```

## Escalate or Defer

- Similar code crosses a public API, package, service, or team ownership boundary.
- Extraction would alter error handling, transactionality, performance, security behavior, or public contracts.
- Fewer than three occurrences exist and no explicit exception applies.
- Generated code, framework conventions, or independent release cadence makes consolidation unsafe.
