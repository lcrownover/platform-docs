# Recovery and Confidence

By the end of this section, you'll know how to undo mistakes at every stage of the Git workflow. More importantly, you'll have the confidence to experiment knowing you can always get back to a good state.

## The Safety Net Mindset

Git is designed to be recoverable. Almost nothing you do is permanent until you push to a shared remote, and even then there are options. The commands in this section are your safety net.

Before we dive in, remember: `git status` is your friend. When something goes wrong, run it first. Git usually tells you exactly what's happening and suggests how to fix it.

## Undoing Uncommitted Changes

These commands help when you've made changes to files but haven't committed yet.

### Discard changes to a file

You edited a file and want to throw away those changes, restoring it to the last committed version:

```bash
git restore notes.md
```

The file reverts to how it looked in your last commit. Your changes are gone (there's no undo for this, so be sure).

!!! warning "Destructive command"
    `git restore` permanently discards uncommitted changes. If you're unsure, stash your changes first (covered below) so you can get them back if needed.

To discard changes to all files in the current directory:

```bash
git restore .
```

### Unstage a file

You ran `git add` but changed your mind. The file is staged, but you want to remove it from the staging area without losing your changes:

```bash
git restore --staged notes.md
```

This only unstages the file. Your edits are still in the working directory, just no longer queued for the next commit. Run `git status` and you'll see the file moved from "Changes to be committed" back to "Changes not staged for commit."

From here you have two options:

- **Re-stage it later** with `git add notes.md` when you're ready
- **Discard the changes entirely** with `git restore notes.md` (no `--staged` flag)

The two-step process (unstage, then discard) is intentional. It prevents you from accidentally throwing away work when you just wanted to unstage.

To unstage everything:

```bash
git restore --staged .
```

### The old way: git checkout and git reset

Before `git restore` existed, people used `git checkout -- filename` to discard changes and `git reset HEAD filename` to unstage. You'll see these in older tutorials and Stack Overflow answers. They still work, but `git restore` is clearer about what it does.

## Stashing Changes

Sometimes you need to switch branches but have uncommitted work you're not ready to commit. Stashing saves your changes temporarily.

### Save your work

```bash
git stash
```

Your working directory is now clean (matching the last commit), and your changes are saved in the stash.

### See what's stashed

```bash
git stash list
```

```
stash@{0}: WIP on main: a1b2c3d Add backup procedures
```

### Get your changes back

```bash
git stash pop
```

This applies the most recent stash and removes it from the stash list. Your changes are back in your working directory.

If you want to apply the stash but keep it in the list (in case you need it again):

```bash
git stash apply
```

### Stash with a message

The default "WIP on branch" message isn't very descriptive. Add your own:

```bash
git stash push -m "Halfway through refactoring config parsing"
```

This makes it easier to remember what each stash contains if you have several.

## Amending Recent Commits

Made a commit and immediately realized you made a mistake? These commands fix the most recent commit.

### Fix the commit message

You just committed with a typo or unclear message:

```bash
git commit --amend -m "Add backup procedures section"
```

This replaces the previous commit message. The commit itself (the actual changes) stays the same.

### Add forgotten files

You committed but forgot to include a file:

```bash
git add forgotten-file.md
git commit --amend --no-edit
```

The `--no-edit` flag keeps the existing commit message. Your forgotten file is now part of the commit as if you'd included it originally.

### Add more changes to the last commit

Same idea, but for changes to files already in the commit:

```bash
# Make your additional edits
git add notes.md
git commit --amend --no-edit
```

### Fix an older commit

`--amend` only works on the most recent commit. To fix an older commit, use interactive rebase.

Say you want to fix the commit message from three commits ago:

```bash
git rebase -i HEAD~3
```

Git opens your editor with a list of the last three commits:

```
pick a1b2c3d Add backup procedures
pick b2c3d4e Add monitoring section
pick c3d4e5f Add contact info
```

Change `pick` to `reword` (or just `r`) on the commit you want to fix:

```
reword a1b2c3d Add backup procedures
pick b2c3d4e Add monitoring section
pick c3d4e5f Add contact info
```

Save and close. Git will open another editor for you to write the new commit message. Save that, and the rebase completes.

To change the contents of an older commit (not just the message), use `edit` instead of `reword`. Git will pause at that commit, letting you make changes and run `git commit --amend`, then `git rebase --continue` to finish.

!!! tip "Rebase options"
    The interactive rebase editor shows other options like `squash` (combine commits), `drop` (delete a commit), and `reorder` (change commit order). These are powerful but beyond the scope of this intro.

!!! warning "Don't amend or rebase pushed commits"
    Both amending and rebasing rewrite history. If you've already pushed these commits to a shared remote, rewriting them creates problems for anyone who pulled them. Only rewrite commits that exist only on your local machine.

## Resetting Commits

When you need to undo one or more commits entirely, `git reset` is the tool. It has three modes that determine what happens to your changes.

### Soft reset: undo commit, keep changes staged

```bash
git reset --soft HEAD~1
```

This undoes the last commit but keeps all changes staged. Useful when you want to re-commit with a different message or combine with other changes.

### Mixed reset (default): undo commit, keep changes unstaged

```bash
git reset HEAD~1
```

This undoes the last commit and unstages the changes, but keeps them in your working directory. You can review what was in the commit and decide what to do.

### Hard reset: undo commit, discard changes

```bash
git reset --hard HEAD~1
```

This undoes the last commit and throws away all changes. Your working directory matches the commit before the one you reset.

!!! danger "Hard reset is destructive"
    `git reset --hard` permanently discards commits and changes. Triple-check you're resetting to the right commit. If you reset too far, see "Recovering lost commits" below.

### Reset multiple commits

`HEAD~1` means "one commit before HEAD." Adjust the number:

```bash
git reset --soft HEAD~3    # Undo last 3 commits, keep changes staged
```

Or reset to a specific commit by hash:

```bash
git reset --hard a1b2c3d
```

## Common "I Messed Up" Scenarios

Here are solutions to situations that make people panic.

### "I committed to the wrong branch"

You meant to commit to a feature branch but accidentally committed to `main`.

**Solution:** Move the commit to the right branch.

```bash
# Create a new branch pointing at your current commit
git branch feature-branch

# Reset main back one commit (keeping changes if you use --soft)
git reset --hard HEAD~1

# Switch to your feature branch
git checkout feature-branch
```

Your commit is now on `feature-branch`, and `main` is back where it was.

### "I need to undo a commit that's already pushed"

You can't rewrite history that others might have pulled. Instead, create a new commit that reverses the changes:

```bash
git revert a1b2c3d
```

This creates a new commit that undoes everything in commit `a1b2c3d`. The original commit stays in history (for accountability), but its effects are reversed.

To revert the most recent commit:

```bash
git revert HEAD
```

### "I accidentally deleted a branch"

If you deleted a branch with `-d` or `-D`, the commits still exist. Find the commit hash:

```bash
git reflog
```

The reflog shows recent commits you've visited, even on deleted branches. Find the commit hash and recreate the branch:

```bash
git branch recovered-branch a1b2c3d
```

### "I did a hard reset and lost commits"

Same solution: use `git reflog` to find the commit hash, then reset back to it:

```bash
git reflog
# Find the commit you want to return to
git reset --hard a1b2c3d
```

!!! tip "Reflog is your last resort"
    Git keeps reflog entries for about 90 days by default. If you realize you lost something, act soon. The reflog is local only (it doesn't sync with remotes).

### "I'm in a merge conflict and want to start over"

```bash
git merge --abort
```

This cancels the merge and returns your branch to its pre-merge state. You can try again or take a different approach.

Similarly, for a rebase gone wrong:

```bash
git rebase --abort
```

### "I have uncommitted changes and can't switch branches"

Git won't let you switch branches if you have uncommitted changes that would conflict. You have options:

1. **Commit your work** (even as a work-in-progress):
   ```bash
   git add .
   git commit -m "WIP: save progress before switching"
   ```

2. **Stash your changes**:
   ```bash
   git stash
   git checkout other-branch
   # Later, when you come back:
   git stash pop
   ```

3. **Discard your changes** (if you don't need them):
   ```bash
   git restore .
   ```

### "Everything is broken and I want to start fresh"

Nuclear option: reset your branch to match the remote exactly.

```bash
git fetch origin
git reset --hard origin/main
```

This throws away all local commits and changes, making your local `main` identical to the remote. Only do this if you're sure you don't need anything local.

## Building Confidence

The best way to get comfortable with recovery commands is to practice them intentionally in your sandbox repository. Break things on purpose:

1. Make commits, then undo them with `git reset`
2. Create merge conflicts, then abort with `git merge --abort`
3. Delete branches, then recover them with `git reflog`
4. Stash changes, switch branches, then pop the stash

The more you practice recovery, the less scary mistakes become. Git almost always has a way out.

## Exercises

These exercises use your sandbox repository. If something goes wrong, you can always delete the sandbox and start fresh.

1. **Practice restore:** Modify `notes.md`, then use `git restore` to discard the changes. Verify the file is back to its committed state.

2. **Unstage a file:** Stage a change with `git add`, then unstage it with `git restore --staged`. Confirm the change is still in your working directory but no longer staged.

3. **Amend a commit:** Make a commit with a typo in the message. Use `git commit --amend` to fix the message. Check `git log` to verify.

4. **Use the stash:** Make some changes, stash them, verify your working directory is clean, then pop the stash and verify your changes are back.

5. **Reset commits:** Make two commits, then use `git reset --soft HEAD~2` to undo them while keeping changes staged. Re-commit everything as a single commit.

6. **Explore reflog:** Run `git reflog` and examine the output. Identify commits from branches you've deleted or resets you've done.
