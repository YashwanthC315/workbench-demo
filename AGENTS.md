# AGENTS.md

# Agent Operational Guide

This document defines repository topology, servicing rules, operational procedures, Git workflows, validation requirements, safety constraints and recovery behavior for AI agents and developers operating within this repository.

Agents must use this guide to:
- understand repository structure
- reason about release servicing
- recommend safe Git operations
- generate Git command sequences
- propagate hotfixes safely
- detect servicing drift
- identify merge risks
- preserve customer isolation
- maintain release traceability

---

# Repository Layout

```text
main
│
├── develop
│
├── release/airtel
│   └── hotfix/airtel-auth-fix
│
├── release/reliance
│
├── release/tata
│
└── feature/branch-visualization
```

---

# Repository Topology

## Branch Types

| Pattern | Example | Purpose |
|---|---|---|
| `main` | `main` | future product development |
| `develop` | `develop` | integration branch |
| `release/<customer>` | `release/airtel` | customer production servicing |
| `release/<version>` | `release/v2.1` | shared stabilization branch |
| `hotfix/<customer>-<issue>` | `hotfix/airtel-auth-fix` | customer production hotfix |
| `hotfix/<severity>-<issue>` | `hotfix/p1-login-outage` | critical emergency fix |
| `feature/<name>` | `feature/dashboard-ui` | feature development |

---

# Source Of Truth

| Branch | Source Of Truth |
|---|---|
| `main` | future product evolution |
| `develop` | pre-release integration |
| `release/*` | deployed production state |
| `hotfix/*` | temporary remediation work |

---

# Branch Rules

## Main Branch

- contains unstable future work
- may diverge from production branches
- receives propagated hotfixes using cherry-pick
- must never be merged directly into release branches

## Release Branches

- represent deployable customer production state
- are long-lived
- must remain taggable
- must remain deployable
- receive validated hotfixes only

## Hotfix Branches

- must originate from release branches
- are temporary
- must use `--no-ff` merge strategy
- may be deleted after merge

## Feature Branches

- used for non-production feature development
- must not directly service production issues

---

# Naming Rules

- branch names must use lowercase
- words must use hyphen separators
- release branches must begin with `release/`
- production fixes must begin with `hotfix/`
- feature branches must begin with `feature/`
- hotfix names should describe production issue

Examples:

```text
release/airtel
hotfix/airtel-auth-fix
feature/branch-graph
```

---

# Customer Isolation

Customer release branches may contain:
- customer-specific configuration
- deployment manifests
- feature flags
- environment variables
- branding assets
- integration settings

Agents must assume customer branches are NOT interchangeable.

Cross-customer propagation requires validation.

---

# Release Strategy

## Tag Format

```text
<customer>-v<major>.<minor>.<patch>
```

Examples:

```text
airtel-v1.0.0
airtel-v1.0.1
reliance-v2.3.4
```

## Patch Rules

- patch increments represent hotfix releases
- tags represent deployed production state
- all production releases must be tagged
- tags must never be deleted

---

# Operational Decision Matrix

| Situation | Recommended Action |
|---|---|
| fix exists only in release branch | cherry-pick to main |
| fix required across customers | cherry-pick across release branches |
| customer-specific configuration change | do not propagate |
| dependency upgrade | require compatibility validation |
| authentication change | require manual review |
| schema migration | require manual review |
| README-only fix | low risk propagation |
| release drift detected | inspect divergence before propagation |

---

# Merge Policy

## Allowed

| Operation | Allowed |
|---|---|
| hotfix -> release using `--no-ff` | yes |
| release -> main via cherry-pick | yes |
| release/customerA -> release/customerB | restricted |
| feature -> develop | yes |

## Prohibited

| Operation | Allowed |
|---|---|
| merge main -> release | no |
| force-push release branches | no |
| delete production tags | no |
| rewrite published history | no |
| auto-resolve customer config conflicts | no |

---

# Recommended Git Strategies

| Scenario | Recommended Command |
|---|---|
| production hotfix | `git merge --no-ff` |
| servicing propagation | `git cherry-pick -x` |
| inspect merge before commit | `git merge --no-commit` |
| strict linear merge | `git merge --ff-only` |
| cleanup feature history | `git merge --squash` |

---

# Scenario: Create Production Hotfix

## Intent

Create isolated production remediation branch from customer release branch.

## Preconditions

- production issue identified
- correct release branch identified

## Commands

```bash
git checkout release/airtel

git pull origin release/airtel

git checkout -b hotfix/airtel-auth-fix
```

## Validation

- confirm branch created from correct release branch

```bash
git branch --show-current
```

---

# Scenario: Apply Hotfix Changes

## Commands

```bash
git status

git add .

git commit -m "HOTFIX: fix auth token expiry"
```

## Validation

```bash
git log --oneline -n 5
```

---

# Scenario: Merge Hotfix Into Release

## Intent

Merge validated hotfix into production servicing branch.

## Commands

```bash
git checkout release/airtel

git pull origin release/airtel

git merge --no-ff hotfix/airtel-auth-fix
```

## Validation

