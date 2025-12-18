# Branching, Merging, and Resolving Conflicts

Goal: create a branch, make changes, merge back, and handle a simple conflict.

## What You’ll Do
- Create and switch branches.
- Merge a feature branch into main.
- Resolve a controlled merge conflict.

## Steps
1. **Create a feature branch**:
   ```bash
   git switch -c feature/add-plan
   echo "Learning goals:" >> plan.txt
   echo "- Understand branching" >> plan.txt
   git add plan.txt
   git commit -m "Add branching goals to plan"
   ```
2. **Merge to main**:
   ```bash
   git switch main
   git merge feature/add-plan
   git log --oneline --graph --decorate -5
   ```
3. **Create a conflict (controlled)**:
   - On `main`, edit `notes.txt` and change the first line to `Hello from main`.
   - Commit it: `git commit -am "Update greeting on main"`.
   - Switch to a new branch and change the same line differently:
     ```bash
     git switch -c feature/conflict-demo
     echo "Hello from the feature branch" > notes.txt
     git commit -am "Update greeting on feature"
     ```
   - Merge back to main to trigger conflict:
     ```bash
     git switch main
     git merge feature/conflict-demo
     ```
4. **Resolve the conflict**:
   - Open `notes.txt`, choose the preferred line, remove conflict markers.
   - Stage and complete the merge:
     ```bash
     git add notes.txt
     git commit -m "Resolve greeting conflict"
     git log --oneline --graph --decorate -5
     ```

## Exercise
- Create another short-lived branch, change `plan.txt`, merge it, and ensure a fast-forward merge happens.
- Run `git branch -a` to list branches and delete merged feature branches:
  ```bash
  git branch -d feature/add-plan
  git branch -d feature/conflict-demo
  ```
- Write down one takeaway about conflict resolution to share with the group.
