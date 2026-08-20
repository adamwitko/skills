---
name: node-upgrader
description: "Use when upgrading Node.js runtime versions or pins. Provides approval-gated, context-safe upgrades for Node.js projects."
user-invocable: true
---

# Node.js Upgrader

Upgrade Node.js runtime pins safely. Default scope: runtime pins only. Do not update dependencies, package manager, lockfile, application code, infrastructure, branches, worktrees, commits, or deployments unless user explicitly approves scope expansion.

## Operating Rules

- Prefer the current supported LTS release and an exact `major.minor.patch` pin. A Current release requires documented approval. Record target EOL date and support from every deployment platform in scope.
- Preserve existing worktree. Do not create a branch or worktree without approval.
- Use the project's existing runtime manager, package manager, and lockfile strategy. Before any dependency install, update, audit, or resolution, run the repository's approved dependency and supply-chain scan. Stop on scan failure or a malware finding.
- Use deterministic, scripts-disabled installation. Enable scripts only when repository policy and user approval permit it.
- Never paste secrets, environment dumps, lockfile contents, full test logs, or command output into chat or evidence. Report concise summaries and local artifact paths only.
- Work in phases. Complete a phase and obtain required approval before starting next phase.

## Compact Evidence

Keep one concise record only when requested by user or repository policy. Use approved location; otherwise create an ignored local file at `.upgrade-evidence/node-<target>.md`. Do not commit it unless user requests.

```md
# Node upgrade: <current> -> <target>

Scope: runtime pins only | Approved by: <user/policy> | Date: <ISO-8601>
Target: <exact version>, <LTS/Current>, EOL <date>, deployment-platform support <result>
Pins: <path or setting: current -> target>
Compatibility: <compatible | risk and owner>
Gates: <command | baseline exit/duration | target exit/duration | artifact path>
Changes: <files>
Rollback: restore <pins>; select <prior runtime>; run <targeted gates>
Open risks: <none | details>
```

## Workflow

### 1. Discover (read-only)

Record repository status, current Node and package-manager versions, package manager, lockfile strategy, workspace structure, supported commands, and runtime pins. Search version-manager files such as `.nvmrc`, `.node-version`, and `.tool-versions`; `package.json` (`engines`, `packageManager`); CI workflows; Dockerfiles; devcontainers; buildpacks; deployment manifests; hosting configuration; and runtime documentation.

Treat missing pins as findings, not permission to introduce tooling. For each command, capture exit code, duration, one-line result, and local evidence path; retain full output locally when repository policy or audit requirements need it.

### 2. Approve Target and Scope (required)

Present exact target, release channel, EOL date, deployment-platform support, every pin to change, validation command matrix, and any requested scope expansion. Stop until user or documented policy approves target and scope.

### 3. Baseline

Run approved baseline gates on current runtime. Start with the project's format, lint, typecheck, build, and unit/integration test commands; add E2E, accessibility, security, dependency, and smoke checks according to repository policy and change risk. Use the same command matrix after upgrade.

Stop and report baseline failures. Do not repair pre-existing failures within this workflow.

### 4. Compatibility and Preparation

Read Node migration guidance for the source-to-target range. Check direct dependencies, native modules, build tools, test runners, package-manager support, supported operating systems, and every deployment platform in scope. Record only actionable findings.

Stop and ask approval when dependency, package-manager, framework, infrastructure, or behavioural application change is required. Do not install or modify dependencies without approved expanded scope and successful malware scan.

### 5. Upgrade and Validate

Install/select approved target runtime using the project's existing approved runtime manager or platform mechanism. Confirm exact `node --version`. Update only approved runtime pins, then run the target command matrix.

On failure, classify it as baseline, target-runtime regression, environment, or inconclusive. Repair only approved target-runtime regressions. Maximum three focused repairs per issue; rerun failed gate after each repair. Stop after third failed repair or when validation cannot complete.

### 6. Review and Close

Confirm diff contains only approved files and no unexpected lockfile, package-manager, generated-file, or dependency changes. Record concise evidence and rollback procedure.

For runtime-pin-only changes, validate rollback procedure by restoring prior pins/runtime and rerunning targeted baseline gates when feasible. Require a full rollback drill only for approved deployment or platform changes.

## Escalate Immediately

- Target, support policy, scope, or required approval is unclear.
- Baseline fails, malware scan fails, or malware scan reports a malware finding.
- Deployment targets require incompatible Node versions.
- Compatibility needs unapproved dependency, package-manager, framework, infrastructure, or behavioural changes.
- More than three focused repairs are needed, more than ten application files would change, or required validation cannot complete.
