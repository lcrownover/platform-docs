# Collaboration

By the end of this section, you'll know how to work effectively with others on a shared repository, including the pull request workflow and best practices for keeping your team's codebase healthy.

## Working on a Team

When you're working alone, you can commit whenever and however you want. On a team, your habits affect everyone. Good collaboration practices prevent conflicts, keep history readable, and make code review effective.

## Daily Habits

These habits prevent most collaboration problems before they start.

### Pull before you start working

At the beginning of each work session:

```bash
git checkout main
git pull
```

This ensures you're starting from the latest code. Many merge conflicts happen because someone started work on an outdated branch.

### Work on branches, not main

Even for small changes, create a branch:

```bash
git checkout -b fix-config-typo
```

This keeps `main` clean and makes code review possible. See [Branching and Merging](03-branching-and-merging.md) for details.

### Keep branches short-lived

The longer your branch lives, the more it diverges from `main`, and the harder it becomes to merge. Aim to merge branches within a few days.

If you're working on something larger, regularly merge `main` into your branch to stay current:

```bash
git checkout my-feature
git merge main
```

### Pull before you push

Before pushing, pull to get any changes others have made:

```bash
git pull
git push
```

If there are conflicts, resolve them locally rather than forcing your changes over others'.

## The Pull Request Workflow

Pull requests (PRs) are how most teams review and merge code. Instead of pushing directly to `main`, you push a branch and ask others to review it.

### The basic flow

**Step 1: Create a branch and make your changes**

```bash
git checkout -b add-logging
# ... make commits ...
```

**Step 2: Push your branch to the remote**

```bash
git push -u origin add-logging
```

**Step 3: Open a pull request**

On GitHub, GitLab, or Bitbucket, navigate to your repository and click "New Pull Request" (or similar). Select your branch and the target branch (usually `main`).

**Step 4: Write a good PR description**

- Summarize what the change does and why
- Link to any related issues or tickets
- Note anything reviewers should pay attention to
- Include testing instructions if applicable

**Step 5: Address review feedback**

Reviewers may request changes. Make additional commits on your branch and push them:

```bash
# Make requested changes
git add .
git commit -m "Address review feedback"
git push
```

The PR updates automatically.

**Step 6: Merge when approved**

Once reviewers approve, merge the PR (usually via the web interface). Most teams delete the branch after merging.

### Keeping your PR up to date

If `main` changes while your PR is open, you may need to update your branch:

```bash
git checkout my-feature
git fetch origin
git merge origin/main
# Resolve any conflicts
git push
```

Some teams prefer rebasing:

```bash
git checkout my-feature
git fetch origin
git rebase origin/main
# Resolve any conflicts
git push --force-with-lease
```

The `--force-with-lease` flag is safer than `--force` because it fails if someone else pushed to your branch.

## Squashing Commits

When you're working on a feature, you might make many small commits: "WIP", "fix typo", "actually fix it this time". These are useful while working but clutter the history when merged.

Squashing combines multiple commits into one clean commit.

### Why squash?

- **Cleaner history.** `main` shows one commit per feature, not dozens of work-in-progress commits.
- **Easier reverts.** If something breaks, you can revert one commit instead of hunting through many.
- **Better blame.** `git blame` shows meaningful commits, not "fix typo".

### Squash when merging (easiest)

Most Git hosts offer a "Squash and merge" button when merging PRs. This combines all commits in your branch into one commit on `main`. You write a new commit message summarizing the change.

This is the easiest approach and doesn't require any local Git commands.

### Squash locally before pushing

If you want to squash before opening the PR (or if your team requires it), use interactive rebase.

Say you have four commits on your branch:

```bash
git log --oneline -4
```

```
d4e5f6g Fix typo in error message
c3d4e5f Add error handling
b2c3d4e Add logging function
a1b2c3d Initial implementation
```

To squash these into one commit:

```bash
git rebase -i HEAD~4
```

Git opens your editor:

