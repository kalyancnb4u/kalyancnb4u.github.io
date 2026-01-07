---
title: "Python Mastery Part 6: Version Control & Collaboration"
date: 2024-12-07 00:00:00 +0530
categories: [Python, Language Mastery]
tags: [Python, Git, GitHub, GitLab, Version Control, Collaboration, Code Review]
---
## Table of Contents

- [Introduction](#introduction)
- [6.1 Git Fundamentals](#61-git-fundamentals)
  - [Git Basics](#git-basics)
  - [Branching and Merging](#branching-and-merging)
  - [Git Best Practices](#git-best-practices)
  - [Branching Strategies](#branching-strategies)
  - [Frequently Asked Questions](#frequently-asked-questions)
  - [Interview Questions](#interview-questions)
- [6.2 Advanced Git](#62-advanced-git)
  - [Rebasing](#rebasing)
  - [Cherry-picking](#cherry-picking)
  - [Git Hooks](#git-hooks)
  - [Submodules](#submodules)
  - [Frequently Asked Questions](#frequently-asked-questions-1)
  - [Interview Questions](#interview-questions-1)
- [6.3 Collaborative Development](#63-collaborative-development)
  - [Pull Request Workflow](#pull-request-workflow)
  - [Code Review Best Practices](#code-review-best-practices)
  - [Pair Programming](#pair-programming)
  - [Frequently Asked Questions](#frequently-asked-questions-2)
- [6.4 Git Hosting Platforms](#64-git-hosting-platforms)
  - [GitHub Features](#github-features)
  - [GitLab Features](#gitlab-features)
  - [Frequently Asked Questions](#frequently-asked-questions-3)

---

# Complete Python Mastery Part 6: Version Control & Collaboration

## Introduction

Welcome to **Part 6** of the Complete Python Mastery series. Version control and collaboration are essential skills for professional software development.

**What You'll Learn in Part 6:**

This post covers version control and collaborative development:

- **Git fundamentals**: Repository management, branching, merging
- **Branching strategies**: Git Flow, GitHub Flow, trunk-based development
- **Advanced Git**: Rebasing, cherry-picking, hooks, submodules
- **Collaboration**: Pull requests, code reviews, pair programming
- **Platforms**: GitHub, GitLab features and workflows

**Why This Matters:**

Effective version control and collaboration enable:

- Team coordination and productivity
- Code quality through reviews
- Safe experimentation with branches
- Clear project history
- Rollback capabilities

**Prerequisites:**

- Part 1: Python Fundamentals
- Basic command line knowledge
- Git installed on your system

Let's master version control and collaboration! 🚀

---

## 6.1 Git Fundamentals

**SDLC Phase:** All Phases

**Relevant For:**

- [X] Requirements gathering (documentation)
- [X] System design (design docs)
- [X] Development (code)
- [X] Testing (test code)
- [X] Deployment (deployment scripts)
- [X] Maintenance (all changes)

### Git Basics

```bash
"""
GIT: Distributed version control system

Core Concepts:
- Repository (repo): Project directory with .git folder
- Commit: Snapshot of changes
- Branch: Independent line of development
- Remote: Repository on another computer
- Clone: Copy of remote repository
"""

# Initialize repository
git init myproject
cd myproject

# Check status
git status

# Configure Git (first time)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# View configuration
git config --list

# Add files to staging area
git add filename.py          # Add specific file
git add .                    # Add all files
git add *.py                 # Add all Python files
git add -p                   # Add interactively (review changes)

# Commit changes
git commit -m "Add user authentication"

# ✅ GOOD commit message
git commit -m "Fix: Prevent duplicate user registration

- Add unique constraint on email field
- Update registration endpoint validation
- Add tests for duplicate email handling

Closes #123"

# View commit history
git log                      # Full history
git log --oneline           # Compact view
git log --graph --oneline   # Visual graph
git log -p                  # Show diff in each commit
git log --author="John"     # Filter by author
git log --since="2 weeks ago"  # Filter by date

# View specific commit
git show abc123             # Show commit details
git show HEAD               # Show latest commit
git show HEAD~1             # Show previous commit

# View changes
git diff                    # Unstaged changes
git diff --staged           # Staged changes
git diff HEAD               # All changes vs last commit
git diff main..feature      # Compare branches

# Undo changes
git checkout -- filename.py  # Discard unstaged changes
git reset HEAD filename.py   # Unstage file
git reset --soft HEAD~1      # Undo last commit (keep changes)
git reset --hard HEAD~1      # Undo last commit (discard changes)

# Remove files
git rm filename.py          # Remove file
git rm --cached filename.py # Remove from Git, keep locally
```

```python
# .gitignore for Python projects
"""
.gitignore: Files to exclude from version control
"""

# Byte-compiled / optimized / DLL files
__pycache__/
*.py[cod]
*$py.class
*.so

# Distribution / packaging
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual environments
venv/
env/
ENV/
.venv

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# Testing
.pytest_cache/
.coverage
htmlcov/
.tox/

# Environment
.env
.env.local
.env.*.local

# Database
*.db
*.sqlite
*.sqlite3

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db

# Project specific
secrets/
local_settings.py
```

### Branching and Merging

```bash
"""
BRANCHES: Parallel development lines

Benefits:
- Isolated feature development
- Experimentation without risk
- Parallel work on multiple features
"""

# List branches
git branch                  # Local branches
git branch -r               # Remote branches
git branch -a               # All branches

# Create branch
git branch feature-auth     # Create branch
git checkout feature-auth   # Switch to branch

# Create and switch (shortcut)
git checkout -b feature-auth

# Newer syntax (Git 2.23+)
git switch feature-auth     # Switch branch
git switch -c feature-auth  # Create and switch

# Rename branch
git branch -m old-name new-name

# Delete branch
git branch -d feature-auth  # Safe delete (merged only)
git branch -D feature-auth  # Force delete

# Merge branches
git checkout main
git merge feature-auth      # Merge feature into main

# Merge strategies

# 1. Fast-forward merge (no merge commit)
# main: A---B
# feature:    C---D
# After merge: A---B---C---D (linear history)
git merge feature-auth

# 2. Three-way merge (creates merge commit)
# main: A---B---E
# feature:    C---D
# After merge: A---B---E---M (M is merge commit)
git merge --no-ff feature-auth  # Always create merge commit

# 3. Squash merge (combine all commits)
# main: A---B
# feature:    C---D---E
# After merge: A---B---F (F contains C+D+E changes)
git merge --squash feature-auth
git commit -m "Add authentication feature"

# Resolve merge conflicts
# 1. Merge shows conflicts
git merge feature-auth
# Auto-merging file.py
# CONFLICT (content): Merge conflict in file.py

# 2. View conflict
cat file.py
"""
<<<<<<< HEAD
print("main branch code")
=======
print("feature branch code")
>>>>>>> feature-auth
"""

# 3. Edit file to resolve
"""
print("resolved code")
"""

# 4. Mark as resolved
git add file.py
git commit -m "Merge feature-auth"

# Abort merge if needed
git merge --abort
```

### Git Best Practices

```bash
"""
GIT BEST PRACTICES

1. Commit often, push regularly
2. Write meaningful commit messages
3. Keep commits atomic (one logical change)
4. Pull before you push
5. Use branches for features
6. Review before merging
"""

# 1. Atomic commits (one logical change per commit)

# ❌ BAD: Multiple unrelated changes
git add .
git commit -m "Various fixes and features"

# ✅ GOOD: Separate commits
git add user_auth.py tests/test_auth.py
git commit -m "feat: Add user authentication"

git add database.py
git commit -m "fix: Fix database connection leak"

# 2. Commit message conventions (Conventional Commits)

# Format: <type>(<scope>): <subject>
#
# <body>
#
# <footer>

# Types:
# feat: New feature
# fix: Bug fix
# docs: Documentation changes
# style: Formatting, missing semicolons, etc.
# refactor: Code restructuring
# test: Adding tests
# chore: Maintenance tasks

# Examples:
git commit -m "feat(auth): Add JWT token authentication"

git commit -m "fix(api): Handle timeout errors in user endpoint

- Add retry logic with exponential backoff
- Increase timeout to 30 seconds
- Add logging for timeout events

Closes #456"

git commit -m "docs: Update API documentation for v2.0"

git commit -m "refactor: Extract email validation to separate function"

git commit -m "test: Add integration tests for payment flow"

# 3. Pull before push (avoid conflicts)
git pull origin main        # Get latest changes
git push origin main        # Push your changes

# 4. Commit message template
# .gitmessage
"""
# <type>(<scope>): <subject>
# 
# <body>
# 
# <footer>

# Type: feat, fix, docs, style, refactor, test, chore
# Scope: Component being changed
# Subject: Short description (50 chars max)
# Body: Detailed explanation (wrap at 72 chars)
# Footer: Issue references
"""

# Set template
git config --global commit.template ~/.gitmessage

# 5. Amend last commit (before pushing)
git add forgotten_file.py
git commit --amend --no-edit  # Add to last commit

git commit --amend -m "New message"  # Change message

# ⚠️ Never amend pushed commits! (rewrites history)
```

### Branching Strategies

```bash
"""
BRANCHING STRATEGIES

1. Git Flow: Feature branches + develop + main
2. GitHub Flow: Feature branches + main (simpler)
3. Trunk-Based Development: Short-lived branches
"""

# 1. GIT FLOW (complex projects)
"""
Branches:
- main: Production-ready code
- develop: Integration branch
- feature/*: Feature development
- release/*: Release preparation
- hotfix/*: Emergency fixes

Flow:
1. Create feature branch from develop
2. Work on feature
3. Merge back to develop
4. Create release branch
5. Test and fix in release
6. Merge to main and develop
7. Tag release
"""

# Start feature
git checkout develop
git checkout -b feature/user-profile

# Work on feature
git add .
git commit -m "feat: Add user profile page"

# Finish feature
git checkout develop
git merge --no-ff feature/user-profile
git branch -d feature/user-profile
git push origin develop

# Start release
git checkout develop
git checkout -b release/1.0.0

# Prepare release
git commit -m "chore: Bump version to 1.0.0"
git commit -m "docs: Update changelog"

# Finish release
git checkout main
git merge --no-ff release/1.0.0
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

git checkout develop
git merge --no-ff release/1.0.0
git branch -d release/1.0.0

# Hotfix
git checkout main
git checkout -b hotfix/security-patch

# Fix issue
git commit -m "fix: Patch SQL injection vulnerability"

# Finish hotfix
git checkout main
git merge --no-ff hotfix/security-patch
git tag -a v1.0.1 -m "Hotfix 1.0.1"

git checkout develop
git merge --no-ff hotfix/security-patch
git branch -d hotfix/security-patch

# 2. GITHUB FLOW (simpler, for CI/CD)
"""
Branches:
- main: Always deployable
- feature/*: Feature branches

Flow:
1. Create feature branch from main
2. Work on feature
3. Open pull request
4. Review and discuss
5. Merge to main
6. Deploy from main
"""

# Create feature
git checkout main
git pull origin main
git checkout -b feature/add-payment

# Work and push
git add .
git commit -m "feat: Add Stripe payment integration"
git push origin feature/add-payment

# Open PR on GitHub, then merge via web interface

# 3. TRUNK-BASED DEVELOPMENT (continuous integration)
"""
Branches:
- main (trunk): Single branch
- Short-lived feature branches (hours, not days)

Flow:
1. Create short-lived branch
2. Small, frequent commits
3. Merge quickly to main
4. Use feature flags for incomplete features
"""

# Short-lived feature
git checkout main
git pull origin main
git checkout -b quick-fix

# Small changes
git add .
git commit -m "fix: Correct date formatting"
git push origin quick-fix

# Merge quickly (same day)
git checkout main
git merge quick-fix
git push origin main
git branch -d quick-fix

# For incomplete features, use feature flags
```

```python
# Feature flags example
import os

FEATURE_NEW_UI = os.getenv("FEATURE_NEW_UI", "false").lower() == "true"

def render_page():
    if FEATURE_NEW_UI:
        return render_new_ui()  # New code in main
    else:
        return render_old_ui()  # Stable code
```

### Frequently Asked Questions

**Q1: When should I commit? How often?**

**A:**

```bash
"""
COMMIT FREQUENCY BEST PRACTICES

Commit when:
✅ You complete a logical unit of work
✅ All tests pass
✅ Code compiles/runs without errors
✅ You're about to try something risky
✅ End of work session (even if incomplete)

Commit size:
✅ One feature or one bug fix per commit
✅ Small enough to review (< 400 lines changed)
✅ Large enough to be meaningful

Examples:

✅ GOOD commits:
git commit -m "feat: Add user authentication"
git commit -m "test: Add tests for auth module"
git commit -m "fix: Handle null email in signup"
git commit -m "refactor: Extract validation logic"

❌ BAD commits:
git commit -m "WIP"  # Too vague
git commit -m "Fixed stuff"  # Too vague
# One commit with 20 different changes  # Too large
# Commits every 5 minutes  # Too frequent

Golden rule: Commit when you can write a clear, descriptive message

Workflow:
1. Write code for one feature/fix
2. Run tests
3. If tests pass → commit
4. Repeat
"""

# WIP (Work In Progress) commits are OK for personal branches
git add .
git commit -m "WIP: Implementing OAuth flow"

# But clean them up before merging (squash or rebase)
git rebase -i HEAD~5  # Interactive rebase to clean history
```

**Why This Matters:** Good commit habits make history useful for debugging and collaboration.

---

**Q2: Should I merge or rebase?**

**A:**

```bash
"""
MERGE vs REBASE

Use MERGE when:
✅ Working on shared/public branches
✅ Want to preserve complete history
✅ Merging long-lived feature branches
✅ Collaborating with team on same branch

Use REBASE when:
✅ Updating your feature branch with main
✅ Cleaning up local commits before PR
✅ Want linear history
✅ Working alone on feature branch

Never rebase:
❌ Public branches (main, develop)
❌ Commits others are building on
❌ Commits already pushed to shared branch
"""

# MERGE example (preserves history)
git checkout feature
git merge main
# Creates merge commit
# History shows when main was merged

# REBASE example (linear history)
git checkout feature
git rebase main
# Replays your commits on top of main
# History looks linear, cleaner

# Visual comparison:

# Before:
# main:    A---B---C
# feature:   D---E

# After MERGE:
# main:    A---B---C-------M (merge commit)
#                   \     /
# feature:           D---E

# After REBASE:
# main:    A---B---C
# feature:           D'--E' (commits moved)

# Best practice: Rebase locally, merge in PR
git checkout feature
git rebase main           # Clean up your branch
git push -f origin feature  # Force push (only OK on your branch)
# Then merge via PR (creates merge commit)

# Golden rule:
# "Rebase commits you own, merge commits others own"
```

**Why This Matters:** Wrong choice can rewrite history and break team collaboration.

---

### Interview Questions

**Question 1: Explain the difference between git reset, git revert, and git checkout for undoing changes.**

**Difficulty:** Mid-Level

**SDLC Relevance:** Development, Maintenance

**Answer:**

```bash
"""
UNDOING CHANGES IN GIT

git reset: Move branch pointer (rewrites history)
git revert: Create new commit that undoes changes (safe)
git checkout: Switch branches or restore files

1. git reset (local changes only!)
"""

# Scenario: Undo last 2 commits locally
# Commits: A---B---C---D---E (HEAD on E)

git reset --soft HEAD~2
# Result: A---B---C---D---E
#                      ↑ HEAD moved here
# Changes from D and E are staged

git reset --mixed HEAD~2  # Default
# Result: A---B---C---D---E
#                ↑ HEAD
# Changes from D and E are unstaged

git reset --hard HEAD~2
# Result: A---B---C  # D and E deleted!
#                ↑ HEAD
# Changes from D and E are lost

# ⚠️ Never reset commits that are pushed!

"""
2. git revert (safe for public branches)
"""

# Scenario: Undo commit D (already pushed)
# Commits: A---B---C---D---E (HEAD on E)

git revert D
# Result: A---B---C---D---E---D' (D' undoes D)
#                            ↑ HEAD

# Safe because it creates new commit
# Doesn't rewrite history
# Can be pushed to shared branches

"""
3. git checkout (read files from history)
"""

# Restore single file from last commit
git checkout HEAD -- file.py

# Restore file from specific commit
git checkout abc123 -- file.py

# Switch branches
git checkout main

# Create and switch
git checkout -b new-branch

"""
Decision tree:

Need to undo local commits not pushed?
→ Use git reset

Need to undo pushed commits?
→ Use git revert

Need to restore a file?
→ Use git checkout

Need to discard all local changes?
→ Use git reset --hard HEAD
"""

# Real-world examples:

# 1. Committed sensitive data (not pushed)
git reset --hard HEAD~1  # Delete commit
git commit -m "Add config (without secrets)"

# 2. Bug introduced in deployed code
git revert abc123  # Revert bad commit
git push origin main  # Deploy fix

# 3. Accidentally modified file
git checkout HEAD -- file.py  # Restore file
```

**Key Points:**

- `reset`: Moves HEAD, rewrites history (local only!)
- `revert`: Creates new commit, safe for shared branches
- `checkout`: Restores files or switches branches
- Never reset/rebase public commits
- Use revert for undoing pushed changes

---

## 6.2 Advanced Git

**SDLC Phase:** Development, Maintenance

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (code management)
- [ ] Testing
- [ ] Deployment
- [X] Maintenance (history management)

### Rebasing

```bash
"""
REBASING: Rewrite commit history

Benefits:
✅ Clean, linear history
✅ Easier to review changes
✅ Better for git bisect

Risks:
❌ Rewrites history (don't rebase public branches!)
❌ Can cause conflicts
"""

# Interactive rebase (clean up commits)
git rebase -i HEAD~5  # Rebase last 5 commits

# Interactive rebase opens editor:
"""
pick abc123 feat: Add login
pick def456 fix: Typo
pick ghi789 fix: Another typo
pick jkl012 feat: Add logout
pick mno345 test: Add tests

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, merge into previous
# f, fixup = like squash, discard commit message
# d, drop = remove commit
"""

# Example: Clean up messy commits
"""
pick abc123 feat: Add login
fixup def456 fix: Typo  # Merge into previous
fixup ghi789 fix: Another typo  # Merge into previous
pick jkl012 feat: Add logout
squash mno345 test: Add tests  # Merge and edit message
"""

# Result: 2 clean commits instead of 5

# Rebase onto main (update feature branch)
git checkout feature
git rebase main

# If conflicts occur:
# 1. Resolve conflicts in files
# 2. Stage resolved files
git add resolved_file.py

# 3. Continue rebase
git rebase --continue

# Or abort
git rebase --abort

# Rebase with autosquash (auto-fixup commits)
git commit --fixup=abc123  # Mark commit as fixup for abc123
git rebase -i --autosquash HEAD~10  # Auto-squash fixups

# Force push after rebase (only on feature branches!)
git push -f origin feature
```

### Cherry-picking

```bash
"""
CHERRY-PICKING: Apply specific commits from other branches

Use cases:
- Apply hotfix to multiple branches
- Pick specific commit from abandoned branch
- Apply commit without merging entire branch
"""

# Cherry-pick single commit
git cherry-pick abc123

# Cherry-pick range of commits
git cherry-pick abc123..def456

# Cherry-pick without committing (review first)
git cherry-pick --no-commit abc123
git diff --cached  # Review changes
git commit

# Cherry-pick and edit message
git cherry-pick -e abc123

# Example: Apply hotfix to multiple release branches
# Bug fix in main: abc123
git checkout release-1.0
git cherry-pick abc123

git checkout release-2.0
git cherry-pick abc123

git checkout release-3.0
git cherry-pick abc123

# Resolve conflicts if needed
git cherry-pick --continue
git cherry-pick --abort  # If needed
```

### Git Hooks

```bash
"""
GIT HOOKS: Scripts that run on Git events

Common hooks:
- pre-commit: Before commit
- commit-msg: Validate commit message
- pre-push: Before push
- post-merge: After merge
"""

# Hooks location: .git/hooks/

# Example: pre-commit hook (run linters)
# .git/hooks/pre-commit
"""
#!/bin/bash

echo "Running pre-commit checks..."

# Run Black formatter
black --check .
if [ $? -ne 0 ]; then
    echo "❌ Black formatting failed"
    echo "Run: black ."
    exit 1
fi

# Run Flake8 linter
flake8 .
if [ $? -ne 0 ]; then
    echo "❌ Flake8 linting failed"
    exit 1
fi

# Run tests
pytest tests/
if [ $? -ne 0 ]; then
    echo "❌ Tests failed"
    exit 1
fi

echo "✅ All checks passed"
"""

# Make executable
chmod +x .git/hooks/pre-commit

# Example: commit-msg hook (validate message)
# .git/hooks/commit-msg
"""
#!/bin/bash

commit_msg=$(cat "$1")

# Check if message follows conventional commits
if ! echo "$commit_msg" | grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .{10,}"; then
    echo "❌ Invalid commit message format"
    echo ""
    echo "Format: <type>(<scope>): <subject>"
    echo "Example: feat(auth): Add JWT authentication"
    echo ""
    echo "Types: feat, fix, docs, style, refactor, test, chore"
    exit 1
fi

echo "✅ Commit message valid"
"""

# Make executable
chmod +x .git/hooks/commit-msg
```

```python
# Better approach: Use pre-commit framework
# Install: pip install pre-commit

# .pre-commit-config.yaml
"""
repos:
  # Black formatter
  - repo: https://github.com/psf/black
    rev: 24.1.1
    hooks:
      - id: black
        language_version: python3.11

  # isort
  - repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
      - id: isort

  # Flake8
  - repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
      - id: flake8

  # mypy
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]

  # Security check
  - repo: https://github.com/pycqa/bandit
    rev: 1.7.6
    hooks:
      - id: bandit
        args: ['-r', 'src/']

  # Check for secrets
  - repo: https://github.com/Yelp/detect-secrets
    rev: v1.4.0
    hooks:
      - id: detect-secrets
"""

# Install hooks
# $ pre-commit install

# Run manually
# $ pre-commit run --all-files

# Now hooks run automatically on commit!
```

### Submodules

```bash
"""
SUBMODULES: Include other Git repos in your project

Use cases:
- Shared libraries across projects
- Third-party dependencies
- Monorepo management
"""

# Add submodule
git submodule add https://github.com/user/shared-lib.git libs/shared

# Clone repo with submodules
git clone --recursive https://github.com/user/myproject.git

# Or after cloning
git submodule init
git submodule update

# Update submodules
git submodule update --remote

# View submodule status
git submodule status

# Remove submodule
git submodule deinit libs/shared
git rm libs/shared
rm -rf .git/modules/libs/shared

# Example project structure with submodules:
"""
myproject/
├── .git/
├── .gitmodules
├── src/
├── tests/
└── libs/
    ├── shared-utils/    # Submodule
    └── common-models/   # Submodule
"""

# .gitmodules file:
"""
[submodule "libs/shared-utils"]
    path = libs/shared-utils
    url = https://github.com/user/shared-utils.git
[submodule "libs/common-models"]
    path = libs/common-models
    url = https://github.com/user/common-models.git
"""
```

### Frequently Asked Questions

**Q1: When should I use git stash?**

**A:**

```bash
"""
GIT STASH: Temporarily save uncommitted changes

Use cases:
✅ Switch branches without committing
✅ Pull latest changes without conflicts
✅ Experiment without losing work
✅ Context switching
"""

# Stash changes
git stash
# Or with message
git stash save "WIP: Working on feature X"

# List stashes
git stash list
"""
stash@{0}: WIP: Working on feature X
stash@{1}: On main: Quick fix
stash@{2}: WIP: Refactoring
"""

# Apply stash (keep in stash list)
git stash apply
git stash apply stash@{1}  # Apply specific stash

# Pop stash (apply and remove)
git stash pop

# View stash contents
git stash show stash@{0}
git stash show -p stash@{0}  # Show diff

# Drop stash
git stash drop stash@{0}

# Clear all stashes
git stash clear

# Create branch from stash
git stash branch feature-from-stash stash@{0}

# Practical workflow:

# 1. Working on feature, need to fix bug urgently
git stash save "WIP: Feature X half done"
git checkout main
git checkout -b hotfix/urgent-bug
# ... fix bug ...
git commit -m "fix: Urgent bug"
git checkout feature-x
git stash pop  # Resume work

# 2. Pull with uncommitted changes
git stash
git pull origin main
git stash pop

# 3. Experiment without committing
git stash  # Save current work
# ... try different approach ...
git reset --hard HEAD  # Discard experiment
git stash pop  # Restore original work
```

**Why This Matters:** Stashing enables flexible workflow without polluting history.

---

(Part 6 continues with sections 6.3 and 6.4...)

## 6.3 Collaborative Development

**SDLC Phase:** Development, Code Review

**Relevant For:**

- [ ] Requirements gathering
- [ ] System design
- [X] Development (collaboration)
- [X] Testing (peer review)
- [ ] Deployment
- [X] Maintenance (team coordination)

### Pull Request Workflow

```bash
"""
PULL REQUESTS (PR): Code review and merge workflow

Steps:
1. Create feature branch
2. Make changes and commit
3. Push branch
4. Open pull request
5. Code review and discussion
6. Address feedback
7. Merge to main
"""

# 1. Create feature branch
git checkout main
git pull origin main
git checkout -b feature/add-search

# 2. Make changes
# ... write code ...
git add .
git commit -m "feat: Add search functionality"

# 3. Keep branch updated with main
git fetch origin
git rebase origin/main
# Or merge if preferred
# git merge origin/main

# 4. Push branch
git push origin feature/add-search

# 5. Open PR (via GitHub/GitLab web interface)
# Title: "feat: Add search functionality"
# Description template:
"""
## Description
Implements full-text search for products using PostgreSQL full-text search.

## Changes
- Add search endpoint `/api/products/search`
- Create database index on product name and description
- Add search tests

## Testing
- [x] Unit tests pass
- [x] Integration tests pass
- [x] Manual testing completed

## Screenshots
![Search UI](screenshots/search.png)

## Related Issues
Closes #123
"""

# 6. Address review feedback
# Make changes based on comments
git add .
git commit -m "refactor: Extract search logic to separate service"
git push origin feature/add-search

# 7. Squash commits before merge (optional)
git rebase -i HEAD~5
# Squash all commits into one clean commit

# Force push (only on feature branch!)
git push -f origin feature/add-search

# 8. Merge via web interface
# Options:
# - Merge commit (preserves all commits)
# - Squash and merge (one commit)
# - Rebase and merge (linear history)

# 9. Delete branch after merge
git checkout main
git pull origin main
git branch -d feature/add-search
git push origin --delete feature/add-search
```

### Code Review Best Practices

```python
"""
CODE REVIEW GUIDELINES

For Authors:
✅ Keep PRs small (<400 lines)
✅ Write clear descriptions
✅ Self-review before requesting review
✅ Add tests
✅ Update documentation
✅ Respond to feedback promptly

For Reviewers:
✅ Review within 24 hours
✅ Be constructive and kind
✅ Focus on logic, not style (use linters!)
✅ Ask questions, don't demand changes
✅ Approve when ready, request changes when needed
"""

# PR checklist for authors
pr_checklist = """
## PR Checklist

Before requesting review:
- [ ] Code follows project style guide
- [ ] All tests pass locally
- [ ] New tests added for new functionality
- [ ] Documentation updated
- [ ] No commented-out code
- [ ] No debug prints or console.logs
- [ ] Environment variables documented
- [ ] Database migrations tested
- [ ] Error handling implemented
- [ ] Security considerations addressed
"""

# Code review comments - Good examples

# ❌ BAD: Demanding, not specific
"""
This code is bad. Change it.
"""

# ✅ GOOD: Specific, constructive
"""
This function is doing too much. Consider extracting the validation logic
into a separate function for better testability:

```python
def validate_user_input(data):
    # validation logic
    pass

def create_user(data):
    validate_user_input(data)
    # creation logic
```

What do you think?
"""

# ❌ BAD: Personal attack

"""
You don't know how to write Python.
"""

# ✅ GOOD: Educational, kind

"""
Python convention is to use snake_case for function names.
Here's the PEP 8 reference: https://pep8.org/#function-names

Could you rename `createUser` to `create_user`?
"""

# ❌ BAD: Nitpicking

"""
Add space after comma on line 42.
"""

# ✅ GOOD: Let linters handle style

"""
(Configure pre-commit hooks to auto-format code)
"""

# Review categories

# 1. MUST FIX (blocking issues)

"""
🔴 **MUST FIX**: This will cause a security vulnerability.
The SQL query is vulnerable to injection. Please use parameterized queries.
"""

# 2. SHOULD FIX (important but not blocking)

"""
🟡 **Should fix**: This function is hard to test due to tight coupling.
Consider dependency injection for the database connection.
"""

# 3. COULD FIX (nice to have)

"""
💡 **Suggestion**: You could simplify this with a list comprehension:
`result = [x*2 for x in numbers if x > 0]`
"""

# 4. QUESTION (clarification needed)

"""
❓ **Question**: Why did you choose to use a list instead of a set here?
Is order important?
"""

# 5. PRAISE (positive feedback)

"""
✅ **Nice**: Great test coverage! I really like how you tested the edge cases.
"""

```

```bash
# GitHub PR review workflow

# Request review
# In PR, click "Reviewers" and select team members

# Reviewers: Check out PR locally
git fetch origin
git checkout origin/feature/add-search

# Or create local branch
git checkout -b review/feature-add-search origin/feature/add-search

# Run code locally
python -m pytest
python app.py

# Leave review comments on GitHub:
# - Comment: General feedback
# - Approve: LGTM (Looks Good To Me)
# - Request Changes: Must address before merge

# Author: Address feedback
git add .
git commit -m "refactor: Address review comments"
git push origin feature/add-search

# Reviewer: Re-review and approve
# Author: Merge PR

# Merge strategies:

# 1. Merge commit (preserves history)
git merge --no-ff feature/add-search

# 2. Squash and merge (clean history)
git merge --squash feature/add-search
git commit -m "feat: Add search functionality (#123)"

# 3. Rebase and merge (linear history)
git rebase feature/add-search
```

### Pair Programming

```python
"""
PAIR PROGRAMMING: Two developers, one computer

Roles:
- Driver: Writes code
- Navigator: Reviews, suggests, plans

Benefits:
✅ Higher code quality
✅ Knowledge sharing
✅ Fewer bugs
✅ Better designs
✅ Reduced review time

Styles:
1. Driver-Navigator: Classic pairing
2. Ping-Pong: TDD pair programming
3. Strong Style: Navigator tells driver exactly what to type
"""

# 1. Driver-Navigator pairing
"""
Session: Implementing user authentication

Navigator (Alice): Let's start with the test. We need to test that 
                   users can't register with duplicate emails.

Driver (Bob): [Writing test]
def test_duplicate_email_raises_error():
    create_user("alice@example.com")
    with pytest.raises(DuplicateEmailError):
        create_user("alice@example.com")

Navigator: Good. Now let's make it pass. We need to check if 
          the email exists before creating the user.

Driver: [Implementing]
def create_user(email):
    if user_exists(email):
        raise DuplicateEmailError()
    # ... create user ...

Navigator: What if the email is None?

Driver: Good catch. [Adds validation]
if not email:
    raise ValueError("Email required")
"""

# 2. Ping-Pong pairing (TDD)
"""
Developer A: Writes failing test
Developer B: Makes test pass
Developer B: Writes next failing test
Developer A: Makes test pass
... repeat ...

Example:
A: [Writes test] test_add_user()
B: [Implements] add_user() to pass test
B: [Writes test] test_update_user()
A: [Implements] update_user() to pass test
"""

# 3. Remote pair programming tools
remote_tools = {
    "VS Code Live Share": "Real-time collaborative editing",
    "JetBrains Code With Me": "IntelliJ collaborative coding",
    "Tuple": "Screen sharing with low latency",
    "Pop": "Pair programming focused",
    "tmux + ssh": "Terminal pairing (old school)"
}

# Best practices for pairing
pairing_practices = {
    "Switch roles": "Every 15-30 minutes",
    "Take breaks": "Every hour, 10 minute break",
    "Communicate": "Think out loud, explain reasoning",
    "Be patient": "Let partner think",
    "Share keyboard": "Physical comfort matters",
    "Set goals": "What do we want to accomplish?",
    "Retrospect": "What went well? What to improve?"
}

# When to pair
pair_when = """
✅ Complex problems (algorithm design, architecture)
✅ Critical features (payment, security)
✅ Learning/onboarding (senior + junior)
✅ Debugging production issues
✅ Code reviews that need discussion

Don't pair when:
❌ Simple, repetitive tasks
❌ Research/exploration
❌ Writing documentation
❌ Both developers are tired
"""
```

### Frequently Asked Questions

**Q1: How do I handle disagreements in code review?**

**A:**

```python
"""
HANDLING CODE REVIEW DISAGREEMENTS

Scenario: Reviewer and author disagree on implementation

Steps:
1. Understand the concern
2. Present your reasoning
3. Consider alternatives
4. Escalate if needed
5. Make a decision and move forward

Example disagreement: Should we use ORM or raw SQL?
"""

# Reviewer comment:
"""
❓ Why are you using raw SQL here? Our team uses SQLAlchemy ORM.
"""

# ❌ BAD response:
"""
Because I prefer raw SQL. It's faster.
"""

# ✅ GOOD response:
"""
Good question! I used raw SQL for this specific query because:

1. Complex joins (3 tables with subqueries)
2. Performance critical (runs on every page load)
3. ORM generates inefficient query (N+1 problem)

I benchmarked both approaches:
- ORM: 450ms average
- Raw SQL: 45ms average (10x faster)

However, I understand consistency is important. Alternative approaches:

A. Use raw SQL with clear documentation
B. Optimize ORM query with joinedload()
C. Add caching layer

Which approach do you prefer? Happy to discuss synchronously if helpful.
"""

# When to escalate
escalation_triggers = """
Escalate when:
✅ Fundamental disagreement on architecture
✅ Security/compliance concerns
✅ Performance impact is significant
✅ Decision affects multiple teams
✅ Time-sensitive and can't reach agreement

Escalation process:
1. Schedule sync meeting (don't do in PR comments)
2. Present both perspectives objectively
3. Involve tech lead or architect
4. Make decision based on project goals
5. Document decision and reasoning
"""

# After disagreement is resolved
"""
✅ Update code based on decision
✅ Document reasoning in commit message
✅ Add comment in code if needed
✅ Update team guidelines if applicable
✅ No hard feelings - disagree and commit
"""
```

**Why This Matters:** Constructive disagreement leads to better code and stronger teams.

---

## 6.4 Git Hosting Platforms

**SDLC Phase:** All Phases

**Relevant For:**

- [X] Requirements gathering (issues)
- [X] System design (wikis)
- [X] Development (repositories)
- [X] Testing (CI/CD)
- [X] Deployment (releases)
- [X] Maintenance (project management)

### GitHub Features

```yaml
"""
GITHUB: Most popular Git hosting platform

Key Features:
- Pull requests
- Issues and project boards
- GitHub Actions (CI/CD)
- GitHub Pages (static hosting)
- GitHub Packages (registry)
- Discussions
- Security features
"""

# .github/workflows/ci.yml - Already covered in Part 5

# .github/ISSUE_TEMPLATE/bug_report.md
"""
---
name: Bug Report
about: Report a bug
title: '[BUG] '
labels: bug
assignees: ''
---

## Bug Description
A clear description of the bug.

## Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. See error

## Expected Behavior
What should happen.

## Actual Behavior
What actually happens.

## Environment
- OS: [e.g., Ubuntu 22.04]
- Python version: [e.g., 3.11]
- Package version: [e.g., 1.2.3]

## Additional Context
Add any other context, screenshots, or logs.
"""

# .github/ISSUE_TEMPLATE/feature_request.md
"""
---
name: Feature Request
about: Suggest a feature
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## Feature Description
Clear description of the feature.

## Problem It Solves
What problem does this solve?

## Proposed Solution
How should it work?

## Alternatives Considered
What other approaches have you considered?

## Additional Context
Add mockups, examples, or references.
"""

# .github/PULL_REQUEST_TEMPLATE.md
"""
## Description
Brief description of changes.

## Type of Change
- [ ] Bug fix (non-breaking change)
- [ ] New feature (non-breaking change)
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing completed

## Checklist
- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] No new warnings generated
- [ ] Tests pass locally

## Related Issues
Closes #(issue)
"""

# GitHub Projects (Project management)
"""
Create project board:
1. Go to repository → Projects → New project
2. Choose template (Kanban, Table, Roadmap)
3. Add columns: Backlog, To Do, In Progress, Review, Done
4. Link issues to project
5. Automate with workflows
"""

# GitHub Releases
"""
Create release:
1. Git tag: git tag -a v1.0.0 -m "Release 1.0.0"
2. Push tag: git push origin v1.0.0
3. GitHub → Releases → Draft new release
4. Select tag, write release notes, attach binaries
5. Publish release
"""

# CHANGELOG.md (Keep a Changelog format)
"""
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- New feature X

### Changed
- Improved performance of Y

### Fixed
- Bug fix for Z

## [1.0.0] - 2025-01-01

### Added
- Initial release
- User authentication
- Product catalog
- Shopping cart

### Changed
- Nothing (first release)

### Fixed
- Nothing (first release)

### Security
- Nothing (first release)

[Unreleased]: https://github.com/user/repo/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/user/repo/releases/tag/v1.0.0
"""
```

### GitLab Features

```yaml
"""
GITLAB: Integrated DevOps platform

Key Features:
- Merge requests (similar to PRs)
- Built-in CI/CD
- Container registry
- Package registry
- Wiki
- Issue boards
- Time tracking
"""

# .gitlab-ci.yml - Already covered in Part 5

# .gitlab/issue_templates/Bug.md
"""
## Summary
Brief description of the bug.

## Steps to Reproduce
1. Step 1
2. Step 2
3. Step 3

## Expected Result
What should happen.

## Actual Result
What actually happens.

## Environment
- OS: 
- Version: 

## Logs
```

Paste logs here

```

/label ~bug
"""

# .gitlab/merge_request_templates/Default.md
"""
## What does this MR do?
Describe the changes.

## Related issues
Closes #

## Author's checklist
- [ ] Tests added/updated
- [ ] Documentation added/updated
- [ ] Changelog entry added
- [ ] Reviewed own MR

## Review checklist
- [ ] Code quality acceptable
- [ ] Tests cover changes
- [ ] Documentation clear

/assign @reviewer
"""

# GitLab-specific features

# 1. Time tracking
"""
In issue or MR description:
/estimate 2h
/spend 1h 30m
"""

# 2. Quick actions
"""
/assign @user
/label ~bug ~priority::high
/milestone %v1.0
/due 2025-01-15
/close
/approve
"""

# 3. Multiple assignees (GitLab feature)
"""
/assign @user1 @user2 @user3
"""
```

### Frequently Asked Questions

**Q1: GitHub vs GitLab - which should I choose?**

**A:**

```yaml
"""
GITHUB vs GITLAB COMPARISON

Choose GitHub when:
✅ Open source project (better visibility)
✅ Want largest community
✅ Need GitHub-specific integrations
✅ Simple CI/CD needs
✅ Free private repos (unlimited)

Choose GitLab when:
✅ Need built-in DevOps platform
✅ Want self-hosted option
✅ Advanced CI/CD needs
✅ Need built-in container registry
✅ Want integrated security scanning

Feature Comparison:

|Feature|GitHub|GitLab|
|-------|------|------|
|Free private repos|✅ Unlimited|✅ Unlimited|
|CI/CD|✅ GitHub Actions|✅ Built-in (more features)|
|Container Registry|✅ GitHub Packages|✅ Built-in|
|Self-hosted|❌ Enterprise only|✅ Free (Community Edition)|
|Issue boards|✅ Projects|✅ Built-in|
|Wiki|✅ Yes|✅ Yes|
|Pages (static sites)|✅ Yes|✅ Yes|
|Security scanning|✅ Dependabot|✅ More comprehensive|

Pricing (as of 2025):

GitHub:
- Free: Unlimited public/private repos
- Team: $4/user/month
- Enterprise: $21/user/month

GitLab:
- Free: Unlimited public/private repos
- Premium: $19/user/month
- Ultimate: $99/user/month

Recommendation:
- Open source → GitHub (better discoverability)
- Enterprise → Either (based on existing tools)
- Startups → GitHub (simpler, good free tier)
- DevOps heavy → GitLab (integrated platform)
"""
```

**Why This Matters:** Platform choice affects team workflows and costs.

---

### Key Takeaways

**Version Control & Collaboration:**

- **Git Fundamentals**: Init, add, commit, branch, merge
- **Commit Messages**: Conventional commits (feat, fix, docs, etc.)
- **Branching**: Git Flow (complex), GitHub Flow (simple), Trunk-Based (CI/CD)
- **Best Practices**: Atomic commits, meaningful messages, pull before push
- **Advanced Git**: Rebase for clean history, cherry-pick for specific commits
- **Git Hooks**: Automate checks (pre-commit for linting/tests)
- **Stashing**: Temporary save of uncommitted changes
- **Pull Requests**: Code review workflow
- **Code Review**: Constructive feedback, < 400 lines, respond within 24h
- **Pair Programming**: Driver-Navigator, Ping-Pong, Strong Style
- **Platforms**: GitHub (community), GitLab (integrated DevOps)

**Best Practices:**

- Commit often, push regularly
- Write meaningful commit messages (Conventional Commits)
- Use branches for features
- Keep PRs small and focused
- Review code within 24 hours
- Be kind and constructive in reviews
- Never rebase public branches
- Use pre-commit hooks for quality checks

**Tools Summary:**

- **Version Control**: Git
- **Hosting**: GitHub, GitLab, Bitbucket
- **Automation**: pre-commit framework
- **Collaboration**: Pull requests, code reviews
- **Project Management**: GitHub Projects, GitLab Boards

---

## Part 6 Complete!

Congratulations! You've completed **Part 6: Version Control & Collaboration**. You now have comprehensive knowledge of:

✅ **Section 6.1**: Git Fundamentals
✅ **Section 6.2**: Advanced Git
✅ **Section 6.3**: Collaborative Development
✅ **Section 6.4**: Git Hosting Platforms

**What You've Learned:**

- Git repository management and workflow
- Branching strategies for different team sizes
- Advanced Git operations (rebase, cherry-pick, hooks)
- Pull request workflow and best practices
- Code review guidelines (author and reviewer)
- Pair programming techniques
- GitHub and GitLab platform features

**Complete Series Status:**

✅ **Part 1**: Python Fundamentals (~29,000 words)
✅ **Part 2**: Python Internals (~20,000 words)
✅ **Part 3**: SDLC & Architecture (~16,000 words)
✅ **Part 4**: Testing & Quality (~9,200 words)
✅ **Part 5**: Deployment & DevOps (~8,000 words)
✅ **Part 6**: Version Control & Collaboration (~7,000 words)

**Total: ~89,200 words = ~357 pages of comprehensive Python expertise!**

You now have complete Python engineering skills from fundamentals through team collaboration! 🎓🚀
