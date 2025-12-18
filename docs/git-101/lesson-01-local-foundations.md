# Local Foundations

By the end of this section, you'll have Git configured, a repository created, and your first commits under your belt.

## Configure Your Identity

Before you make any commits, Git needs to know who you are. This information appears in every commit you make—it's how your team knows who changed what.

If you haven't already, complete the [Initial Configuration](../tools/git.md#initial-configuration) steps in the Git tool guide. This sets your name and email globally so you don't have to configure it for each repository.

!!! note "Global vs Local Config"
    The `--global` flag sets these values for all repositories on your machine. You can override them per-repository by running the same commands without `--global` from inside a repo.

## Create Your First Repository

Let's create a scratch repository you can experiment with freely.

```bash
mkdir -p ~/scratch/git-sandbox
cd ~/scratch/git-sandbox
```

Now initialize Git:

```bash
git init
```

You'll see: `Initialized empty Git repository in .../git-sandbox/.git/`

That `.git` folder is where Git stores everything—all your history, branches, and metadata. You'll never need to touch it directly, but knowing it exists helps demystify what Git is doing.

Run your first status check:

```bash
git status
```

You'll see something like:

```
On branch main
No commits yet
nothing to commit (create/copy some files and use "git add" to track)
```

This tells you three things:

1. You're on a branch called `main` (we'll cover branches later)
2. There are no commits yet
3. There's nothing to commit because the folder is empty

## The Stage and Commit Workflow

Here's where Git differs from simple backup systems. Git doesn't automatically track every file or save every change. You explicitly tell it what to record. This happens in two steps:

1. **Stage** — Select which changes you want to include in your next commit
2. **Commit** — Save those staged changes as a permanent snapshot

Think of it like packing a box before shipping it. You don't throw everything in at once—you choose what goes in the box (staging), then seal and label it (commit).

### Create a file and check status

```bash
echo "# Server Notes" > notes.md
git status
```

Git now shows `notes.md` as an "untracked file." Git sees it exists but isn't tracking changes to it yet.

### Stage the file

```bash
git add notes.md
git status
```

Now `notes.md` appears under "Changes to be committed." It's staged—ready to be included in your next commit.

!!! tip "Stage multiple files at once"
    `git add .` stages all changes in the current directory. Useful, but be careful—make sure you're not staging files you don't want (like secrets or build artifacts). Always run `git status` first.

### Commit the staged changes

```bash
git commit -m "Add server notes file"
```

Done. You've created your first commit. Git responds with a summary:

```
[main (root-commit) a1b2c3d] Add server notes file
 1 file changed, 1 insertion(+)
 create mode 100644 notes.md
```

The string `a1b2c3d` is an abbreviated commit hash—a unique identifier for this exact snapshot. Every commit gets one.

### The full cycle

Let's do it again to reinforce the pattern. Edit the file:

```bash
echo "Reboot procedure: sudo reboot" >> notes.md
```

Check what changed:

```bash
git status
```

Git shows `notes.md` as "modified." It's already tracked, so Git notices the change, but that change isn't staged yet.

See exactly what changed:

```bash
git diff
```

This shows the lines added (prefixed with `+`) and removed (prefixed with `-`). Get comfortable with `git diff`—it's your preview before committing.

Stage and commit:

```bash
git add notes.md
git commit -m "Add reboot procedure to server notes"
```

### Why two steps?

The staging area lets you craft clean, logical commits even when your working directory is messy. You might have three files changed, but only two are related to the same task. Stage those two, commit them, then stage and commit the third separately.

Clean commits make history easier to read and problems easier to diagnose. When something breaks, you want to find "Add SSL config" in your history—not "Various changes and fixes."

## Writing Good Commit Messages

Commit messages are for your future self and your teammates. Write them like you'll be the one debugging this at 2 AM six months from now.

### Use imperative mood

Write messages as commands: "Add config file" not "Added config file" or "Adding config file."

This matches Git's own conventions (e.g., "Merge branch 'feature'") and reads naturally when you look at history: "If I apply this commit, it will **Add config file**."

**Good:**

- `Add SSL certificate rotation script`
- `Fix incorrect timeout in backup job`
- `Remove deprecated API endpoint`

**Bad:**

- `Added some stuff`
- `WIP`
- `asdfasdf`
- `Fixed it`

### Be specific

Your message should explain what changed and, briefly, why. Someone reading the log should understand the intent without reading the code.

| Instead of... | Write... |
|--------------|----------|
| `Update config` | `Increase nginx worker connections to 4096` |
| `Fix bug` | `Fix race condition in session cleanup` |
| `Changes` | `Add retry logic to LDAP connection` |

### Keep the first line short

The first line should be under 50 characters—this is what appears in logs, GitHub, and most Git tools. If you need more detail, add a blank line and then a longer description:

```bash
git commit -m "Add retry logic to LDAP connection

Previous implementation failed silently on network timeouts.
Now retries 3 times with exponential backoff before failing."
```

## Exercises

1. **Practice the cycle:** Create two more files in your sandbox, stage them separately, and commit each with a descriptive message. Run `git status` before and after each command to see how the state changes.

2. **Selective staging:** Modify two files, but only stage and commit one. Verify the other file still shows as modified after the commit.

3. **View your history:** Run `git log` to see your commits. Try `git log --oneline` for a compact view. We'll dig deeper into history in the next section.
