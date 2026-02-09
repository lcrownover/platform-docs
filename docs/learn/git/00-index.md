# Git 101

You've probably been there: a config file that worked yesterday is broken today, and nobody knows what changed. Or you're afraid to touch a script because there's no way to undo it. Or you're emailing files back and forth with `-v2-final-FINAL.txt` in the name.

Git solves these problems. It tracks every change you make, who made it, and why, so you can experiment freely, collaborate without stepping on each other, and always get back to a known-good state.

## Why You Should Care

If you manage configs, scripts, documentation, or infrastructure-as-code, Git gives you:

- **A real undo button.** Made a bad change? Roll back to any previous version in seconds.
- **Fearless experimentation.** Try risky changes on a branch. If it works, merge it. If not, delete it. Your main copy stays safe.
- **Accountability without blame.** When something breaks at 2 AM, `git log` and `git blame` tell you exactly what changed and when, so you can fix it instead of guessing.
- **Collaboration that scales.** Multiple people can work on the same files without overwriting each other's work.

## Lessons

Work through these in order:

1. **Local Foundations**: Set up Git, create a repo, make your first commits, ignore files you don't want tracked
2. **Understanding History**: Read logs, compare changes, find who changed what
3. **Branching and Merging**: Work on isolated changes and combine them safely
4. **Working with Remotes**: Sync with shared repositories (push, pull, clone)
5. **Collaboration**: Pull requests, code review, and team workflows
6. **Recovery and Confidence**: Fix mistakes and get unstuck
7. **Branching Strategies** (advanced): Git Flow, GitHub Flow, and trunk-based development

## Tips for Success

- **Use a scratch repo.** Create a throwaway folder to experiment in. Break things on purpose. Delete it and start over when you get stuck. That's the fastest way to learn.
- **Run `git status` constantly.** Before and after every operation. It tells you exactly what Git sees, and it's the quickest way to stay oriented.
- **Don't memorize, understand.** Git has a lot of commands, but they all revolve around a few core concepts. Once those click, the commands make sense.

## A Note on Git's Evolving Commands

Git has been around since 2005, and its command set has evolved over time. Some older commands do multiple unrelated things, while newer commands split those responsibilities apart.

For example, `git checkout` historically did two very different jobs: switching branches and restoring files. This was confusing, so Git introduced `git switch` (for branches) and `git restore` (for files) to make intentions clearer. Both the old and new commands work, but you'll see different ones in different tutorials.

Some common examples:

| Old way | New way | What it does |
|---------|---------|--------------|
| `git checkout branch-name` | `git switch branch-name` | Switch branches |
| `git checkout -b new-branch` | `git switch -c new-branch` | Create and switch to branch |
| `git checkout -- file.txt` | `git restore file.txt` | Discard uncommitted changes |
| `git reset HEAD file.txt` | `git restore --staged file.txt` | Unstage a file |

When you're reading Stack Overflow answers or older documentation, you'll encounter the older syntax. It still works fine. This guide generally uses the newer commands where they exist, but points out the old way so you recognize it in the wild.
