# Airtel Production Hotfix — Agent Response

---

## Question 

A production authentication issue exists in release/airtel.
I want to create a hotfix, merge it safely, tag the release and propagate the fix back to main.
Give me the exact commands and validations required

---

## Problem Statement

A production authentication issue has been identified in `release/airtel`. The fix must be isolated, validated, merged safely into the production branch, tagged as a release, and propagated back to `main` without destabilizing any other customer branch or the future development stream.

---

## Identified Issues

- **Authentication failure in production** — classified as **CRITICAL risk** under the repository's risk matrix; mandatory manual review applies at every stage.
- **Hotfix must be isolated** — `release/airtel` is a customer-specific branch; no cross-customer propagation unless explicitly validated.
- **Main branch must not be merged into release** — `main` contains unstable future work; propagation back must use cherry-pick only.
- **Merge history must remain auditable** — `--no-ff` is required; force-push and history rewriting are prohibited.

---

## Git Commands

**1. Sync and branch off production**
```bash
git checkout release/airtel
git pull origin release/airtel
git checkout -b hotfix/airtel-auth-fix
```
Ensures the hotfix starts from the latest deployed production state, not an outdated local copy.

---

**2. Confirm you are on the right branch**
```bash
git branch --show-current
```
Verifies the hotfix branch was created correctly before any changes are made.

---

**3. Stage and commit the fix**
```bash
git add .
git commit -m "HOTFIX: fix auth token expiry"
```
Commits the isolated fix with a traceable message.

---

**4. Merge hotfix into release using no-ff**
```bash
git checkout release/airtel
git pull origin release/airtel
git merge --no-ff hotfix/airtel-auth-fix
```
`--no-ff` forces a merge commit, preserving the hotfix in branch history for audit purposes. Direct commits to release are prohibited.

---

**5. Validate the merge**
```bash
git log --graph --decorate --oneline --all
```
Visually confirms the merge commit exists and the branch graph is clean before tagging.

---

**6. Tag the production release**
```bash
git tag airtel-v1.0.1
git push origin release/airtel --tags
```
Records the deployed production state. Tags are permanent and must never be deleted.

---

**7. Identify the commit SHA for propagation**
```bash
git log --oneline release/airtel -n 5
```
Retrieves the exact commit SHA needed for cherry-pick. Do not guess or copy from memory.

---

**8. Propagate fix to main via cherry-pick**
```bash
git checkout main
git pull origin main
git cherry-pick -x <commit_sha>
git push origin main
```
`-x` appends the origin commit reference to the message, keeping the fix traceable across branches. Merging the release branch directly into main is prohibited.

---

**9. Clean up the hotfix branch**
```bash
git branch -d hotfix/airtel-auth-fix
git push origin --delete hotfix/airtel-auth-fix
git fetch --prune
```
Hotfix branches are temporary. Removing them after merge keeps the repository clean.

---

**10. Verify no servicing drift remains**
```bash
git cherry -v release/airtel main
```
Confirms the fix is present in both branches and no cherry-picks have been missed.
