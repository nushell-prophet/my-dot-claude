# Jujutsu (jj) Guide for AI Agents

> **📋 IMMEDIATE ACTION REQUIRED:**
> 
> **Review your recent jj commands against this guide.** If you used any of the following interactive commands, they will fail in automated environments:
> - `jj split` (without specific files)
> - `jj squash -i`
> - `jj diffedit` 
> - `jj rebase -i`
> - `jj resolve`
> 
> **Replace them with the AI-safe alternatives shown in this guide.**

## Overview
Jujutsu is a modern version control system that provides a simplified workflow compared to Git. This guide focuses on key differences from Git that AI agents should understand.

## ⚠️ Critical for AI Agents: Avoid Interactive Commands

**Interactive commands will fail in automated environments.** Always use non-interactive alternatives:

### Commands to AVOID:
- `jj split` (interactive by default)
- `jj squash -i` (interactive flag)
- `jj diffedit` (interactive by default)
- `jj rebase -i` (interactive flag)
- `jj resolve` (interactive by default)

### Safe Non-Interactive Alternatives:
- `jj split --tool :builtin` or split by files: `jj split <files>`
- `jj squash` (move entire change) or `jj squash <files>`
- `jj edit <commit>` then modify and `jj squash`
- `jj rebase -s <source> -d <dest>` (specific revisions)
- Edit conflict markers in files directly, then `jj squash`

## Core Conceptual Differences

### 1. Working Copy is Always a Commit
- **Git**: Working copy is separate from commits; requires `git add` and `git commit`
- **jj**: Working copy is automatically committed on every operation
- **Impact**: No staging area needed; changes are automatically tracked

### 2. No Index/Staging Area
- **Git**: Three-tree architecture (working copy, index, HEAD)
- **jj**: Two-tree architecture (working copy, repository)
- **Impact**: Simpler mental model; use file-specific operations instead of `git add -p`

### 3. Change IDs vs Commit IDs
- **Git**: Only commit IDs (SHA hashes)
- **jj**: Both change IDs (stable across rewrites) and commit IDs
- **Impact**: Change IDs remain constant when commits are rewritten

### 4. No Current Branch Concept
- **Git**: Current branch moves with new commits
- **jj**: No current branch; bookmarks don't move automatically
- **Impact**: Must manually update bookmarks after creating commits

### 5. Conflicts Can Be Committed
- **Git**: Conflicts block operations until resolved
- **jj**: Conflicts are stored in commits and can be resolved later
- **Impact**: Operations never fail due to conflicts

### 6. Automatic Descendant Rebasing
- **Git**: Manual rebasing of dependent branches
- **jj**: Automatic rebasing of all descendants when rewriting commits
- **Impact**: No orphaned commits; dependency chains maintained

## Key Command Differences

### Repository Operations
```bash
# Initialize
git init                     → jj git init [--colocate]
git clone <url>             → jj git clone <url>

# Status and diffs
git status                  → jj st
git diff                    → jj diff
git diff HEAD               → jj diff
git diff <rev>^ <rev>       → jj diff -r <rev>
git log --oneline --graph   → jj log
```

### Working with Changes
```bash
# Creating commits
git add <file>              → (automatic in jj)
git commit -m "msg"         → jj commit -m "msg"
git commit --amend          → jj describe / jj squash

# Moving between commits
git checkout <commit>       → jj edit <commit>
git switch -c new main      → jj new main

# Abandoning changes
git reset --hard            → jj abandon
git reset --soft HEAD~      → jj squash --from @-
```

### Advanced Operations
```bash
# Non-interactive operations (AI-safe)
git add -p; git commit --amend → jj squash <files> (specific files)
git rebase <branch>         → jj rebase -s <source> -d <dest>
git cherry-pick             → jj duplicate -d <dest>

# Conflict resolution (automated)
git add <file>; git rebase --continue → Edit files directly; jj squash

# Undoing operations
git reflog; git reset       → jj op log; jj undo
```

## Unique jj Features

### 1. Operation Log
- Records all repository operations with metadata
- Enables powerful undo functionality: `jj undo`
- View with: `jj op log`

### 2. Content Movement Commands (AI-Safe Variants)
- `jj split <files>`: Split specific files into new commit
- `jj squash`: Move entire change to parent
- `jj squash <files>`: Move specific files to parent
- `jj squash --from <commit>`: Move changes from specific commit
- `jj squash --into <commit>`: Move changes to specific commit

### 3. Revsets (Query Language)
- `@`: Current working copy commit
- `@-`: Parent of working copy
- `::@`: Ancestors of working copy
- `@::`: Descendants of working copy
- `all()`: All visible commits
- `heads()`: All head commits

## Advanced Revsets for AI Agents

### Content-Based Queries
```bash
# Find commits by description patterns
description(glob:"*fix*")           # Contains "fix"
description(regex:"^(feat|fix):")   # Conventional commits
description(exact:"Initial commit") # Exact match

# Find commits by author/committer
author("bot@company.com")           # Specific author
author(glob:"*bot*")               # Pattern matching
mine()                             # Your commits
author_date(after:"2024-01-01")    # Date filtering
committer_date(before:"1 week ago") # Relative dates

# Find commits by file changes
files("src/")                      # Touching src/ directory
files(glob:"*.py")                 # Python files
files(regex:"test.*\.py")          # Regex patterns
diff_contains("TODO")              # Content changes
```

### State-Based Queries
```bash
# Repository state
empty()                            # Empty commits
conflicts()                        # Commits with conflicts
immutable()                        # Immutable commits
divergent()                        # Divergent changes
present(@)                         # Check if revision exists

# Bookmark-related
bookmarks()                        # All bookmarked commits
remote_bookmarks()                 # Remote bookmarks
tracked_remote_bookmarks()         # Tracked remotes
git_refs()                         # Git references
git_head()                         # Git HEAD
```

### Complex Combinations
```bash
# AI-useful patterns
mine() & empty()                   # My empty commits
conflicts() & @::                  # Conflicts in my branch
files("src/") & description("test") # Test changes in src/
author_date(after:"yesterday") & mine() # My recent work
bookmarks() & ~immutable()         # Mutable bookmarked commits
heads() & ~bookmarks()             # Anonymous heads
```

