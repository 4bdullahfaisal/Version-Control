# Git Commands 

---

## What is Git?

Git is a **version control system** — it tracks changes to files over time.

**Analogy:** Git is like a "save point" in a video game — you can go back to any previous state.

---

## Git Configuration (One-time setup)

```bash
# Set your name and email (required for commits)
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"

# Check current configuration
git config --list
```

---

## Working with Repositories

```bash
# Clone an existing repository
git clone https://github.com/username/repo-name.git

# Initialize a new repository
git init

# Check status of your repo
git status
```
---

## Working with Remote Repositories

```bash
# Add a remote
git remote add origin https://github.com/username/repo.git

# Show remotes
git remote -v

# Remove a remote
git remote remove origin

# Push to remote
git push origin main

# Push a specific branch
git push origin feature-branch

# Pull from remote
git pull origin main

# Fetch changes (without merging)
git fetch
```
---

## Basic Git Workflow

```bash
# 1. Check which files changed
git status

# 2. Stage files (add to commit)
git add filename.txt      # Stage specific file
git add .                 # Stage all files

# 3. Commit changes (save snapshot)
git commit -m "Your message here"

# 4. Push to GitHub
git push origin main

# 5. Pull latest changes
git pull
```

---

## File Operations

```bash
# View changes in files
git diff                  # See unstaged changes
git diff --staged         # See staged changes

# Rename or move file
git mv old-name.txt new-name.txt

# Delete file
git rm filename.txt
```

---

## Viewing History

```bash
# Show commit history
git log

# Compact history (one line per commit)
git log --oneline

# Show history with graph
git log --graph --oneline --all

# Show commits by author
git log --author="Your Name"
```

---

## Branching

```bash
# List branches
git branch                # Local branches
git branch -a             # All branches (including remote)

# Create a new branch
git branch feature-branch

# Switch to a branch
git checkout feature-branch

# Create and switch in one command
git checkout -b new-branch

# Switch to main branch
git checkout main
```

---

## Merging

```bash
# Merge a branch into current branch
git merge feature-branch

# Merge with commit message
git merge feature-branch -m "Merged feature"

# Abort merge if conflict
git merge --abort
```

---

## Branch Management (Delete, Rename)

```bash
# Delete a local branch
git branch -d branch-name

# Force delete (if not merged)
git branch -D branch-name

# Delete a remote branch
git push origin --delete branch-name

# Rename current branch
git branch -m new-name
```

---

## Git Stash (Temporary save)

```bash
# Save current uncommitted work
git stash

# Save with message
git stash save "WIP: feature"

# List stashes
git stash list

# Apply latest stash (keep it in stash)
git stash apply

# Apply and remove from stash
git stash pop

# Drop a stash
git stash drop stash@{0}
```

---

## Undoing Changes

```bash
# Unstage a file (keep changes)
git reset HEAD filename.txt

# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Revert a commit (creates new commit)
git revert commit-hash

# Discard changes in working directory
git checkout -- filename.txt
```

---

## .gitignore

Create a `.gitignore` file to exclude files from being tracked:

```
# Ignore secrets
.pem
.env
*.key

# Ignore logs
*.log

# Ignore dependencies
node_modules/

# Ignore system files
.DS_Store
Thumbs.db
```

---

## Common Git Workflow (Daily Use)

```bash
# 1. Get latest changes
git pull

# 2. Create a new branch
git checkout -b feature/new-feature

# 3. Make changes and stage
git add .

# 4. Commit
git commit -m "Add new feature"

# 5. Push branch
git push origin feature/new-feature

# 6. Switch back to main
git checkout main

# 7. Merge (after pull request or review)
git merge feature/new-feature

# 8. Push main
git push
```

---

## Quick Error Fixes

| Error | Fix |
|-------|-----|
| "Permission denied" | Check SSH key or use HTTPS |
| "Merge conflict" | Manually resolve conflicts, then `git add .` and `git commit` |
| "Your branch is ahead" | `git push` |
| "Your branch is behind" | `git pull` |
| "Detached HEAD" | `git checkout main` or `git switch -` |

---

## Commands Summary Table

| Task | Command |
|------|---------|
| Clone repo | `git clone url` |
| Check status | `git status` |
| Stage all | `git add .` |
| Commit | `git commit -m "msg"` |
| Push | `git push origin main` |
| Pull | `git pull` |
| Create branch | `git branch name` |
| Switch branch | `git checkout name` |
| Merge branch | `git merge name` |
| View log | `git log --oneline` |
| Stash work | `git stash` |
| Pop stash | `git stash pop` |
| Undo last commit | `git reset --soft HEAD~1` |
| Discard all changes | `git reset --hard HEAD` |

---
