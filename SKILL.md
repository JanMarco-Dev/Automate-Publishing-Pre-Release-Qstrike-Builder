---
name: publish-prerelease-qstrike-builder
description: Use when publishing a pre-release version of QStrike Builder packages to npm from the staging branch, after an approved PR has been squash-and-merged into staging
---

'PROMPT: I need to publish a pre-release of QStrike Builder from the staging branch. The PR was just squashed and merged. What do I do?'

# Publish Pre-release QStrike Builder

## Overview

Step-by-step process for publishing a QStrike Builder pre-release via Lerna. Always runs from the `staging` branch, always publishes with an `alpha` prerelease identifier.

## Pre-conditions

- You are on the `staging` branch
- An approved PR (feature branch → staging) has been squash-and-merged

## Steps

### 1. Sync staging with main

After the approved PR has been squash-and-merged into staging:

```bash
git checkout main && git pull origin main
git checkout staging && git merge main
```

**If `git merge main` produces conflicts**, they are almost certainly version conflicts — `staging` has alpha versions (e.g. `3.11.5-alpha.0`) while `main` has stable versions (e.g. `3.11.4`). Since staging's alpha versions are correct for a pre-release publish, **abort the merge and skip it**:

```bash
git merge --abort
```

Then pull any new commits from `origin/staging` instead:

```bash
git pull origin staging
```

The merge with `main` is only needed to pick up hotfixes. If `staging` is already ahead of `main` with alpha version bumps, the merge will always conflict and is unnecessary — just ensure `staging` is up to date with its remote.

### 2. Clean build cache

```bash
rm -rf node_modules/.cache
```

### 3. Verify latest merged commit

```bash
git fetch origin
git log -1
```

Confirm the top commit is the one just merged — check the message and author.

### 4. Run a build first

Verify the build succeeds before publishing:

```bash
npx lerna run build
```

Fix any build errors before proceeding.

### 5. Publish pre-release

Run the publish command with the `prerelease` bump and `alpha` preid:

```bash
echo y | npm run publish -- prerelease --preid alpha --pre-dist-tag next
```

This expands to: `npx lerna run build && npx lerna publish prerelease --preid alpha --pre-dist-tag next`

Lerna will show the version bump list (e.g. `1.2.3-alpha.0`) — verify it looks correct, then the piped `y` confirms automatically. If the versions look wrong, cancel with Ctrl+C and investigate.

### 6. Verify npm packages updated

Run to confirm the pre-release version was published:

```bash
npx lerna ls --json | grep '"name"' | awk -F'"' '{print $4}' | xargs -I{} npm info {}@next version 2>/dev/null
```

Or check manually via GitHub **qstrike-builder** repository → **Packages** tab.

If the command prints version numbers (e.g. `1.2.3-alpha.0`), the publish succeeded.

### 7. Handle branch protection errors

If publish fails with a branch protection / push error:

**7a. Check if a publish commit was already created:**

```bash
git log --oneline -5
```

Look for a commit like `chore(release): publish` in the log.

**7b. If publish commit exists but wasn't pushed:**

```bash
git push origin staging
```

**7c. If no publish commit and packages weren't published:**

```bash
npm run publish -- from-package --pre-dist-tag next
```

This builds then publishes only packages whose version in `package.json` doesn't yet exist on npm. The `--pre-dist-tag next` flag ensures packages are tagged as `next`, not `latest`.

### 8. Create Release PR

First check if a `Release/next.` PR is already open:

```bash
gh pr list --base main --head staging --state open --json title --jq '.[].title'
```

- If the output includes `Release/next.` — skip this step, the PR already exists.
- If no PR exists, create one:

```bash
gh pr create --title "Release/next." --body "" --base main --head staging
```

This requires the GitHub CLI (`gh`) to be installed and authenticated. See `GITHUB_CLI_SETUP.md` if needed.

## Publishing Multiple Pre-releases

If a pre-release has already been published for the current version cycle, Lerna automatically increments the prerelease number on the next run:

| Publish run | Version example |
|-------------|----------------|
| 1st | `2.16.5-alpha.0` |
| 2nd | `2.16.5-alpha.1` |
| 3rd | `2.16.5-alpha.2` |

No extra flags needed — just run step 5 again after merging the new PR into staging. The Release PR (step 8) is also reused — do not open a new one if it already exists.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `rm -rf node_modules/.cache` before publishing | Always clear the cache — stale builds cause silent failures |
| Using `npm run publish` without `-- prerelease ...` | Without the bump arg lerna prompts interactively; always pass `prerelease --preid alpha --pre-dist-tag next` |
| Skipping `git log -1` after merge | Ensures you're publishing the right commit, not a stale one |
| PR title wrong | Must be exactly `Release/next.` (with the period) |
| Using `from-package` without `--pre-dist-tag next` | Packages get tagged as `latest` on npm, polluting the stable channel — always include `--pre-dist-tag next` in recovery publishes |
| Trying to resolve `git merge main` conflicts manually | These are alpha-vs-stable version conflicts — staging's versions are correct. Always `git merge --abort` and pull `origin/staging` instead |