### Bulk Operations with all:
```bash
# When working with many commits
jj rebase -d main -s 'all:feat/*'  # Rebase all feat branches
jj abandon 'all:empty() & mine()'   # Abandon my empty commits
jj describe 'all:@::' -m "Updated"  # Describe all descendants
```

### 4. Bookmark Management
- `jj bookmark create <name>`: Create bookmark
- `jj bookmark move <name> --to <rev>`: Move bookmark
- `jj bookmark track <name>@<remote>`: Track remote bookmark

## Configuration for Automated Environments

### Essential AI Agent Configuration
Create or update `~/.jjconfig.toml`:

```toml
[ui]
# Disable interactive features
paginate = "never"                  # No pager interruption
color = "never"                     # Easier output parsing
diff-editor = ":builtin"            # Avoid external editors
merge-editor = ":builtin"           # Avoid external editors
default-command = "log"             # Safe default command

# Conflict handling
conflict-marker-style = "diff"      # Simpler conflict format

[snapshot]
# Control automatic tracking
auto-track = 'all()'                # Track all files (default)
# auto-track = 'none()'             # Disable auto-tracking
# auto-track = '!glob("*.tmp")'     # Track except temp files

[template-aliases]
# Pre-configured templates for AI parsing
json = 'json()'
csv = 'commit_id ++ "," ++ description ++ "," ++ author.email()'
simple = 'commit_id ++ " " ++ description'

[revset-aliases]
# Common AI patterns
work = 'mine() & author_date(after:"1 week ago")'
recent = 'author_date(after:"yesterday")'
problems = 'conflicts() | divergent() | empty()'

[git]
# Git integration settings
private-commits = 'description(glob:"*WIP*") | description(glob:"*tmp*")'
push = true                         # Enable git push
fetch = true                        # Enable git fetch

[core]
# Performance settings
fsmonitor = true                    # Enable filesystem monitoring
watchman = true                     # Use Watchman if available
```

### Environment Variables
```bash
# Set in your automation environment
export JJ_CONFIG=/path/to/automation.toml
export JJ_USER="ai-agent@company.com"
export JJ_EMAIL="ai-agent@company.com"
export NO_COLOR=1                   # Disable colors globally
export PAGER=""                     # Disable pager
```

### Repository-Specific Configuration
Create `.jj/config.toml` in your repository:

```toml
[ui]
# Project-specific UI settings
default-revset = "main..@"          # Show work branch by default
log-word-wrap = true                # Better log formatting

[snapshot]
# Project-specific tracking
auto-track = '!glob("*.pyc") & !glob("node_modules/**")'

[template-aliases]
# Project-specific templates
pr-format = 'description ++ "\n\nCommit: " ++ commit_id'
changelog = 'description ++ " (" ++ author.name() ++ ")"'
```

## Template System for Structured Output

### Built-in Templates for AI Parsing
```bash
# JSON output for programmatic processing
jj log --template='json()' -r 'mine()'

# Custom structured output
jj log --template='{
  "commit": commit_id,
  "change": change_id,
  "author": author.email(),
  "date": author_date,
  "message": description,
  "files": files,
  "branches": bookmarks
}' -r '@::main'

# CSV format for spreadsheet processing
jj log --template='commit_id ++ "," ++ description.first_line() ++ "," ++ author.email() ++ "," ++ author_date'

# Simple machine-readable format
jj log --template='commit_id ++ "\t" ++ change_id ++ "\t" ++ description.first_line()'
```

### Conditional Templates
```bash
# Show conflicts differently
jj log --template='
if(conflicts,
  "CONFLICT: " ++ description,
  "OK: " ++ description
)' -r 'all()'

# Show empty commits
jj log --template='
if(empty,
  "[EMPTY] " ++ description,
  description
) ++ " (" ++ commit_id.short() ++ ")"'

# Different format for different authors
jj log --template='
if(author.email().contains("bot"),
  "🤖 " ++ description,
  "👤 " ++ description
)'
```

### Advanced Template Functions
```bash
# Commit metadata extraction
jj log --template='
{
  "id": commit_id,
  "change": change_id,
  "parents": parents.map(|p| p.commit_id()),
  "author": {
    "name": author.name(),
    "email": author.email(),
    "date": author_date.format("%Y-%m-%d %H:%M:%S")
  },
  "committer": {
    "name": committer.name(),
    "email": committer.email(),
    "date": committer_date.format("%Y-%m-%d %H:%M:%S")
  },
  "message": {
    "full": description,
    "summary": description.first_line(),
    "body": description.rest()
  },
  "files": files.map(|f| f.path()),
  "stats": {
    "empty": empty,
    "conflicts": conflicts,
    "divergent": divergent
  }
}'

# File change analysis
jj log --template='
files.map(|f| 
  f.path() ++ " (" ++ 
  if(f.change_type() == "added", "A",
    if(f.change_type() == "deleted", "D",
      if(f.change_type() == "modified", "M", "?")
    )
  ) ++ ")"
).join(", ")' -r '@'
```

### Template Aliases Usage
```bash
# Using pre-configured aliases from config
jj log --template=json -r 'work'     # JSON output of work commits
jj log --template=csv -r 'recent'    # CSV format of recent commits
jj log --template=simple -r 'problems' # Simple format for problem commits
```

## Mental Model Adjustments

### Git Mental Model
```
Working Copy → Index → Repository
    ↓           ↓         ↓
  Unstaged → Staged → Committed
```

### jj Mental Model
```
Working Copy ↔ Repository
     ↓           ↓
  Change ID → Commit ID
```

## Common Workflows

### 1. Starting Work
```bash
# Git
git checkout -b feature main
git add <files>
git commit -m "Initial work"

# jj
jj new main -m "Initial work"
# Edit files - they're automatically committed
```

### 2. Amending Previous Commit
```bash
# Git
git add <files>
git commit --amend

# jj
jj describe @-  # Edit description
jj squash --into @-  # Move current changes
```

### 3. Selective Staging (AI-Safe)
```bash
# Git
git add -p
git commit

# jj (AI-safe alternatives)
jj split <files>  # Split specific files into new commit
jj squash <files>  # Move specific files to parent
jj new; mv <files> to staging area; jj squash  # Manual staging
```

