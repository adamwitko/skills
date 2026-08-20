---
name: production-change-readiness
description: "Assess whether a repository change is ready for production by checking scope, blast radius, compatibility, validation, observability, rollout, rollback, ownership, and evidence."
user-invocable: true
---

# Production Change Readiness

**TL;DR**: Check a change's scope, risk, compatibility, validation, and rollback plan before it ships, then return a Ready / Ready with conditions / Not ready / Blocked decision backed by evidence — never a guess.

Use repository instructions, architecture, deployment model, risk policy, and operational conventions as the source of truth. **This assesses readiness; it does not authorize deployment.**

## Use When

- A PR, commit, or deployment is being evaluated for release to production.
- Someone asks "is this ready to ship/deploy/merge to prod" or wants a go/no-go call.
- A risky change (migration, auth, public API, infra, permissions) needs a structured readiness check before approval.

Don't use for:

- Routine dependency bumps or docs-only changes with no production impact.
- Test-only changes that purely add coverage with no production code touched.
- Style/lint-only diffs or a general code-quality review — use a code-review skill instead.
- Deciding _how_ to implement a feature. This skill only assesses release readiness of a completed or proposed change.

Do use this skill when a test-only change removes, skips, or reduces coverage of existing tests — that erodes confidence and carries its own release risk.

## Operating Rules

- Establish the full boundary: code, configuration, data, dependencies, infrastructure, CI/CD, generated artifacts, documentation, and external contracts.
- Separate evidence, policy, assumptions, missing evidence, and recommendations.
- Do not deploy, migrate production data, modify infrastructure, rotate credentials, or disable safeguards without explicit authorization and scope.
- Follow existing security, dependency, testing, approval, and compliance workflows. Do not duplicate or weaken them.
- Treat irreversible migrations, destructive data operations, permission changes, public contracts, and untested rollback as elevated risk.
- A passing build is insufficient. Check behavior, compatibility, operations, observability, and recovery.
- Do not expose secrets, personal data, tokens, environment dumps, or full logs. Use concise results and local evidence paths.
- Missing required evidence means **unknown** or **blocked**, never approved.

## Readiness Decision

- **Ready**: required gates passed, release conditions are authorized, and residual risks have owners.
- **Ready with conditions**: listed conditions must be met or explicitly accepted by an authorized owner.
- **Not ready**: a required gate failed, a blocking risk lacks mitigation, or recovery is inadequate.
- **Blocked**: scope, authority, environment, policy, or required evidence is unavailable.

Never mark a change **Ready** while a required gate is unknown.

## Workflow

### 1. Establish Scope and Authority (read-only)

Identify:

- Purpose, affected users and systems, environments, release mechanism
- Changed resources, data classification, trust boundaries, public contracts, downstream consumers
- Release window, sequencing constraints, owners, approvers, explicit authorization

Read relevant repository artifacts: instructions, issues, RFCs, ADRs, incidents, runbooks, ownership files, deployment definitions, and history. If intended behavior, target environment, owner, or authority is unclear, stop and ask. Do not broaden scope silently.

### 2. Determine Risk and Blast Radius

Use repository policy when available. Otherwise classify:

- **Low**: isolated behavior, bounded impact, no data or contract change, straightforward rollback.
- **Medium**: shared or user-visible behavior, dependency/runtime change, operational configuration, or one service boundary.
- **High**: authentication, authorization, sensitive data, billing, destructive or irreversible migration, public contract, cross-service coordination, critical path, infrastructure access, or broad deployment.

Record:

- Affected components and failure modes
- Maximum credible impact, detection time, recovery time
- Release dependencies

Use `git log` or `git blame` for claims about ownership, change frequency, or historical failures.

### 3. Check Compatibility and Safety

Inspect the smallest relevant code paths, tests, configuration, and deployment definitions. Check:

- **Compatibility**: mixed versions across schemas, data, serialization, APIs, events, CLIs, configuration
- **Platform support**: dependencies, runtime, OS, region, infrastructure
- **Security**: authentication, authorization, privacy, secrets, security controls
- **Resilience**: concurrency, retries, idempotency, timeouts, limits, failure semantics
- **Migration safety**: order, locks, capacity, backfill, validation, cleanup timing
- **Progressive delivery**: feature flags, defaults, cohort targeting, dark launch, emergency disablement

Use existing security, dependency, privacy, database, or compliance workflows for specialized checks. Report their result and scope; do not treat this skill as a replacement.

### 4. Verify Validation Gates

Find canonical repository commands and run only authorized, relevant checks. Use comparable baseline and proposed gates when useful. Select from:

- **Code quality**: format, lint, typecheck, build
- **Tests**: unit, integration, contract, end-to-end, smoke, load
- **Data/migration**: dry runs, rollback or restore rehearsals, consistency checks, configuration checks
- **Security/supply chain**: security, dependency, license, secret, container, infrastructure checks
- **Pre-release**: preview, canary, staging, production-like verification

Record command, scope, result, duration, and artifact path. Separate pre-existing, environmental, flaky, and change-related failures. Do not repair unrelated failures.

### 5. Review Rollout and Recovery

Require a risk-appropriate procedure covering:

- **Deployment**: order, pause points, canary/staged/flagged rollout, communication
- **Monitoring**: success and guardrail metrics, alert thresholds, observation window, decision owner
- **Rollback**: trigger, exact action, duration, operator
- **Recovery**: data recovery or forward fix when rollback cannot undo a migration or side effect

A version revert is not automatically a rollback. Check data, queues, caches, external calls, configuration, and side effects after code reversal. Label rollback as **verified**, **simulated**, **documented-only**, or **unknown**; do not call inspection a test.

### 6. Close With Decision and Evidence

Report:

- Decision, scope, risk, blast radius
- Gate results (passed/failed/unknown/blocked)
- Rollout and recovery requirements
- Residual risks with owners, conditions, next actions
- Approvals and local evidence paths

Recheck the final diff and release artifact against approved scope. Flag unexpected dependency, lockfile, generated-file, infrastructure, permission, or environment changes.

## Compact Evidence

Create this record only when repository policy or the user requests it. Use an approved location; otherwise use ignored `.change-readiness/<change-id>.md`. Do not commit it unless requested.

```md
# Production change: <change-id>

Decision: <Ready | Ready with conditions | Not ready | Blocked>
Scope: <summary> | Risk: <Low | Medium | High>
Owner: <name or team> | Approver: <name or policy> | Date: <ISO-8601>
Impact: <users, services, data, contracts, environments>
Gates: <gate | result | command or artifact path>
Rollout: <sequence, canary or flag, observation window>
Stop conditions: <metrics, alerts, or operator signals>
Rollback: <action, duration, owner>
Recovery: <forward fix or restore plan when rollback is insufficient>
Residual risks: <none | risk, owner, acceptance or mitigation>
Open conditions: <none | exact requirement before release>
```

## Escalate or Defer

- Change scope, target environment, authorization, owner, or acceptance criteria is unclear.
- Required baseline or validation gate fails, cannot run, or has no canonical command.
- Security, dependency, privacy, compliance, data, or infrastructure review is required but unavailable.
- Migration, contract, permission, or external side effect cannot safely support mixed versions or recovery.
- Rollback is impossible, untested, or does not address data and side effects.
- Observability cannot detect the relevant failure before impact exceeds the recovery objective.
- More than three focused repairs are needed, or validation reveals unrelated repository failures.