```
pick a1b2c3d Initial implementation
pick b2c3d4e Add logging function
pick c3d4e5f Add error handling
pick d4e5f6g Fix typo in error message
```

Change `pick` to `squash` (or `s`) for all commits except the first:

```
pick a1b2c3d Initial implementation
squash b2c3d4e Add logging function
squash c3d4e5f Add error handling
squash d4e5f6g Fix typo in error message
```

Save and close. Git opens another editor for you to write the combined commit message. Write something descriptive:

```
Add logging with error handling

- Implement logging function
- Add error handling for edge cases
- Include meaningful error messages
```

Save, and your four commits are now one.

### Push after squashing

If you've already pushed the branch, you'll need to force push after squashing:

```bash
git push --force-with-lease
```

!!! warning "Only squash your own branches"
    Squashing rewrites history. Only do this on branches that are exclusively yours. Never squash commits that others have based work on.

## Code Review Best Practices

### As the author

- **Keep PRs small.** Smaller changes are easier to review and less likely to have bugs. If a feature is large, break it into smaller PRs.
- **Write a clear description.** Explain what and why, not just what files changed.
- **Respond to feedback gracefully.** Reviewers are trying to help. If you disagree, discuss it.
- **Test before requesting review.** Don't waste reviewers' time on broken code.

### As a reviewer

- **Be constructive.** Suggest improvements, don't just criticize.
- **Explain your reasoning.** "This could cause a race condition because..." is more helpful than "This is wrong."
- **Approve when it's good enough.** Perfect is the enemy of good. If it works and is maintainable, approve it.
- **Review promptly.** Slow reviews block others' work.

## Force Push Safety

`git push --force` overwrites the remote branch with your local version, destroying any commits that aren't in your local copy.

```bash
# Dangerous on shared branches!
git push --force
```

### When force push is acceptable

- On your own feature branch that no one else is using
- After squashing commits before merge
- After rebasing to update your branch

### When force push is dangerous

- On `main` or any shared branch
- On any branch someone else might have pulled

### Use --force-with-lease instead

```bash
git push --force-with-lease
```

This fails if the remote branch has commits you don't have locally (meaning someone else pushed). It's not foolproof, but it catches the most common mistakes.

!!! danger "Never force push to main"
    Force pushing to `main` can delete others' work and break everyone's local repositories. Many teams protect `main` to prevent this entirely.

## Common Scenarios

### "My PR has conflicts with main"

```bash
git checkout my-feature
git fetch origin
git merge origin/main
# Resolve conflicts in your editor
git add .
git commit -m "Merge main into my-feature"
git push
```

### "I need to update my PR with more changes"

Just commit and push to the same branch:

```bash
git checkout my-feature
# Make changes
git add .
git commit -m "Add requested error handling"
git push
```

The PR updates automatically.

### "I want to squash my commits before merging"

```bash
git checkout my-feature
git rebase -i HEAD~n  # n = number of commits to squash
# Change 'pick' to 'squash' for all but the first
# Write a new commit message
git push --force-with-lease
```

### "Someone else pushed to my branch"

Pull their changes before continuing:

```bash
git checkout my-feature
git pull
```

If you've already made local commits, this creates a merge commit. That's fine.

## Exercises

1. **Practice the PR workflow:** Create a branch, make a few commits, push it, and open a PR (even if you're the only reviewer). Merge it using the web interface.

2. **Try squash and merge:** Open a PR with multiple commits. When merging, use the "Squash and merge" option. Check the history on `main` to see the single combined commit.

3. **Squash locally:** Create a branch with 3-4 small commits. Use `git rebase -i` to squash them into one. Push with `--force-with-lease`.

4. **Resolve a conflict:** Create two branches from `main`. On each, edit the same line differently. Merge one to `main`. On the other branch, merge `main` in and resolve the conflict. Push and complete the PR.

5. **Review a PR:** If you're working with others, review someone else's PR. Practice leaving constructive feedback.