### 4. Resolving Conflicts (AI-Safe)
```bash
# Git
git merge feature
# Fix conflicts in files
git add <files>
git commit

# jj (AI-safe - no interactive resolution)
jj new @ feature  # Create merge commit
# Conflicts are in working copy with markers like:
# <<<<<<< Conflict 1 of 1
# %%%%%%% Changes from base to side #1
# +content from side 1
# +++++++ Contents of side #2
# content from side 2
# >>>>>>> Conflict 1 of 1 ends

# Edit files directly to resolve, then:
jj squash  # Incorporate resolution
```

## Advanced Conflict Resolution

### Understanding Conflict Markers
jj uses different conflict marker styles controlled by `ui.conflict-marker-style`:

#### 1. `diff` style (AI-recommended)
```
<<<<<<< Conflict 1 of 1
%%%%%%% Changes from base to side #1
-original content
+side 1 content
+++++++ Contents of side #2
side 2 content
>>>>>>> Conflict 1 of 1 ends
```

#### 2. `snapshot` style
```
<<<<<<< Conflict 1 of 1
%%%%%%% Base
original content
+++++++ Side #1
side 1 content
+++++++ Side #2
side 2 content
>>>>>>> Conflict 1 of 1 ends
```

#### 3. `git` style (familiar but limited)
```
<<<<<<< Side #1
side 1 content
=======
side 2 content
>>>>>>> Side #2
```

### Programmatic Conflict Detection
```bash
# Find all commits with conflicts
jj log -r 'conflicts()'

# Find conflicts in specific paths
jj log -r 'conflicts() & files("src/")'

# Show conflict details
jj show -r 'conflicts()' --no-pager

# Check if current working copy has conflicts
jj st | grep -q "Warning: There are unresolved conflicts"
```

### Automated Conflict Resolution Strategies
```bash
# Strategy 1: Always take one side
resolve_take_side1() {
  local file="$1"
  sed -i '
    /<<<<<<< Conflict/,/>>>>>>> Conflict.*ends/c\
    # Take side 1 content here
  ' "$file"
}

# Strategy 2: Parse and merge programmatically
resolve_conflict_programmatically() {
  local file="$1"
  # Extract conflict sections
  awk '/<<<<<<< Conflict/,/>>>>>>> Conflict.*ends/{
    if(/%%%%%%% Changes from base to side #1/) { in_side1=1; next }
    if(/\+\+\+\+\+\+\+ Contents of side #2/) { in_side1=0; in_side2=1; next }
    if(/>>>>>>> Conflict.*ends/) { in_side1=0; in_side2=0; next }
    if(in_side1) print "SIDE1:", $0
    if(in_side2) print "SIDE2:", $0
  }' "$file"
}

# Strategy 3: Use external merge tools
jj diff --tool=meld -r 'conflicts()'  # If configured
```

### Multi-way Conflicts
```bash
# 3+ way conflicts show as:
# <<<<<<< Conflict 1 of 1
# %%%%%%% Changes from base to side #1
# +side 1 changes
# +++++++ Contents of side #2
# side 2 content
# +++++++ Contents of side #3
# side 3 content
# >>>>>>> Conflict 1 of 1 ends

# Handle programmatically by parsing all sides
parse_multiway_conflict() {
  local file="$1"
  local current_side=""
  while IFS= read -r line; do
    case "$line" in
      "%%%%%%% Changes from base to side #"*)
        current_side="base_changes"
        ;;
      "+++++++ Contents of side #"*)
        current_side="side_$(echo "$line" | grep -o '#[0-9]*' | tr -d '#')"
        ;;
      ">>>>>>> Conflict"*"ends")
        current_side=""
        ;;
      *)
        if [[ -n "$current_side" ]]; then
          echo "$current_side: $line"
        fi
        ;;
    esac
  done < "$file"
}
```

### Working with Conflicted Commits
```bash
# Don't resolve conflicts in-place - create resolution commits
jj new 'conflicts()'  # Create working copy on conflicted commit
# Edit files to resolve conflicts
jj squash  # Move resolution into conflicted commit

# Batch resolve multiple conflicts
for commit in $(jj log -r 'conflicts()' --no-graph --template='change_id'); do
  jj new "$commit"
  # Apply automated resolution
  jj squash -m "Auto-resolve conflicts in $commit"
done
```

### 5. Viewing History
```bash
# Git
git log --oneline --graph --all

# jj
jj log  # Shows relevant commits by default
jj log -r 'all()'  # Shows all commits
```

## Best Practices for AI Agents

1. **Always check status first**: Use `jj st` to understand current state
2. **Use change IDs for stability**: Prefer change IDs over commit IDs in scripts
3. **Leverage automatic rebasing**: Don't worry about orphaned commits
4. **Use operation log for recovery**: `jj op log` and `jj undo` for mistakes
5. **Understand revsets**: Learn the query language for powerful selections
6. **Work with conflicts**: Don't avoid them; jj handles them gracefully
7. **⚠️ NEVER use interactive commands**: Always use file-specific or non-interactive variants
8. **Specify files explicitly**: Use `jj squash <files>` instead of `jj squash -i`
9. **Edit conflicts directly**: Parse conflict markers programmatically, don't use `jj resolve`

## Common Gotchas

1. **Bookmarks don't move**: Must manually update after creating commits
2. **Empty commits allowed**: Working copy can be empty (unlike Git)
3. **No detached HEAD**: Always on a commit, never "detached"
4. **Automatic file tracking**: New files are tracked automatically
5. **Root commit exists**: Virtual commit at repository root (`root()`)
6. **⚠️ Interactive commands fail**: Will hang or error in automated environments
7. **Conflict markers are different**: Not the same format as Git's `<<<<<<<` markers

## Error Handling Patterns for AI Agents

### Common Error Categories and Solutions

#### 1. Immutable Commit Errors
```bash
# Error: "Commit X is immutable"
# Solution: Use --ignore-immutable flag or work with descendants

# Check if commit is immutable
jj log -r 'immutable() & <revision>'

# Work around immutable commits
jj rebase -d <new_dest> -s <source> --ignore-immutable
jj duplicate <immutable_commit> -d <new_dest>  # Create copy instead
```