```bash
git status

git log --graph --decorate --oneline --all
```

## Restrictions

- hotfix merges must use `--no-ff`
- direct commits to release branches prohibited

---

# Scenario: Tag Production Release

## Commands

```bash
git tag airtel-v1.0.1

git push origin release/airtel --tags
```

## Validation

```bash
git tag --sort=-creatordate
```

---

# Scenario: Propagate Hotfix To Main

## Intent

Forward propagate validated production fix into future development stream.

## Preconditions

- hotfix merged into release branch
- hotfix validated
- commit SHA identified

## Commands

```bash
git checkout main

git pull origin main

git cherry-pick -x <commit_sha>

git push origin main
```

## Validation

```bash
git status

git log --oneline --decorate --graph
```

## Restrictions

- do not merge release branch into main
- use cherry-pick with `-x`

---

# Scenario: Propagate Hotfix Across Customers

## Intent

Apply shared production fix to another customer release branch.

## Commands

```bash
git checkout release/reliance

git pull origin release/reliance

git cherry-pick -x <commit_sha>

git push origin release/reliance
```

## Validation

```bash
git status
```

## Restrictions

- validate compatibility before propagation
- do not propagate customer-specific assets

---

# Scenario: Resolve Cherry-Pick Conflict

## Detect Conflicts

```bash
git status

git diff --name-only --diff-filter=U
```

## Inspect Conflicts

```bash
git diff
```

## Conflict Markers

```text
<<<<<<< HEAD
=======
>>>>>>>
```

## Resolution Process

1. inspect both changes
2. preserve customer-specific configuration
3. remove conflict markers
4. validate resolved content

## Continue Cherry-Pick

```bash
git add .

git cherry-pick --continue
```

## Abort Cherry-Pick

```bash
git cherry-pick --abort
```

---

# Scenario: Abort Failed Merge

## Commands

```bash
git merge --abort
```

---

# Scenario: Abort Failed Rebase

## Commands

```bash
git rebase --abort
```

---

# Scenario: Detect Release Drift

## Intent

Identify divergence between release and main branches.

## Commands

```bash
git log release/airtel..main

git log main..release/airtel
```

## Detect Missing Cherry-Picks

```bash
git cherry -v release/airtel main
```

---

# Scenario: Inspect Repository State

## Visualize Branch Graph

```bash
git log --graph --decorate --oneline --all
```

## Detailed Commit History

```bash
git log --oneline --decorate --graph
```

## Check Branch Containing Commit

```bash
git branch --contains <sha>
```

## Inspect Commit

```bash
git show <sha>
```

## View Current Branch

```bash
git branch --show-current
```

## View Tracking Information

```bash
git branch -vv
```

## Inspect Remote Configuration

```bash
git remote -v
```

## View Tags

```bash
git tag
```

## Inspect Tag

```bash
git show <tag>
```

---

# Scenario: Validate Repository State

## Commands

```bash
git status
```

## Validation Requirements

Before merging or tagging:

- unit tests must pass
- smoke tests must pass
- deployment manifests must validate
- no unresolved conflicts allowed
- release branch must remain deployable

---

# Scenario: Cleanup Hotfix Branch

## Delete Local Hotfix Branch

```bash
git branch -d hotfix/airtel-auth-fix
```

## Delete Remote Hotfix Branch

```bash
git push origin --delete hotfix/airtel-auth-fix
```

## Remove Stale References

```bash
git fetch --prune
```

---

# Scenario: Inspect Merged Branches

## Commands

```bash
git branch --merged
```

---

# Recovery Rules

## If Cherry-Pick Fails

- inspect conflicted files
- preserve customer configuration
- never auto-resolve production deployment conflicts
- abort operation if validation fails

Commands:

```bash
git status

git diff --name-only --diff-filter=U

git cherry-pick --abort
```

---

## If Merge Fails

Commands:

```bash
git merge --abort
```

Restrictions:

- never reset shared release branches
- never force-push release branches

---

## If Rebase Fails

Commands:

```bash
git rebase --abort
```

---

# Risk Classification

| Change Type | Risk | Manual Review Required |
|---|---|---|
| README change | low | no |
| feature flag change | medium | yes |
| dependency upgrade | high | yes |
| authentication change | critical | mandatory |
| schema migration | critical | mandatory |
| deployment manifest change | critical | mandatory |

---

# Safety Constraints

Agents must never:
- merge main directly into release branches
- execute `git push --force` on protected branches
- delete production tags
- rewrite published history
- auto-resolve customer deployment conflicts
- auto-tag releases without validation
- merge unrelated release branches
- bypass validation requirements

---

# Preferred Commands

## Preferred

```bash
git merge --no-ff

git cherry-pick -x

git merge --ff-only

git merge --abort

git cherry-pick --abort
```

## Restricted

```bash
git merge -s ours

git merge -X theirs

git merge --allow-unrelated-histories

git push --force
```

---

# Operational Principles

- release branches represent deployable production state
- production fixes must be traceable
- cherry-pick preferred over merge for servicing
- customer branches are isolated by default
- all production releases require tags
- servicing history must remain auditable
- safety overrides automation