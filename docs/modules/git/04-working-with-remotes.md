# Working with Remotes

By the end of this section, you'll know how to sync your local repository with shared repositories using clone, push, pull, and fetch.

## What Is a Remote?

So far, everything you've done has been local. Your repository lives on your machine, and no one else can see it. A remote is a copy of your repository hosted somewhere else (usually a server like GitHub, GitLab, or Bitbucket).

Remotes enable:

- **Backup.** Your code exists somewhere other than your laptop.
- **Collaboration.** Others can access the same repository.
- **Deployment.** Many workflows deploy directly from a remote repository.

When you clone a repository, Git automatically names the remote `origin`. You can have multiple remotes, but most workflows only use one.

### See your remotes

```bash
git remote -v
```

If you created your repository locally with `git init`, you won't have any remotes yet. If you cloned from somewhere, you'll see:

```
origin  https://github.com/username/repo.git (fetch)
origin  https://github.com/username/repo.git (push)
```

### Add a remote

If you started locally and want to connect to a remote:

```bash
git remote add origin https://github.com/username/repo.git
```

Now `origin` points to that URL. You can push your local commits there.

## Cloning a Repository

Cloning downloads a complete copy of a remote repository to your machine, including all history.

### Clone with SSH (recommended)

```bash
git clone git@github.com:username/repo.git
```

SSH is the preferred method for regular Git use. Once you've set up SSH keys with your Git host, you never need to enter credentials. Pushes and pulls just work.

If you haven't set up SSH keys yet, it's worth the few minutes of setup. GitHub, GitLab, and Bitbucket all have guides for adding your SSH key to your account.

!!! tip "Set up SSH keys once, benefit forever"
    SSH keys are more secure than passwords and eliminate credential prompts entirely. If you're going to use Git regularly, take the time to set this up. Your future self will thank you.

### Clone with HTTPS

```bash
git clone https://github.com/username/repo.git
```

HTTPS works without any setup, which makes it fine for quick one-off clones. However, you'll need to enter credentials (or configure a credential helper) for push operations. For repositories you'll work with frequently, SSH is worth the initial setup.

When you clone, Git automatically:

- Downloads all commits and branches
- Sets up `origin` pointing to the URL you cloned from
- Checks out the default branch (usually `main`)

### Clone into a specific folder

```bash
git clone https://github.com/username/repo.git my-folder-name
```

### Clone a specific branch

By default, clone checks out the default branch. To start on a different branch:

```bash
git clone -b develop https://github.com/username/repo.git
```

## Push, Pull, and Fetch

These three commands handle synchronization between your local repository and the remote.

### git push: send your commits to the remote

After committing locally, push sends those commits to the remote:

```bash
git push origin main
```

This pushes your `main` branch to the `origin` remote. If you're on `main` and it's configured to track `origin/main` (which clone sets up automatically), you can just run:

```bash
git push
```

#### First push of a new branch

If you created a branch locally and want to push it to the remote for the first time:

```bash
git push -u origin my-new-branch
```

The `-u` flag (short for `--set-upstream`) tells Git to track this remote branch. After that, plain `git push` and `git pull` work without specifying the branch.

#### Push rejected?

If someone else pushed commits since you last pulled, Git rejects your push:

```
! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'origin'
hint: Updates were rejected because the remote contains work that you do not
hint: have locally.
```

This is Git protecting you from overwriting others' work. Pull first, then push.

### git pull: get commits from the remote and merge

Pull downloads new commits from the remote and merges them into your current branch:

```bash
git pull
```

This is actually two operations combined:

1. `git fetch` (download new commits)
2. `git merge` (merge them into your branch)

If there are no conflicts, Git merges automatically. If there are conflicts, you'll need to resolve them just like any other merge (see [Branching and Merging](03-branching-and-merging.md#handling-merge-conflicts)).

#### Pull with rebase

Some teams prefer rebasing instead of merging when pulling:

```bash
git pull --rebase
```

Instead of creating a merge commit, this replays your local commits on top of the remote commits. The result is a linear history without merge commits cluttering your log every time you sync.

**The tradeoff:** Rebase rewrites your local commit hashes. This is fine for commits that only exist on your machine, but you should understand what's happening. Some developers find the cleaner history worth it; others prefer seeing the true merge history.

If your team uses rebase-based pulls, you can set it as the default:

```bash
git config --global pull.rebase true
```

With this set, `git pull` automatically rebases instead of merging. You can override it with `git pull --no-rebase` when needed.

!!! note "Follow your team's convention"
    Whether to use merge or rebase when pulling is a team decision. If you're joining an existing project, look at the commit history to see what convention they follow, or ask.

### git fetch: download without merging

Fetch downloads new commits from the remote but doesn't merge them:

```bash
git fetch origin
```

This updates your knowledge of what's on the remote (stored in `origin/main`, `origin/develop`, etc.) without changing your working directory. Useful when you want to see what's changed before deciding what to do.

After fetching, you can:

```bash
# See what's new on the remote
git log main..origin/main

# See the diff
git diff main origin/main

# Merge when ready
git merge origin/main
```

Fetch is the safe way to check what's happening on the remote without committing to a merge.

In practice, most people just use `git pull` for daily work. Fetch is useful when you want to inspect changes before merging, or when you're troubleshooting a conflict and want to understand what's on the remote first.

## Working with Branches on Remotes

### See all branches (local and remote)

```bash
git branch -a
```

Remote branches appear as `remotes/origin/branch-name`.

### Check out a remote branch

If someone else created a branch and pushed it, fetch first, then check it out:

```bash
git fetch origin
git checkout feature-branch
```

Git automatically creates a local branch that tracks the remote one.

### Delete a remote branch

After merging a branch, clean it up on the remote:

```bash
git push origin --delete feature-branch
```

## Common Scenarios

### "I cloned a repo but want to push to my own copy"

You cloned someone else's repository but want your own remote (common when starting from a template):

```bash
# Remove the original remote
git remote remove origin

# Add your own
git remote add origin https://github.com/your-username/your-repo.git

# Push
git push -u origin main
```

### "I need to see what changed on the remote"

```bash
git fetch origin
git log HEAD..origin/main --oneline
```

This shows commits on the remote that you don't have locally.

### "Someone else's push broke my pull"

If `git pull` fails because of conflicts with changes you weren't expecting:

```bash
# Abort the merge
git merge --abort

# Fetch and look at what changed
git fetch origin
git log main..origin/main
git diff main origin/main

# When you understand the changes, try the merge again
git merge origin/main
```

## Exercises

1. **Clone a repository:** Find a public repository on GitHub and clone it. Run `git remote -v` to see the remote configuration. Run `git log --oneline -10` to see recent history.

2. **Practice the pull workflow:** If you have a repository with a remote, make a change on GitHub's web interface (edit a README), then pull the change locally.

3. **Explore fetch:** Run `git fetch origin`, then compare your local branch to the remote with `git log main..origin/main`. See what's different before merging.

4. **Push a new branch:** Create a local branch, make a commit, and push it with `git push -u origin branch-name`. Verify it appears on the remote.

5. **Clean up a remote branch:** After pushing a test branch, delete it with `git push origin --delete branch-name`. Verify it's gone on the remote.

## Next Steps

Now that you know the mechanics of working with remotes, see [Collaboration](05-collaboration.md) for team workflows, pull requests, and best practices for working with others.
