# Viewing History and Ignoring Noise

Goal: inspect changes confidently and keep noise out of your repo.

## What You’ll Do
- Read history with short, useful logs.
- Compare working, staged, and committed changes.
- Create a `.gitignore` and confirm it works.

## Steps
1. **Inspect history**:
   ```bash
   git log --oneline --graph --decorate -5
   git show HEAD
   ```
2. **Compare changes**:
   ```bash
   echo "Temporary note" > scratch.log
   git diff              # working tree vs. HEAD
   git add notes.txt
   git diff --cached     # staged vs. HEAD
   git status
   ```
3. **Ignore temp files**:
   ```bash
   echo "scratch.log" >> .gitignore
   echo "*.tmp" >> .gitignore
   git status --ignored
   git add .gitignore
   git commit -m "Add ignore rules for scratch files"
   ```
4. **Clean up** (optional but recommended):
   ```bash
   rm -f scratch.log
   git status
   ```

## Exercise
- Add a directory `logs/` with a dummy file, then update `.gitignore` to exclude the directory and confirm with `git status --ignored`.
- Run `git show HEAD~1` to inspect your earlier commit and describe what changed.
- Capture a one-line log with a graph showing all commits so far:
  ```bash
  git log --oneline --graph --all
  ```
