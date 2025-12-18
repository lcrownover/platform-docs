# Getting Started with Repos and Commits

Goal: set up Git, create a repo, and make clean commits.

## What You’ll Do
- Initialize a repo and set identity.
- Track files, stage changes, and commit with a clear message.
- Check status to confirm a clean working tree.

## Steps
1. **Create a workspace folder** (macOS/Linux):
   ```bash
   mkdir -p ~/scratch/git-101 && cd ~/scratch/git-101
   ```
   (Windows PowerShell):
   ```powershell
   New-Item -Type Directory "$HOME\\scratch\\git-101" -Force | Out-Null
   Set-Location "$HOME\\scratch\\git-101"
   ```
2. **Initialize Git**:
   ```bash
   git init
   git status
   ```
3. **Set identity (if not already global)**:
   ```bash
   git config user.name "Your Name"
   git config user.email "you@example.com"
   ```
4. **Add your first file**:
   ```bash
   echo "Hello Git 101" > notes.txt
   git status
   git add notes.txt
   git status
   git commit -m "Add hello notes"
   ```
5. **Verify clean state**:
   ```bash
   git status
   git log --oneline --graph
   ```

## Exercise
- Add a second file (`plan.txt`) with two bullet points, stage, and commit it with an imperative message.
- Modify `notes.txt`, run `git status`, and view the diff before and after staging:
  ```bash
  git diff
  git add notes.txt
  git diff --cached
  ```
- Confirm the working tree is clean (`git status`). Write down any surprises for group discussion.
