# Understanding History

By the end of this section, you'll know how to navigate your repository's history, compare versions, and track down exactly who changed what and when.

## Reading the Commit Log

Your commit history is more than a backup. It's a searchable record of every decision you've made. Learning to read it quickly is one of the most useful Git skills.

### Basic log

From your sandbox repository (or any Git repo), run:

```bash
git log
```

You'll see something like:

```
commit 8f3a2b1c9d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a
Author: Your Name <you@example.com>
Date:   Mon Jan 15 14:32:01 2024 -0800

    Add reboot procedure to server notes

commit a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0
Author: Your Name <you@example.com>
Date:   Mon Jan 15 14:30:45 2024 -0800

    Add server notes file
```

Each entry shows:

- **Commit hash**: The full 40-character unique identifier
- **Author**: Who made the commit (from your Git config)
- **Date**: When it was committed
- **Message**: What you wrote with `-m`

Press `q` to exit the log viewer.

### Compact view with --oneline

For a quick overview:

```bash
git log --oneline
```

```
8f3a2b1 Add reboot procedure to server notes
a1b2c3d Add server notes file
```

This shows just the abbreviated hash and the first line of each message. Much faster to scan when you have hundreds of commits.

### Filtering the log

Real repositories have thousands of commits. These flags help you find what you need:

```bash
# Last 5 commits
git log -5

# Commits from the past week
git log --since="1 week ago"

# Commits by a specific author
git log --author="lcrown"

# Commits that changed a specific file
git log -- notes.md

# Search commit messages for a keyword
git log --grep="reboot"
```

You can combine these. To find commits by "lcrown" in the last month that mention "ssl":

```bash
git log --author="lcrown" --since="1 month ago" --grep="ssl"
```

### Visualizing branches

Once you start working with branches (next section), this becomes invaluable:

```bash
git log --oneline --graph --all
```

The `--graph` flag draws ASCII art showing how branches split and merge. The `--all` flag includes all branches, not just the one you're on.

## Comparing Changes with Diff

You used `git diff` briefly in the last section. Let's go deeper. It's one of Git's most powerful tools for understanding what actually changed.

### Working directory vs staged

When you modify files, `git diff` shows unstaged changes:

```bash
echo "New line" >> notes.md
git diff
```

```diff
diff --git a/notes.md b/notes.md
index 3b18e51..f4a3c92 100644
--- a/notes.md
+++ b/notes.md
@@ -1,2 +1,3 @@
 # Server Notes
 Reboot procedure: sudo reboot
+New line
```

Lines starting with `+` are additions. Lines starting with `-` are deletions. The `@@` line tells you where in the file the change occurred.

After staging:

```bash
git add notes.md
git diff
```

You'll see nothing. `git diff` with no arguments only shows *unstaged* changes. To see what's staged:

```bash
git diff --staged
```

Now you see the changes again. This distinction matters when you're selectively staging parts of your work.

### Comparing commits

Compare any two commits by their hashes:

```bash
git diff a1b2c3d 8f3a2b1
```

Or compare against a previous commit using `~` notation:

```bash
# Current HEAD vs one commit ago
git diff HEAD~1

# Current HEAD vs three commits ago
git diff HEAD~3
```

`HEAD` always refers to your current commit. `HEAD~1` means "one commit before HEAD."

### Diff specific files

When a commit touches many files but you only care about one:

```bash
git diff HEAD~1 -- notes.md
```

The `--` separates the commit references from the file paths.

### Stat view for overview

When you want to see *which* files changed without the full diff:

```bash
git diff --stat HEAD~3
```

```
 notes.md     | 2 ++
 config.yaml  | 5 ++---
 2 files changed, 4 insertions(+), 3 deletions(-)
```

This shows files changed and a rough sense of how much.

## Finding Who Changed What

When something breaks (or when you want to understand why code looks the way it does), Git can tell you exactly who changed each line and when.

### git blame

This command shows the commit, author, and date for every line in a file:

```bash
git blame notes.md
```

```
a1b2c3d4 (Your Name 2024-01-15 14:30:45 -0800 1) # Server Notes
8f3a2b1c (Your Name 2024-01-15 14:32:01 -0800 2) Reboot procedure: sudo reboot
```

Each line shows:

- Abbreviated commit hash
- Author
- Date and time
- Line number
- The actual content

Despite the name, `git blame` isn't about assigning blame. It's about understanding context. When you find a strange line of code, blame tells you which commit introduced it so you can read the commit message and understand *why*.

### Blame a specific range

For long files, you can limit blame to specific lines:

```bash
# Lines 10-20 only
git blame -L 10,20 config.yaml
```

### git show

Once blame tells you which commit changed a line, `git show` gives you the full picture:

```bash
git show 8f3a2b1
```

This displays:

- The full commit message
- The author and date
- The complete diff of all changes in that commit

It's the natural follow-up to blame: "This line was changed in commit X" → "What else changed in commit X, and why?"

### git log for a file

To see the full history of changes to a specific file:

```bash
git log -- notes.md
```

Add `-p` to see the actual diffs in each commit:

```bash
git log -p -- notes.md
```

This is useful when you want to understand how a file evolved over time.

## Exercises

These exercises use your sandbox repository. If you don't have one, revisit the [Local Foundations](01-local-foundations.md) section to create it.

1. **Explore the log:** Run `git log`, `git log --oneline`, and `git log --oneline --graph`. If your sandbox only has a few commits, that's fine. The point is getting comfortable with the commands.

2. **Practice diff:** Modify `notes.md`, then run `git diff`. Stage the changes and run `git diff` again (notice it's empty). Run `git diff --staged` to see the staged changes. Commit when you're done.

3. **Use blame:** Run `git blame notes.md` and pick a commit hash from the output. Run `git show <hash>` to see the full commit.

4. **Search history:** Add a few more commits with different messages. Then use `git log --grep` to find commits containing a specific word from one of your messages.