#### 2. Divergent Change Errors
```bash
# Error: "Change X has diverged"
# Solution: Resolve divergence explicitly

# Find divergent changes
jj log -r 'divergent()'

# Resolve divergence by abandoning one version
jj abandon <unwanted_commit_id>

# Or combine divergent changes
jj new <divergent_commit1> <divergent_commit2>  # Merge them
```

#### 3. Empty Commit Handling
```bash
# jj allows empty commits by default, but you might want to detect them

# Find empty commits
jj log -r 'empty()'

# Remove empty commits
jj abandon 'empty() & mine()'

# Prevent empty commits in workflows
check_empty_commit() {
  if jj log -r '@' --template='empty' | grep -q 'true'; then
    echo "Warning: Creating empty commit"
    return 1
  fi
}
```

#### 4. Bookmark Conflicts
```bash
# Error: "Bookmark X is conflicted"
# Solution: Resolve bookmark conflicts manually

# Find conflicted bookmarks
jj bookmark list | grep -E '\?\?\?'

# Resolve by choosing one side
jj bookmark move <bookmark_name> --to <chosen_commit>

# Or create new bookmark
jj bookmark create <new_name> -r <commit>
jj bookmark delete <conflicted_name>
```

#### 5. Large Repository Issues
```bash
# Error: "Revset evaluation too expensive"
# Solution: Use more specific revsets

# Instead of expensive operations
jj log -r 'all()'  # Can be slow on large repos

# Use more specific queries
jj log -r '::@ | @::'  # Ancestors and descendants only
jj log -r 'mine() & author_date(after:"1 week ago")'  # Recent work only

# Use --limit to cap results
jj log -r 'all()' --limit 100
```

#### 6. Push/Pull Errors
```bash
# Error: "No such remote bookmark"
# Solution: Check remote configuration

# List remotes
jj git remote list

# Track remote bookmarks
jj bookmark track <bookmark>@<remote>

# Handle authentication errors
jj git fetch --all-remotes  # Test connectivity
```

### Defensive Programming Patterns

#### Pre-flight Checks
```bash
# Check repository state before operations
preflight_checks() {
  # Check for conflicts
  if jj log -r 'conflicts()' --template='commit_id' | grep -q '.'; then
    echo "Error: Repository has conflicts"
    return 1
  fi
  
  # Check for divergent changes
  if jj log -r 'divergent()' --template='commit_id' | grep -q '.'; then
    echo "Error: Repository has divergent changes"
    return 1
  fi
  
  # Check working copy state
  if jj st | grep -q 'Warning:'; then
    echo "Error: Working copy has warnings"
    return 1
  fi
  
  return 0
}
```

#### Safe Operation Wrappers
```bash
# Wrapper for risky operations
safe_rebase() {
  local source="$1"
  local dest="$2"
  
  # Create checkpoint
  local checkpoint=$(jj op log --limit 1 --template='operation.id')
  
  # Attempt operation
  if ! jj rebase -s "$source" -d "$dest"; then
    echo "Rebase failed, restoring checkpoint"
    jj op restore "$checkpoint"
    return 1
  fi
  
  # Verify result
  if jj log -r 'conflicts()' --template='commit_id' | grep -q '.'; then
    echo "Rebase created conflicts, restoring checkpoint"
    jj op restore "$checkpoint"
    return 1
  fi
  
  return 0
}

# Wrapper for commit operations
safe_commit() {
  local message="$1"
  shift
  local files=("$@")
  
  # Check for uncommitted changes
  if jj st | grep -q 'No changes'; then
    echo "No changes to commit"
    return 1
  fi
  
  # Commit with files if specified
  if [ ${#files[@]} -gt 0 ]; then
    jj commit -m "$message" "${files[@]}"
  else
    jj commit -m "$message"
  fi
}
```

#### Error Recovery Patterns
```bash
# Automatic error recovery
recover_from_error() {
  local error_type="$1"
  
  case "$error_type" in
    "immutable")
      echo "Attempting to work around immutable commit"
      jj rebase --ignore-immutable "${@:2}"
      ;;
    "divergent")
      echo "Attempting to resolve divergence"
      jj abandon "$(jj log -r 'divergent()' --template='commit_id' | head -1)"
      ;;
    "conflicts")
      echo "Attempting automated conflict resolution"
      # Apply automated resolution strategy
      auto_resolve_conflicts
      ;;
    *)
      echo "Unknown error type: $error_type"
      return 1
      ;;
  esac
}
```

#### Validation Functions
```bash
# Validate repository state
validate_repo_state() {
  local issues=0
  
  # Check for problems
  if jj log -r 'conflicts()' --template='commit_id' | grep -q '.'; then
    echo "ISSUE: Repository has conflicts"
    ((issues++))
  fi
  
  if jj log -r 'divergent()' --template='commit_id' | grep -q '.'; then
    echo "ISSUE: Repository has divergent changes"
    ((issues++))
  fi
  
  if jj log -r 'empty() & mine()' --template='commit_id' | grep -q '.'; then
    echo "WARNING: You have empty commits"
  fi
  
  return $issues
}
```

## Filesets for Precise Operations

### Basic Fileset Syntax
```bash
# Basic patterns
jj squash "src/"              # Directory
jj squash "*.py"              # Extension
jj squash "glob:test_*.py"    # Glob pattern
jj squash "regex:.*\.test\..*" # Regex pattern

# Path prefixes
jj squash "root:src/"         # From repository root
jj squash "cwd:./local/"      # From current directory
```

### Advanced Fileset Operations
```bash
# Combine patterns with operators
jj squash "src/ | tests/"     # Union: src OR tests
jj squash "src/ & glob:*.py"  # Intersection: src AND *.py
jj squash "src/ ~ glob:*.test" # Difference: src EXCEPT *.test

# Negation
jj squash "~glob:*.tmp"       # Everything except .tmp files
jj squash "all() ~ (glob:*.pyc | glob:*.pyo)" # Exclude compiled Python

# Complex combinations
jj squash "(src/ | lib/) & ~glob:*test*"  # Source files, no tests
jj squash "glob:*.{py,js,ts} & ~node_modules/" # Code files, no deps
```

