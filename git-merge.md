# Git Merge and Rebase Guide

## Table of Contents
- [Git Merge](#git-merge)
- [Git Rebase](#git-rebase)
- [When to Use Merge vs Rebase](#when-to-use-merge-vs-rebase)
- [Common Scenarios](#common-scenarios)
- [Best Practices](#best-practices)
- [Working with forks](#working-with-forks-and-upstream-updates)

## Git Merge

Git merge is used to combine changes from different branches. It creates a new merge commit that ties together the histories of both branches.

### Basic Merge Commands

#### Merge a branch into current branch
```bash
# Switch to target branch (usually main/master)
git checkout main
git pull origin main

# Merge feature branch
git merge feature-branch-name
```

#### Fast-forward merge (when possible)
```bash
git merge --ff-only feature-branch-name
```

#### Force a merge commit (no fast-forward)
```bash
git merge --no-ff feature-branch-name
```

### Merge Types

#### 1. Fast-forward Merge
- Occurs when the target branch hasn't diverged from the source branch
- Simply moves the branch pointer forward
- No merge commit is created

#### 2. Three-way Merge
- Occurs when both branches have diverged
- Creates a new merge commit with two parents
- Preserves the complete history of both branches

### Handling Merge Conflicts

When Git can't automatically merge changes:

```bash
# After running git merge, if conflicts occur:
# 1. Git will mark conflicted files
git status

# 2. Edit conflicted files manually
# Look for conflict markers:
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> branch-name

# 3. Stage resolved files
git add resolved-file.txt

# 4. Complete the merge
git commit
```

### Useful Merge Options

```bash
# See what would be merged without actually merging
git merge --no-commit --no-ff feature-branch

# Merge with a custom commit message
git merge -m "Custom merge message" feature-branch

# Abort a merge in progress
git merge --abort
```

## Git Rebase

Git rebase rewrites commit history by replaying commits from one branch onto another. It creates a linear history without merge commits.

### Basic Rebase Commands

#### Rebase current branch onto another branch
```bash
# Switch to feature branch
git checkout feature-branch

# Rebase onto main
git rebase main
```

#### Interactive rebase (to edit commit history)
```bash
# Rebase last 3 commits interactively
git rebase -i HEAD~3

# Rebase from a specific commit
git rebase -i commit-hash
```

### Interactive Rebase Options

When you run `git rebase -i`, you can:
- `pick` (p): Use the commit as-is
- `reword` (r): Use the commit but edit the message
- `edit` (e): Use the commit but stop for amending
- `squash` (s): Combine with previous commit
- `fixup` (f): Like squash but discard commit message
- `drop` (d): Remove the commit

Example interactive rebase:
```
pick abc1234 Add user authentication
squash def5678 Fix typo in auth
reword ghi9012 Update documentation
drop jkl3456 Debug commit to remove
```

### Handling Rebase Conflicts

```bash
# When conflicts occur during rebase:
# 1. Resolve conflicts in files
# 2. Stage the resolved files
git add resolved-file.txt

# 3. Continue the rebase
git rebase --continue

# Or abort the rebase
git rebase --abort

# Or skip the current commit
git rebase --skip
```

### Advanced Rebase Operations

#### Rebase with autosquash
```bash
# Create fixup commits
git commit --fixup=commit-hash

# Automatically arrange and squash fixup commits
git rebase -i --autosquash HEAD~5
```

#### Rebase onto a different base
```bash
# Rebase feature-branch from old-base onto new-base
git rebase --onto new-base old-base feature-branch
```

## When to Use Merge vs Rebase

### Use Merge When:
- Working on a shared/public branch
- You want to preserve the exact history of how changes were integrated
- Working with a team that prefers merge commits for tracking
- The feature branch has already been pushed and shared

### Use Rebase When:
- Cleaning up your local commit history before pushing
- You want a linear, clean project history
- Working on a personal feature branch
- Incorporating latest changes from main into your feature branch

### Golden Rule of Rebasing
**Never rebase commits that have been pushed to a shared repository and that others may have based work on.**

## Common Scenarios

### Scenario 1: Update feature branch with latest main
```bash
# Using merge (preserves history)
git checkout feature-branch
git merge main

# Using rebase (linear history)
git checkout feature-branch
git rebase main
```

### Scenario 2: Preparing feature branch for merge
```bash
# Clean up commits with interactive rebase
git checkout feature-branch
git rebase -i main

# Then merge into main
git checkout main
git merge feature-branch
```

### Scenario 3: Fixing mistakes in recent commits
```bash
# Amend the last commit
git commit --amend

# Fixup a specific commit
git commit --fixup=commit-hash
git rebase -i --autosquash HEAD~5
```

### Scenario 4: Squashing multiple commits
```bash
# Interactive rebase to squash last 3 commits
git rebase -i HEAD~3
# Change 'pick' to 'squash' for commits you want to combine
```

## Working with Forks and Upstream Updates

When you fork a repository and make changes, you'll need to periodically sync with the upstream (parent) repository to get the latest updates. This often leads to conflicts that need to be resolved.

### Initial Fork Setup

#### 1. Clone your fork
```bash
git clone https://github.com/YOUR-USERNAME/REPO-NAME.git
cd REPO-NAME
```

#### 2. Add upstream remote
```bash
# Add the original repository as upstream
git remote add upstream https://github.com/ORIGINAL-OWNER/REPO-NAME.git

# Verify remotes
git remote -v
# origin    https://github.com/YOUR-USERNAME/REPO-NAME.git (fetch)
# origin    https://github.com/YOUR-USERNAME/REPO-NAME.git (push)
# upstream  https://github.com/ORIGINAL-OWNER/REPO-NAME.git (fetch)
# upstream  https://github.com/ORIGINAL-OWNER/REPO-NAME.git (push)
```

### Syncing with Upstream

#### Method 1: Merge Strategy (Preserves Fork History)
```bash
# 1. Fetch latest changes from upstream
git fetch upstream

# 2. Switch to your main branch
git checkout main

# 3. Merge upstream changes
git merge upstream/main

# 4. Push updated main to your fork
git push origin main
```

#### Method 2: Rebase Strategy (Cleaner History)
```bash
# 1. Fetch latest changes from upstream
git fetch upstream

# 2. Switch to your main branch
git checkout main

# 3. Rebase your changes onto upstream
git rebase upstream/main

# 4. Force push to your fork (use with caution)
git push --force-with-lease origin main
```

### Handling Conflicts During Upstream Sync

#### Scenario 1: Conflicts in Main Branch
When upstream changes conflict with your changes in the main branch:

```bash
# During merge - if conflicts occur
git fetch upstream
git checkout main
git merge upstream/main
# CONFLICT: Fix conflicts manually

# 1. Check which files have conflicts
git status

# 2. Edit conflicted files
# Remove conflict markers and choose/combine changes
# <<<<<<< HEAD
# Your changes
# =======
# Upstream changes
# >>>>>>> upstream/main

# 3. Stage resolved files
git add conflicted-file.txt

# 4. Complete the merge
git commit

# 5. Push to your fork
git push origin main
```

#### Scenario 2: Conflicts in Feature Branch
When working on a feature branch that conflicts with upstream updates:

```bash
# 1. Update your main branch first
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# 2. Rebase your feature branch onto updated main
git checkout feature-branch
git rebase main

# 3. Resolve conflicts if they occur
# Edit files, then:
git add resolved-file.txt
git rebase --continue

# 4. Force push your feature branch
git push --force-with-lease origin feature-branch
```

### Common Fork Conflict Scenarios

#### Scenario 1: Same File Modified
**Problem**: Both you and upstream modified the same file in the same location.

**Solution**:
```bash
# After conflict occurs during merge/rebase
# 1. Open the conflicted file
# 2. Look for conflict markers:
# <<<<<<< HEAD (your changes)
# Your modification
# =======
# Upstream modification
# >>>>>>> upstream/main

# 3. Decide how to resolve:
# - Keep your changes only
# - Keep upstream changes only
# - Combine both changes
# - Write something completely new

# 4. Remove conflict markers and save
# 5. Stage and continue
git add file.txt
git rebase --continue  # or git commit for merge
```

#### Scenario 2: File Renamed/Moved
**Problem**: You modified a file that upstream renamed or moved.

**Solution**:
```bash
# Git usually handles this automatically, but if not:
# 1. Find where the file was moved in upstream
git log --follow --name-status upstream/main -- old-filename

# 2. Apply your changes to the new location
# 3. Remove the old file if it still exists
git rm old-filename
git add new-filename
```

#### Scenario 3: Configuration Files
**Problem**: You customized configuration files that upstream also updated.

**Solution** - Create a local configuration strategy:
```bash
# 1. Keep upstream config as base
# 2. Create your custom config files
cp config.example.yml config.yml

# 3. Add custom configs to .gitignore
echo "config.yml" >> .gitignore
echo "local.env" >> .gitignore

# 4. Document your customizations
# This way upstream updates won't conflict with your local setup
```

### Fork-Specific Best Practices

#### 1. Regular Sync Schedule
```bash
# Weekly or bi-weekly sync routine
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Update all your feature branches
git checkout feature-branch
git rebase main
```

#### 2. Keep Fork Main Clean
```bash
# Never commit directly to main in your fork
# Always work in feature branches
git checkout -b feature/my-changes main

# This makes syncing with upstream much easier
```

#### 3. Use Feature Branches for Customizations
```bash
# Create a branch for your specific modifications
git checkout -b customizations main

# Make your changes
git add .
git commit -m "Add custom features for my use case"

# When syncing with upstream
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# Rebase your customizations
git checkout customizations
git rebase main
```

#### 4. Handle Large Upstream Changes
When upstream has major refactoring that conflicts heavily:

```bash
# Option 1: Reset and reapply (for major conflicts)
git checkout main
git fetch upstream
git reset --hard upstream/main
git push --force-with-lease origin main

# Then cherry-pick your important commits
git cherry-pick your-commit-hash

# Option 2: Create a new branch from upstream
git checkout -b new-main upstream/main
git cherry-pick commit1 commit2 commit3
```

### Resolving Specific Conflict Types

#### Package Dependencies
```bash
# Common in package.json, requirements.txt, etc.
# 1. Take upstream version as base
# 2. Add back your additional dependencies
# 3. Test that everything still works
npm install  # or pip install -r requirements.txt
```

#### Documentation Changes
```bash
# Usually safe to combine
# Keep upstream updates and add your additional docs
# Use merge strategy rather than choosing one side
```

#### Code Changes in Same Function
```bash
# Most complex scenario
# 1. Understand what upstream changed and why
# 2. Understand what you changed and why
# 3. Combine the logic if possible
# 4. Test thoroughly after resolution
```

### Automation and Tools

#### Create a sync script
```bash
#!/bin/bash
# sync-upstream.sh

echo "Syncing with upstream..."
git fetch upstream
git checkout main
git merge upstream/main

if [ $? -eq 0 ]; then
    echo "Sync successful, pushing to origin..."
    git push origin main
else
    echo "Conflicts detected. Please resolve manually."
    exit 1
fi
```

#### Use GitHub CLI for easier fork management
```bash
# Install GitHub CLI, then:
gh repo fork ORIGINAL-OWNER/REPO-NAME
gh repo sync YOUR-USERNAME/REPO-NAME
```

## Best Practices

### General Guidelines
1. **Always pull before pushing**: `git pull --rebase origin main`
2. **Use descriptive commit messages**
3. **Keep commits atomic** (one logical change per commit)
4. **Test before merging/pushing**

### Merge Best Practices
- Use `--no-ff` for feature branches to preserve branch history
- Delete merged branches: `git branch -d feature-branch`
- Use merge commits for integration points

### Rebase Best Practices
- Only rebase private branches
- Use interactive rebase to clean up commit history
- Rebase feature branches onto main before merging
- Use `git pull --rebase` to avoid unnecessary merge commits

### Workflow Recommendations

#### Feature Branch Workflow with Merge
```bash
# 1. Create and switch to feature branch
git checkout -b feature/new-feature main

# 2. Work and commit
git add .
git commit -m "Implement new feature"

# 3. Update main and merge
git checkout main
git pull origin main
git merge --no-ff feature/new-feature

# 4. Push and cleanup
git push origin main
git branch -d feature/new-feature
```

#### Feature Branch Workflow with Rebase
```bash
# 1. Create and switch to feature branch
git checkout -b feature/new-feature main

# 2. Work and commit
git add .
git commit -m "Implement new feature"

# 3. Rebase onto latest main
git fetch origin
git rebase origin/main

# 4. Force push feature branch (if needed)
git push --force-with-lease origin feature/new-feature

# 5. Merge into main (fast-forward)
git checkout main
git merge feature/new-feature
```

### Troubleshooting

#### Undoing a merge
```bash
# If merge hasn't been pushed yet
git reset --hard HEAD~1

# If merge has been pushed
git revert -m 1 merge-commit-hash
```

#### Undoing a rebase
```bash
# Find the commit before rebase in reflog
git reflog

# Reset to that commit
git reset --hard HEAD@{n}
```

#### Recovering lost commits
```bash
# Use reflog to find lost commits
git reflog

# Cherry-pick or reset to recover
git cherry-pick commit-hash
```

Remember: Git is powerful but can be destructive. Always make sure you have backups of important work, and practice these commands on test repositories first!