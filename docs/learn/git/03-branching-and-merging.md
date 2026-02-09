# Branching and Merging

By the end of this section, you'll understand how to work on isolated changes without affecting your main code, and how to combine those changes back together when they're ready.

## Why Branches Exist

So far, all your commits have been on a single line of history. That works fine when you're the only person working on something simple. But what happens when you want to:

- Try a risky change without breaking what already works?
- Work on two different features at the same time?
- Let multiple people work on the same codebase without stepping on each other?

Branches solve all of these. A branch is just a pointer to a commit, and you can have as many as you want. When you create a new branch, you're saying "I want to start a separate line of work from this point." Your main branch stays safe while you experiment.

Think of it like making a copy of a document before editing it. Except Git does this efficiently (it doesn't actually copy all the files), and it gives you powerful tools to merge your changes back later.

## Creating and Switching Branches

### See your current branch

You've actually been on a branch this whole time. Check which one:

```bash
git branch
```

```
* main
```

The `*` marks your current branch. Right now, `main` is probably your only branch.

### Create a new branch

Let's create a branch to add a new feature to your notes:

```bash
git branch add-backup-notes
```

Now list branches again:

```bash
git branch
```

```
  add-backup-notes
* main
```

You've created the branch, but you're still on `main` (note the `*`).

### Switch to the new branch

```bash
git checkout add-backup-notes
```

```
Switched to branch 'add-backup-notes'
```

Now `git branch` shows:

```
* add-backup-notes
  main
```

!!! tip "Create and switch in one command"
    `git checkout -b branch-name` creates a new branch and switches to it immediately. This is what you'll use most of the time.

### Make changes on the branch

Now any commits you make will only affect this branch. Let's add some content:

```bash
echo "## Backup Procedures" >> notes.md
echo "Daily: rsync to backup server" >> notes.md
git add notes.md
git commit -m "Add backup procedures section"
```

Check your history:

```bash
git log --oneline
```

You'll see your new commit at the top. But here's the key insight: this commit only exists on `add-backup-notes`. Switch back to `main` and look:

```bash
git checkout main
git log --oneline
```

Your "Add backup procedures section" commit isn't there. Check the file:

```bash
cat notes.md
```

The backup procedures aren't there either. They're safely isolated on your feature branch.

### The shortcut: git switch

Git added `git switch` as a clearer alternative to `git checkout` for branch operations:

```bash
git switch add-backup-notes    # switch to existing branch
git switch -c new-branch       # create and switch (like checkout -b)
```

Both `checkout` and `switch` work. Use whichever you prefer, but know that `checkout` does other things too (like restoring files), while `switch` is only for branches.

## Merging Branches

Once your feature is ready, you'll want to merge it back into `main`.

### Fast-forward merge

The simplest case is when `main` hasn't changed since you branched. Git can just move `main` forward to include your new commits.

Make sure you're on `main`:

```bash
git checkout main
```

Now merge your feature branch:

```bash
git merge add-backup-notes
```

```
Updating a1b2c3d..f5e6d7c
Fast-forward
 notes.md | 2 ++
 1 file changed, 2 insertions(+)
```

"Fast-forward" means Git just moved the `main` pointer forward. No merge commit needed.

Check your log:

```bash
git log --oneline
```

Your backup procedures commit is now part of `main`.

### Three-way merge

Fast-forward only works when the target branch hasn't changed. When both branches have new commits, Git performs a three-way merge and creates a merge commit.

Let's set up that scenario. First, create and switch to a new branch:

```bash
git checkout -b add-monitoring-notes
echo "## Monitoring" >> notes.md
echo "Check Nagios dashboard hourly" >> notes.md
git add notes.md
git commit -m "Add monitoring section"
```

Now switch back to `main` and make a different change:

```bash
git checkout main
echo "## Contact Info" >> notes.md
echo "On-call: ops@example.com" >> notes.md
git add notes.md
git commit -m "Add contact info section"
```

Now `main` and `add-monitoring-notes` have diverged (both have commits the other doesn't). Merge the feature branch:

```bash
git merge add-monitoring-notes
```

Git opens your editor to write a merge commit message. The default message (usually "Merge branch 'add-monitoring-notes'") is fine. Save and close.

Check the graph view:

```bash
git log --oneline --graph
```

```
*   8c9d0e1 Merge branch 'add-monitoring-notes'
|\
| * 4a5b6c7 Add monitoring section
* | 7d8e9f0 Add contact info section
|/
* f5e6d7c Add backup procedures section
* ...
```

You can see where the branches diverged and came back together.

### Delete merged branches

After merging, you don't need the feature branch anymore:

```bash
git branch -d add-backup-notes
git branch -d add-monitoring-notes
```

The `-d` flag only works if the branch has been merged. This prevents you from accidentally deleting unmerged work. (Use `-D` to force-delete an unmerged branch, but be careful.)

## Handling Merge Conflicts

Merge conflicts happen when both branches changed the same part of the same file. Git can't automatically decide which version to keep, so it asks you to resolve it manually.

### Create a conflict

Let's intentionally create one. Start a new branch:

```bash
git checkout -b update-contact
```

Edit the contact info (change it to something different):

```bash
# Using sed to replace the email
sed -i '' 's/ops@example.com/oncall@example.com/' notes.md
git add notes.md
git commit -m "Update contact email to oncall address"
```

Now switch to `main` and make a conflicting change to the same line:

```bash
git checkout main
sed -i '' 's/ops@example.com/support@example.com/' notes.md
git add notes.md
git commit -m "Update contact email to support address"
```

Both branches changed the same line differently. Try to merge:

```bash
git merge update-contact
```

```
Auto-merging notes.md
CONFLICT (content): Merge conflict in notes.md
Automatic merge failed; fix conflicts and then commit the result.
```

### Understand the conflict markers

Open `notes.md`. You'll see something like:

```
## Contact Info
<<<<<<< HEAD
On-call: support@example.com
=======
On-call: oncall@example.com
>>>>>>> update-contact
```

The markers show:

- `<<<<<<< HEAD` to `=======`: The version on your current branch (main)
- `=======` to `>>>>>>> update-contact`: The version on the branch you're merging

### Resolve the conflict

Edit the file to keep the version you want (or combine them). Remove the conflict markers entirely. For example, if you decide `oncall@example.com` is correct:

```
## Contact Info
On-call: oncall@example.com
```

Or maybe you want both:

```
## Contact Info
On-call: oncall@example.com
Support: support@example.com
```

After editing, stage and commit:

```bash
git add notes.md
git commit -m "Merge update-contact, keep oncall email"
```

The merge is complete.

### Check status during conflicts

If you get lost during a conflict, `git status` tells you exactly what's happening:

```bash
git status
```

```
On branch main
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
	both modified:   notes.md
```

!!! tip "Abort if needed"
    If you get into a messy conflict and want to start over, run `git merge --abort`. This cancels the merge and returns your branch to its pre-merge state.

## When to Branch

**Branch by default.** In professional workflows, `main` represents stable, reviewed code. Committing directly to it should be the exception, not the rule.

Creating a branch takes seconds. Even for small changes, the habit of branching pays off because it builds muscle memory for when it really matters. More importantly, branches enable code review through pull requests, where teammates can catch issues before they reach `main`.

### When committing directly to main is acceptable

| Scenario | Why it's okay |
|----------|---------------|
| Fixing a typo you just introduced | You're immediately correcting your own recent mistake |
| True emergencies when you're the only responder | Speed matters, but document what you did afterward |

For personal projects with no collaborators, the stakes are lower, but branching is still good practice.

!!! note "Team policies vary"
    Many teams protect `main` entirely, requiring all changes to go through pull requests. Even if your team doesn't enforce this, treating `main` as protected builds good habits. For more on how teams structure their branching workflows, see [Branching Strategies](07-branching-strategies.md).

### Branch naming conventions

Good branch names describe what you're working on:

```
add-ssl-rotation
fix-backup-timeout
update-nginx-config
```

Some teams use prefixes to categorize work:

```
feature/add-ssl-rotation
bugfix/backup-timeout
hotfix/critical-security-patch
```

Pick a convention and stick with it. The name should tell someone what to expect without reading the code.

## Exercises

These exercises use your sandbox repository.

1. **Create and merge a branch:** Create a branch called `add-troubleshooting`, add a "Troubleshooting" section to `notes.md` with a few tips, commit it, switch to `main`, and merge. Verify the content appears on `main`.

2. **Practice the graph view:** After the merge, run `git log --oneline --graph --all`. Identify where your branch was created and where it merged back.

3. **Create a conflict on purpose:** Create two branches from `main`. On each branch, edit the same line of the same file differently. Merge one branch to `main` successfully, then try to merge the second. Resolve the conflict.

4. **Explore branch cleanup:** Run `git branch` to see all your branches. Delete any that have been merged using `git branch -d`.

5. **Try git switch:** If you've been using `git checkout`, try the equivalent commands with `git switch`. Switch between branches and create a new one with `git switch -c`.