### AI-Specific Fileset Patterns
```bash
# Code organization patterns
CODE_FILES="glob:*.{py,js,ts,java,cpp,c,h,hpp,rs,go}"
TEST_FILES="glob:*{test,spec}*.{py,js,ts} | glob:test_*"
CONFIG_FILES="glob:*.{json,yaml,yml,toml,ini,cfg,conf}"
DOC_FILES="glob:*.{md,rst,txt,doc,docx}"

# Use in operations
jj squash "$CODE_FILES & src/"     # Code files in src
jj split "$TEST_FILES"             # Split out test files
jj abandon "($CODE_FILES | $TEST_FILES) & glob:*tmp*"  # Remove temp files

# Build artifacts
BUILD_ARTIFACTS="glob:*.{o,pyc,pyo,class,jar,exe,dll,so,dylib}"
jj file untrack "$BUILD_ARTIFACTS"
```

### Working with File States
```bash
# File status queries
jj file list                  # All tracked files
jj file list --revision @-    # Files in parent commit
jj file list "src/"          # Files in directory

# File changes
jj diff --name-only "src/"    # Changed files in src
jj diff --name-only "glob:*.py" # Changed Python files

# Selective operations on file states
jj squash "$(jj diff --name-only 'glob:*.py')"  # Squash Python changes
```

### Bulk File Operations
```bash
# Process multiple file patterns
bulk_squash() {
  local patterns=("$@")
  for pattern in "${patterns[@]}"; do
    if jj diff --name-only "$pattern" | grep -q .; then
      jj squash "$pattern" -m "Update $(basename "$pattern")"
    fi
  done
}

# Usage
bulk_squash "src/*.py" "tests/*.py" "docs/*.md"

# Conditional file operations
squash_if_exists() {
  local pattern="$1"
  local message="$2"
  if jj file list "$pattern" | grep -q .; then
    jj squash "$pattern" -m "$message"
  fi
}
```

### Configuration-Based Filesets
```bash
# Define common patterns in config
# .jj/config.toml
[template-aliases]
code = 'files(glob:"*.{py,js,ts,java,cpp,c,h,hpp,rs,go}")'
tests = 'files(glob:"*{test,spec}*.{py,js,ts}") | files(glob:"test_*")'
docs = 'files(glob:"*.{md,rst,txt}")'

# Use in commands
jj log --template=code       # Show commits touching code
jj squash 'files(glob:"*.py") & ~files(glob:"*test*")'  # Python non-test files
```

### Repository Structure Patterns
```bash
# Monorepo patterns
FRONTEND="packages/frontend/ | apps/web/"
BACKEND="packages/backend/ | apps/api/"
SHARED="packages/shared/ | lib/"

# Microservice patterns
SERVICE_A="services/auth/ | services/user/"
SERVICE_B="services/payment/ | services/order/"

# By responsibility
FEATURES="src/features/ | components/"
UTILS="src/utils/ | src/helpers/"
TESTS="src/**/*test* | tests/ | __tests__/"

# Use in operations
jj squash "$FRONTEND & glob:*.ts"    # Frontend TypeScript
jj rebase -s "commits touching $BACKEND" -d main  # Rebase backend changes
```

## File Operations

### Tracking Files
```bash
# Git
git add <file>              → (automatic in jj)
git rm <file>               → rm <file>
git mv <old> <new>          → mv <old> <new>

# Untracking
git rm --cached <file>      → jj file untrack <file>

# Selective tracking with filesets
jj file track "src/ & ~glob:*.tmp"     # Track src except temps
jj file untrack "glob:*.{log,tmp,cache}" # Untrack by pattern
```

### Ignoring Files
- Same `.gitignore` format supported
- Use `jj file untrack` for already-tracked files
- Configure `snapshot.auto-track` for automatic tracking patterns
- Use filesets for complex ignore patterns in config

## Common AI Agent Workflows

### 1. Automated Code Review
```bash
# Find recent changes by team members
review_recent_changes() {
  local days="${1:-7}"
  local authors="${2:-all}"
  
  if [ "$authors" = "all" ]; then
    jj log -r "author_date(after:\"$days days ago\")" --template='
    {
      "commit": commit_id,
      "author": author.email(),
      "date": author_date,
      "message": description.first_line(),
      "files": files.map(|f| f.path()),
      "changes": files.len()
    }'
  else
    jj log -r "author_date(after:\"$days days ago\") & author(glob:\"*$authors*\")" --template='json()'
  fi
}

# Analyze code patterns
find_potential_issues() {
  # Find large commits
  jj log -r 'mine() & author_date(after:"1 week ago")' --template='
  if(files.len() > 10,
    "LARGE: " ++ description.first_line() ++ " (" ++ files.len() ++ " files)",
    ""
  )' | grep -v "^$"
  
  # Find commits with potential issues
  jj log -r 'description(regex:"(fix|bug|issue|problem|error)")' --template='
  "ISSUE: " ++ description.first_line() ++ " (" ++ commit_id.short() ++ ")"'
}
```

### 2. Automated Branch Management
```bash
# Clean up old branches
cleanup_branches() {
  local cutoff_days="${1:-30}"
  
  # Find stale bookmarks
  jj bookmark list --template='
  {
    "name": name,
    "target": target,
    "last_commit_date": target.author_date()
  }' | jq -r '
  select(.last_commit_date < (now - ('$cutoff_days' * 24 * 60 * 60))) | 
  .name
  ' | while read bookmark; do
    echo "Deleting stale bookmark: $bookmark"
    jj bookmark delete "$bookmark"
  done
}

# Sync with remote
sync_with_remote() {
  local remote="${1:-origin}"
  
  # Fetch all changes
  jj git fetch --remote "$remote"
  
  # Update tracking bookmarks
  jj bookmark list --template='
  if(remote_targets.contains("'$remote'"),
    name,
    ""
  )' | grep -v "^$" | while read bookmark; do
    if jj bookmark list "$bookmark" | grep -q "ahead"; then
      echo "Bookmark $bookmark is ahead of remote"
    else
      jj bookmark move "$bookmark" --to "$bookmark@$remote"
    fi
  done
}
```

