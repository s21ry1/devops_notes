# Git Commands Reference for DevOps

## Repository Setup
git init                  # Initialize new Git repository
git clone url            # Clone repository from remote
git remote add origin url # Add remote repository
git remote -v            # List remote repositories

## Basic Commands
git status               # Show working tree status
git add filename         # Stage specific file
git add .                # Stage all changes
git commit -m "message"  # Commit with message
git push origin branch   # Push to remote
git pull origin branch   # Pull from remote

## Branch Management
git branch               # List local branches
git branch -r            # List remote branches
git branch -a            # List all branches
git branch name          # Create new branch
git checkout branch      # Switch to branch
git checkout -b branch   # Create and switch to new branch
git merge branch         # Merge branch into current
git branch -d branch     # Delete local branch
git push origin --delete branch  # Delete remote branch

## History and Diff
git log                  # Show commit history
git log --oneline       # Compact commit history
git log --graph         # Show branch graph
git diff                # Show unstaged changes
git diff --staged       # Show staged changes
git blame file          # Show who changed what
git show commit         # Show commit details

## Stashing
git stash               # Stash changes
git stash list         # List stashes
git stash pop          # Apply and remove stash
git stash apply        # Apply but keep stash
git stash drop         # Remove stash

## Advanced Operations
git rebase master       # Rebase current branch
git cherry-pick commit  # Copy commit to current branch
git reset --hard HEAD   # Reset to last commit
git reset --soft HEAD^  # Undo last commit
git revert commit      # Create new commit that undoes changes

## Tags
git tag                # List tags
git tag -a v1.0 -m "message"  # Create annotated tag
git push origin --tags # Push tags to remote
git tag -d tagname     # Delete local tag
git push origin :tagname # Delete remote tag

## Configuration
git config --global user.name "name"    # Set username
git config --global user.email "email"  # Set email
git config --list      # List configuration
git config --global core.editor "editor" # Set default editor

## Remote Operations
git fetch origin       # Download objects and refs
git remote show origin # Show remote details
git remote prune origin # Remove deleted remote branches
git push -u origin branch # Set upstream branch

## Submodules
git submodule add url    # Add submodule
git submodule update --init # Initialize submodules
git submodule update --recursive # Update submodules

## Maintenance
git gc                  # Garbage collection
git fsck               # Check repository
git prune              # Remove unreachable objects
git clean -fd          # Remove untracked files

## Debugging
git bisect start       # Start binary search
git bisect good        # Mark commit as good
git bisect bad         # Mark commit as bad
git grep "pattern"     # Search in files

## Advanced History
git reflog             # Show reference logs
git log -p file        # Show file history
git log --author="name" # Show author commits
git shortlog          # Summarize git log

## Workflow Commands
git flow init         # Initialize git flow
git flow feature start # Start feature branch
git flow release start # Start release branch
git flow hotfix start # Start hotfix branch

## CI/CD Related
git archive --format=zip HEAD > archive.zip  # Create release archive
git describe          # Generate version description
git rev-parse HEAD    # Get current commit hash
git rev-list --count HEAD  # Count commits

## Large File Storage
git lfs install      # Setup Git LFS
git lfs track "*.psd" # Track large files
git lfs push origin branch # Push LFS files

## Repository Information
git count-objects -v # Show repository size
git rev-parse --git-dir # Show .git directory location
git check-ignore file  # Check if file is ignored
git status -s         # Show short status

## Hooks
.git/hooks/pre-commit  # Pre-commit hook
.git/hooks/post-commit # Post-commit hook
.git/hooks/pre-push   # Pre-push hook

## Security
git secret init      # Initialize git-secret
git secret tell email # Add user to git-secret
git secret hide     # Encrypt files
git secret reveal   # Decrypt files 
