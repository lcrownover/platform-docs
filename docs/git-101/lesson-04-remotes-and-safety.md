# Remotes and Daily Safety Habits

Goal: connect to a remote, push/pull changes, and practice safe daily commands.

## What You’ll Do
- Add a remote and sync branches.
- Push and pull with fast-forward expectations.
- Use status/diff/log to stay safe; avoid surprises.

## Steps
1. **Add a remote** (replace with your own URL or a throwaway repo):
   ```bash
   git remote add origin https://github.com/<your-username>/git-101-demo.git
   git remote -v
   ```
2. **Push main**:
   ```bash
   git push -u origin main
   ```
   If using SSH, ensure your key/agent is configured before pushing.
3. **Simulate pulling new work**:
   - Make a small change on `main`, commit it, push again.
   - Ask a partner (or yourself in another clone) to add a line to `plan.txt`, commit, and push.
   - From your original clone:
     ```bash
     git pull --ff-only
     git log --oneline --graph --decorate -5
     ```
4. **Safety checks to keep using daily**:
   ```bash
   git status            # before and after edits
   git diff              # before staging
   git diff --cached     # before committing
   git log --oneline -5  # after pulling or merging
   ```

## Exercise
- Add a new branch, push it, and open a draft PR in your hosting platform (optional but recommended).
- Practice recovering a local change:
  ```bash
  echo "Oops" >> notes.txt
  git status
  git restore notes.txt   # or 'git checkout -- notes.txt' if older Git
  git status
  ```
- Summarize the three commands you’ll run before every commit (`status`, `diff`, `diff --cached`).