### 3. Automated Testing Integration
```bash
# Run tests on specific changes
test_changes() {
  local base_commit="${1:-main}"
  
  # Get list of changed files
  local changed_files=$(jj diff --from "$base_commit" --name-only)
  
  # Run tests for affected components
  for file in $changed_files; do
    case "$file" in
      src/*)
        echo "Running tests for $file"
        pytest "tests/$(basename "$file" .py)_test.py" 2>/dev/null || true
        ;;
      *.js|*.ts)
        echo "Running JS tests for $file"
        npm test -- --testPathPattern="$(basename "$file" .js)" 2>/dev/null || true
        ;;
    esac
  done
}

# Pre-commit validation
validate_commit() {
  local commit="${1:-@}"
  
  # Check commit message format
  local msg=$(jj log -r "$commit" --template='description.first_line()')
  if ! echo "$msg" | grep -qE '^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+'; then
    echo "ERROR: Commit message doesn't follow conventional format"
    return 1
  fi
  
  # Check for conflicts
  if jj log -r "$commit" --template='conflicts' | grep -q 'true'; then
    echo "ERROR: Commit has unresolved conflicts"
    return 1
  fi
  
  # Check for large files
  if jj diff -r "$commit" --stat | awk '{print $3}' | grep -qE '^[0-9]{4,}$'; then
    echo "WARNING: Commit contains large files"
  fi
  
  return 0
}
```

### 4. Automated Release Management
```bash
# Prepare release
prepare_release() {
  local version="$1"
  local base_branch="${2:-main}"
  
  # Create release branch
  jj new "$base_branch" -m "Prepare release $version"
  jj bookmark create "release/$version"
  
  # Generate changelog
  jj log -r "$base_branch..@" --template='
  "- " ++ description.first_line() ++ " (" ++ change_id.short() ++ ")"
  ' > CHANGELOG_$version.md
  
  # Update version files
  find . -name "package.json" -o -name "pyproject.toml" -o -name "Cargo.toml" | while read file; do
    case "$file" in
      */package.json)
        jq '.version = "'$version'"' "$file" > "$file.tmp" && mv "$file.tmp" "$file"
        ;;
      */pyproject.toml)
        sed -i 's/version = ".*"/version = "'$version'"/' "$file"
        ;;
      */Cargo.toml)
        sed -i 's/version = ".*"/version = "'$version'"/' "$file"
        ;;
    esac
  done
  
  jj commit -m "Bump version to $version"
}

# Tag release
tag_release() {
  local version="$1"
  local commit="${2:-@}"
  
  # Create annotated tag (using git in co-located repo)
  git tag -a "v$version" -m "Release version $version" "$(jj log -r "$commit" --template='commit_id')"
  
  # Push to remote
  jj git push --bookmark "release/$version"
  git push origin "v$version"
}
```

### 5. Automated Code Analysis
```bash
# Find code hotspots
find_hotspots() {
  local days="${1:-90}"
  
  # Files changed most frequently
  jj log -r "author_date(after:\"$days days ago\")" --template='
  files.map(|f| f.path()).join("\n")
  ' | sort | uniq -c | sort -nr | head -20
  
  # Authors by activity
  jj log -r "author_date(after:\"$days days ago\")" --template='
  author.email()
  ' | sort | uniq -c | sort -nr
}

# Technical debt analysis
analyze_technical_debt() {
  # Find TODO/FIXME comments
  jj log -r 'diff_contains("TODO") | diff_contains("FIXME") | diff_contains("XXX")' --template='
  "DEBT: " ++ description.first_line() ++ " (" ++ commit_id.short() ++ ")"
  '
  
  # Find large functions/classes (approximate)
  jj log -r 'files(glob:"*.py") | files(glob:"*.js") | files(glob:"*.java")' --template='
  files.map(|f| f.path()).join("\n")
  ' | sort -u | while read file; do
    if [ -f "$file" ]; then
      local line_count=$(wc -l < "$file")
      if [ "$line_count" -gt 500 ]; then
        echo "LARGE FILE: $file ($line_count lines)"
      fi
    fi
  done
}
```

### 6. Automated Dependency Management
```bash
# Update dependencies
update_dependencies() {
  local commit_each="${1:-false}"
  
  # Update package.json dependencies
  if [ -f "package.json" ]; then
    npm update
    if [ "$commit_each" = "true" ]; then
      jj commit -m "Update npm dependencies" package.json package-lock.json
    fi
  fi
  
  # Update Python dependencies
  if [ -f "requirements.txt" ]; then
    pip list --outdated --format=json | jq -r '.[] | .name' | while read pkg; do
      pip install --upgrade "$pkg"
    done
    pip freeze > requirements.txt
    if [ "$commit_each" = "true" ]; then
      jj commit -m "Update Python dependencies" requirements.txt
    fi
  fi
  
  # Update Rust dependencies
  if [ -f "Cargo.toml" ]; then
    cargo update
    if [ "$commit_each" = "true" ]; then
      jj commit -m "Update Rust dependencies" Cargo.toml Cargo.lock
    fi
  fi
}

# Security audit
security_audit() {
  # Check for known vulnerabilities
  if [ -f "package.json" ]; then
    npm audit --json | jq -r '.vulnerabilities | keys[]' | while read vuln; do
      echo "NPM VULNERABILITY: $vuln"
    done
  fi
  
  # Check for sensitive patterns in recent commits
  jj log -r 'author_date(after:"1 week ago")' --template='
  files.map(|f| f.path()).join("\n")
  ' | sort -u | while read file; do
    if [ -f "$file" ]; then
      if grep -qE '(password|secret|key|token).*=.*["\'][^"\']{8,}["\']' "$file"; then
        echo "POTENTIAL SECRET: $file"
      fi
    fi
  done
}
```

## Integration Notes

- **Co-located repos**: Can use both `jj` and `git` commands in same repo
- **Git compatibility**: Full interoperability with Git repositories
- **GitHub integration**: Works with GitHub through Git backend
- **Existing tools**: Most Git tools work with jj repositories
- **CI/CD integration**: Use jj commands in automated pipelines
- **Monitoring**: Track repository health with operation log analysis

