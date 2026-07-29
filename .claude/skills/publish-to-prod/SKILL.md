---
name: publish-to-prod
description: Publish the latest committed work on main to the live site by fast-forwarding the prod branch, which is what GitHub Pages actually builds from. Use when the user asks to publish, deploy, "align with prod", or "push to prod".
---

# Publish to prod

This repo's GitHub Pages is configured to build from the `prod` branch, not `main`.
All normal work happens on `main`; nothing goes live until `prod` is moved forward to match it.
`prod` should only ever move via fast-forward — never rebase, merge commit, or force-push it.

## Steps

1. Confirm the working tree is clean (commit or ask the user to commit first if not):
   ```
   git status
   ```

2. Fetch and see what's new on `main` that hasn't reached `prod` yet:
   ```
   git fetch origin --quiet
   git log --oneline origin/prod..origin/main
   ```
   If this is empty, `prod` is already current — nothing to do.

3. Verify `prod` is a clean ancestor of `main` before touching it. This must be a fast-forward, never a merge or force-push:
   ```
   git merge-base --is-ancestor origin/prod origin/main && echo "safe fast-forward" || echo "NOT an ancestor — stop and investigate"
   ```
   If it's not an ancestor, stop and tell the user — something diverged on `prod` and needs manual review before overwriting it.

4. Fast-forward `prod` to `main`:
   ```
   git push origin main:prod
   ```

5. Poll the live site until the deploy lands (GitHub Pages builds take roughly 30-90 seconds). Check for a string known to be in the new content, not just a 200 status:
   ```
   for i in $(seq 1 12); do
     html=$(curl -s https://chc012.github.io/)
     if echo "$html" | grep -q "<some marker string from the latest change>"; then
       echo "LIVE and updated (attempt $i)"
       break
     fi
     sleep 10
   done
   ```

## Notes

- This is a real, visible-to-the-world publish action. Only do it when the user actually asks to publish/deploy/align-with-prod — not automatically after every commit.
- If `git status` shows uncommitted changes, don't publish partial work — ask whether to commit first.
