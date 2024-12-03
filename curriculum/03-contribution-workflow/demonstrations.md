# Contribution Workflow Demonstrations

## 1. Repository Setup

### Demo: Initial Setup
```bash
# Fork repository on GitHub first, then:
git clone https://github.com/YOUR_USERNAME/bitcoin.git
cd bitcoin
git remote add upstream https://github.com/bitcoin/bitcoin.git

# Verify remotes
git remote -v
```

### Demo: Branch Management
```bash
# Create feature branch
git checkout -b feature-name

# Keep branch updated
git fetch upstream
git rebase upstream/master
```

## 2. Making Changes

### Demo: Code Style Compliance
```bash
# Format code according to style guide
clang-format -i path/to/file.cpp

# Check for lint errors
make lint

# Run static analysis
make static
```

### Demo: Testing Changes
```bash
# Build and run tests
./autogen.sh
./configure
make check

# Run specific test
test/functional/feature_rbf.py
```

## 3. Commit Management

### Demo: Creating Good Commits
```bash
# Stage specific changes
git add -p

# Create well-formatted commit
git commit -m "component: Short description

Longer explanation of what this change does and why.
Include any relevant context or links.

Fixes #1234"
```

### Demo: Commit History Management
```bash
# Squash commits
git rebase -i HEAD~3

# Amend last commit
git commit --amend

# Force push (after rebase/amend)
git push -f origin feature-name
```

## 4. Pull Request Creation

### Demo: PR Preparation
```bash
# Update branch
git fetch upstream
git rebase upstream/master

# Run final checks
make check
make lint
```

### Demo: PR Template Usage
```markdown
## Description
Implements feature X to solve problem Y

## Motivation and Context
This change is needed because...

## How Has This Been Tested?
- Added unit tests for X
- Ran functional tests
- Tested manually by...

## Breaking Changes
None

## Related Issues
Fixes #1234
```

## 5. Code Review Process

### Demo: Handling Review Comments
```bash
# Make requested changes
git add .
git commit --amend
git push -f origin feature-name

# Add review responses
> Can you explain why X?
This approach was chosen because...
```

### Demo: CI Integration
```bash
# Check CI status
open https://travis-ci.org/bitcoin/bitcoin/pull/XXXX

# Debug CI failures
test/functional/test_runner.py --loglevel=DEBUG test_name
```

## 6. Community Interaction

### Demo: IRC Communication
```bash
# Join development channel
/join #bitcoin-core-dev

# Ask for review
<you> Would anyone be willing to review PR #1234? It implements...
```

### Demo: PR Review Club
```bash
# Prepare for review club
git checkout master
git fetch upstream
git checkout pr-to-review
```

## 7. Documentation

### Demo: Documentation Updates
```bash
# Update docs
vim doc/developer-notes.md

# Generate documentation
make doc

# Check formatting
make check-doc
```

## 8. Advanced Workflows

### Demo: Backporting
```bash
# Cherry-pick to maintenance branch
git checkout 0.21
git cherry-pick -x commit-hash

# Resolve conflicts if any
git add .
git cherry-pick --continue
```

### Demo: Interactive Rebase
```bash
# Clean up commit history
git rebase -i upstream/master

# Common rebase commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
```