## AI Agent Workflow Template

```bash
# Safe workflow pattern for AI agents:
jj st                           # Always check status first
jj new <base_commit>           # Create new working commit
# Edit files directly
jj describe -m "commit message" # Set description
jj squash <specific_files>     # Move specific files if needed
jj bookmark create <name>      # Create bookmark if needed
jj log                         # Verify results
```

## Operation Log for Debugging and Recovery

### Understanding Operations
Every jj command creates an operation that records:
- What changed (commits, bookmarks, working copy)
- When it happened (timestamp)
- Who did it (user, hostname)
- How it was done (command + arguments)

### Basic Operation Commands
```bash
# View operation history
jj op log                      # Recent operations
jj op log --limit 20           # More operations
jj op log --at-op=<id>         # Operations at specific point

# Show operation details
jj op show <operation_id>      # What changed in this operation
jj op diff <operation_id>      # Diff between before/after operation

# Time-based queries
jj op log --after="2024-01-01"  # Operations after date
jj op log --before="1 hour ago" # Operations before time
```

### Advanced Operation Debugging
```bash
# See repository state at specific operation
jj log --at-op=<operation_id>   # Commit log at that time
jj status --at-op=<operation_id> # Working copy state then
jj bookmark list --at-op=<operation_id> # Bookmarks at that time

# Compare states across operations
jj diff --from-op=<old_op> --to-op=<new_op>  # Changes between operations

# Find specific types of operations
jj op log --template='
if(operation.type == "push",
  "PUSH: " ++ operation.description,
  if(operation.type == "rebase",
    "REBASE: " ++ operation.description,
    operation.description
  )
)'
```

### Recovery Strategies
```bash
# Undo operations (most common)
jj undo                        # Undo last operation
jj op undo <operation_id>      # Undo specific operation

# Restore to specific point in time
jj op restore <operation_id>   # Restore to exact state

# Selective recovery
jj op restore <operation_id> --what=bookmarks    # Only restore bookmarks
jj op restore <operation_id> --what=working-copy # Only restore working copy

# Find and recover lost commits
jj op log --template='operation.description' | grep -i "abandon"
jj op undo <abandon_operation_id>  # Recover abandoned commits
```

### Handling Concurrent Operations
```bash
# When multiple operations happen simultaneously
jj op log --graph             # Show operation graph
jj op log --template='
operation.id ++ " " ++ 
operation.user ++ "@" ++ operation.hostname ++ " " ++
operation.description'

# Resolve operation conflicts
jj op abandon <operation_id>   # Abandon problematic operation
jj op restore <good_operation_id> # Restore to known good state
```

### AI-Specific Operation Patterns
```bash
# Create checkpoints before risky operations
create_checkpoint() {
  local checkpoint_id=$(jj op log --limit 1 --template='operation.id')
  echo "Checkpoint: $checkpoint_id" > .jj_checkpoint
}

# Restore from checkpoint
restore_checkpoint() {
  local checkpoint_id=$(cat .jj_checkpoint)
  jj op restore "$checkpoint_id"
}

# Automated operation monitoring
monitor_operations() {
  while true; do
    jj op log --limit 1 --template='
    operation.timestamp ++ " " ++
    operation.user ++ " " ++
    operation.description
    ' >> operation_log.txt
    sleep 60
  done
}
```

## Emergency Recovery

```bash
# Quick recovery patterns
jj op log                      # View recent operations
jj undo                        # Undo last operation
jj op undo <operation_id>      # Undo specific operation

# Nuclear option - restore to last known good state
jj op restore <operation_id>   # Complete state restoration

# Find and recover specific items
jj op log --template='operation.description' | grep -E '(abandon|delete|remove)'
```

## Troubleshooting Guide for AI Agents

### Common Issues and Solutions

#### 1. "Command not found" Errors
```bash
# Issue: jj command not found
# Solution: Check installation and PATH
which jj                      # Should show path to jj binary
echo $PATH | grep -q jj       # Check if jj is in PATH

# Alternative: Use full path
/usr/local/bin/jj status      # If installed via package manager
```

#### 2. "No repository found" Errors
```bash
# Issue: Not in a jj repository
# Solution: Initialize or navigate to repo
jj workspace root 2>/dev/null || echo "Not in jj repo"

# Navigate to repo root
cd "$(jj workspace root 2>/dev/null || echo ".")"

# Initialize if needed
jj git init --colocate        # For existing git repo
jj git init                   # For new repo
```

#### 3. Authentication Issues
```bash
# Issue: Cannot push/pull from remote
# Solution: Check authentication
jj git fetch --all-remotes --dry-run  # Test without changes

# Check git credentials (in co-located repo)
git config --list | grep credential
git config credential.helper           # Should show credential helper

# Manual credential test
git ls-remote origin                   # Test git access
```

#### 4. Performance Issues
```bash
# Issue: Commands are slow
# Solution: Optimize repository

# Check repository size
du -sh .jj/                    # jj repository size
jj log -r 'all()' --template='commit_id' | wc -l  # Total commits

# Enable performance features
jj config set core.fsmonitor true
jj config set core.watchman true      # If watchman is installed

# Use more specific revsets
jj log -r '::@ | @::' --limit 100     # Instead of 'all()'
```

#### 5. Bookmark/Branch Issues
```bash
# Issue: Bookmark doesn't exist
# Solution: Check and create bookmarks
jj bookmark list               # List all bookmarks
jj bookmark list --all         # Include remote bookmarks

# Create missing bookmark
jj bookmark create main -r 'git_head()'  # From git HEAD
jj bookmark track main@origin  # Track remote bookmark

# Fix bookmark conflicts
jj bookmark list | grep -E '\?\?\?'  # Find conflicts
jj bookmark move conflicted_name --to chosen_commit
```

#### 6. Working Copy Issues
```bash
# Issue: Working copy is corrupted
# Solution: Reset working copy
jj workspace update-stale      # Update stale working copy
jj new @                       # Create new working copy commit

# Check working copy status
jj debug working-copy          # Debug working copy state
jj file list                   # Check tracked files
```

