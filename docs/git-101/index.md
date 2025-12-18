# Git 101

You've probably been there: a config file that worked yesterday is broken today, and nobody knows what changed. Or you're afraid to touch a script because there's no way to undo it. Or you're emailing files back and forth with `-v2-final-FINAL.txt` in the name.

Git solves these problems. It tracks every change you make, who made it, and why—so you can experiment freely, collaborate without stepping on each other, and always get back to a known-good state.

## Why You Should Care

If you manage configs, scripts, documentation, or infrastructure-as-code, Git gives you:

- **A real undo button.** Made a bad change? Roll back to any previous version in seconds.
- **Fearless experimentation.** Try risky changes on a branch. If it works, merge it. If not, delete it. Your main copy stays safe.
- **Accountability without blame.** When something breaks at 2 AM, `git log` and `git blame` tell you exactly what changed and when—so you can fix it instead of guessing.
- **Collaboration that scales.** Multiple people can work on the same files without overwriting each other's work.

## Lessons

Work through these in order:

1. **Local Foundations** — Set up Git, create a repo, make your first commits
2. **Understanding History** — Read logs, compare changes, ignore files you don't want tracked
3. **Branching and Merging** — Work on isolated changes and combine them safely
4. **Working with Remotes** — Sync with shared repositories (push, pull, clone)
5. **Recovery and Confidence** — Fix mistakes and get unstuck

## Tips for Success

- **Use a scratch repo.** Create a throwaway folder to experiment in. Break things on purpose. Delete it and start over when you get stuck—that's the fastest way to learn.
- **Run `git status` constantly.** Before and after every operation. It tells you exactly what Git sees, and it's the quickest way to stay oriented.
- **Don't memorize—understand.** Git has a lot of commands, but they all revolve around a few core concepts. Once those click, the commands make sense.