#### 7. Operation Log Issues
```bash
# Issue: Operation log is corrupted
# Solution: Recover from operation log
jj op log --limit 10           # Check recent operations
jj op abandon <bad_operation>  # Abandon corrupted operation
jj op restore <good_operation> # Restore to known good state
```

### Debugging Workflow

#### Step 1: Environment Check
```bash
debug_environment() {
  echo "=== Environment Check ==="
  echo "jj version: $(jj --version)"
  echo "Git version: $(git --version 2>/dev/null || echo 'Not installed')"
  echo "Working directory: $(pwd)"
  echo "Repository root: $(jj workspace root 2>/dev/null || echo 'Not in repo')"
  echo "User: $(jj config get user.name 2>/dev/null || echo 'Not configured')"
  echo "Email: $(jj config get user.email 2>/dev/null || echo 'Not configured')"
}
```

#### Step 2: Repository Health Check
```bash
check_repo_health() {
  echo "=== Repository Health Check ==="
  
  # Check for common issues
  local issues=0
  
  # Conflicts
  local conflicts=$(jj log -r 'conflicts()' --template='commit_id' | wc -l)
  if [ "$conflicts" -gt 0 ]; then
    echo "WARNING: $conflicts commits have conflicts"
    ((issues++))
  fi
  
  # Divergent changes
  local divergent=$(jj log -r 'divergent()' --template='commit_id' | wc -l)
  if [ "$divergent" -gt 0 ]; then
    echo "WARNING: $divergent divergent changes"
    ((issues++))
  fi
  
  # Empty commits
  local empty=$(jj log -r 'empty() & mine()' --template='commit_id' | wc -l)
  if [ "$empty" -gt 0 ]; then
    echo "INFO: $empty empty commits (may be intentional)"
  fi
  
  # Working copy warnings
  if jj st 2>&1 | grep -q "Warning:"; then
    echo "WARNING: Working copy has warnings"
    jj st 2>&1 | grep "Warning:"
    ((issues++))
  fi
  
  echo "=== Health Check Complete: $issues issues found ==="
  return $issues
}
```

#### Step 3: Operation Audit
```bash
audit_recent_operations() {
  echo "=== Recent Operations Audit ==="
  
  # Show recent operations with details
  jj op log --limit 10 --template='
  operation.id ++ " " ++ 
  operation.user ++ " " ++
  operation.description ++ " " ++
  if(operation.type == "error", " [ERROR]", "")
  '
  
  # Check for failed operations
  local failed=$(jj op log --limit 50 --template='
  if(operation.type == "error", operation.id, "")
  ' | grep -v "^$" | wc -l)
  
  if [ "$failed" -gt 0 ]; then
    echo "WARNING: $failed failed operations in recent history"
  fi
}
```

### Recovery Procedures

#### Full Repository Recovery
```bash
full_recovery() {
  echo "=== Starting Full Recovery ==="
  
  # Create backup
  local backup_dir="/tmp/jj_backup_$(date +%s)"
  cp -r .jj "$backup_dir"
  echo "Backup created at: $backup_dir"
  
  # Find last known good operation
  local good_op=$(jj op log --limit 20 --template='
  if(operation.type == "error", "", operation.id)
  ' | grep -v "^$" | head -1)
  
  if [ -n "$good_op" ]; then
    echo "Restoring to operation: $good_op"
    jj op restore "$good_op"
  else
    echo "No good operation found, manual intervention needed"
    return 1
  fi
  
  echo "=== Recovery Complete ==="
}
```

#### Selective Recovery
```bash
selective_recovery() {
  local component="$1"  # bookmarks, working-copy, commits
  
  case "$component" in
    bookmarks)
      echo "Recovering bookmarks..."
      jj bookmark list --all
      # Restore bookmarks from remote
      jj git fetch --all-remotes
      ;;
    working-copy)
      echo "Recovering working copy..."
      jj workspace update-stale
      jj new @
      ;;
    commits)
      echo "Recovering commits..."
      jj log -r 'all()' --limit 20
      ;;
    *)
      echo "Unknown component: $component"
      return 1
      ;;
  esac
}
```

## Performance Optimization

### Repository Optimization
```bash
# Reduce revset complexity
optimize_revsets() {
  # Use specific revsets instead of expensive ones
  echo "=== Revset Optimization ==="
  
  # Good patterns
  echo "✓ Use: jj log -r '::@' --limit 50"
  echo "✓ Use: jj log -r 'mine() & author_date(after:\"1 week ago\")'"
  echo "✓ Use: jj log -r 'bookmarks()'"
  
  # Avoid expensive patterns
  echo "✗ Avoid: jj log -r 'all()' (without limit)"
  echo "✗ Avoid: jj log -r 'ancestors(all())'"
  echo "✗ Avoid: Complex regex patterns without file filters"
}

# Enable performance features
enable_performance_features() {
  echo "=== Enabling Performance Features ==="
  
  # Filesystem monitoring
  jj config set core.fsmonitor true
  echo "✓ Enabled filesystem monitoring"
  
  # Watchman (if available)
  if command -v watchman >/dev/null 2>&1; then
    jj config set core.watchman true
    echo "✓ Enabled Watchman"
  fi
  
  # Disable colors for better parsing performance
  jj config set ui.color never
  echo "✓ Disabled colors"
  
  # Disable pager for automation
  jj config set ui.paginate never
  echo "✓ Disabled pager"
}
```

### Memory Management
```bash
# Monitor memory usage
monitor_memory() {
  echo "=== Memory Usage Monitoring ==="
  
  # Check repository size
  echo "Repository size: $(du -sh .jj/ | cut -f1)"
  
  # Check commit count
  local commit_count=$(jj log -r 'all()' --template='commit_id' | wc -l)
  echo "Total commits: $commit_count"
  
  # Memory usage during operations
  /usr/bin/time -v jj log -r 'all()' --template='commit_id' >/dev/null 2>&1 | grep "Maximum resident set size"
}
```

This comprehensive guide provides the essential differences and mental model adjustments needed for AI agents to effectively work with Jujutsu instead of Git, with specific emphasis on avoiding interactive commands that will fail in automated environments, complete with troubleshooting procedures and performance optimization techniques.