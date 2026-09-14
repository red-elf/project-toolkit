# commit-fully — Recursive Submodule Commit & Push

## Overview

`commit-fully` is a deterministic script that recursively commits and pushes all submodules and the main repository to all configured upstreams. It ensures:

1. All submodules are initialized and cloned
2. Every submodule is on a branch (not detached HEAD)
3. Changes are committed with a consistent message
4. All commits are pushed to all configured upstreams
5. All remotes are verified to have received the commits

## Usage

```bash
# From project root
./commit-fully "Your commit message"

# Or with explicit project root
COMMIT_FULLY_ROOT=/path/to/project ./commit-fully "Your commit message"

# Short alias
./commit_fully "Your commit message"
```

## Features

### Submodule Initialization
- Runs `git submodule update --init --recursive` to ensure all submodules are cloned
- Detects and initializes uninitialized submodules

### Branch Management
- Detects detached HEAD state in submodules
- Automatically checks out the default branch from origin (main/master)
- Creates local tracking branch if needed

### Commit & Push
- Commits all changes in each submodule with the provided message
- Uses the project's `push_all` script for multi-upstream push
- Falls back to manual push if `push_all` not available
- Pushes tags

### Verification
- Verifies all remotes received the commits via `git ls-remote`
- Reports success/failure for each remote
- Fails if any remote is missing commits

## Configuration

### Upstreams
Configure upstreams in the `Upstreams/` directory (or `upstreams/` per §11.4.29):

```bash
# Upstreams/GitHub.sh
export UPSTREAMABLE_REPOSITORY="git@github.com:user/repo.git"
export UPSTREAMABLE_PRIMARY=1

# Upstreams/GitFlic.sh
export UPSTREAMABLE_REPOSITORY="git@gitflic.ru:user/repo.git"
```

Run `./Upstreamable/install_upstreams.sh` to configure remotes.

### Push All
The `push_all` script reads upstreams and pushes to all remotes. Place at project root or in `Upstreamable/push_all`.

## Constitution Compliance

- **§2.1 Multi-upstream push is the norm** — Pushes to all configured upstreams
- **§3 Submodule changes propagate through submodule commits first** — Processes submodules before main repo
- **§1.1 Mutation-paired gates** — Verification gate proves commits reached all upstreams
- **§11.4.2 Recorded-evidence requirement** — Captures exact commands, exit codes, raw output
- **§11.4.6 No-guessing mandate** — Uses captured evidence, no guessing

## Example Output

```
[INFO] Starting commit-fully from project root: /path/to/project
[INFO] Commit message: feat: add new feature

[INFO] Ensuring submodules are initialized and cloned...
[SUCCESS] All submodules initialized

[INFO] === Processing submodules ===
[INFO] Processing submodules in myproject:
[INFO] → Processing submodule: submodule1
[INFO] Repository at /path/to/project/submodule1 is on branch: main
[INFO] No changes to commit in submodule1

[INFO] === Processing main repository ===
[INFO] Committing changes in myproject with message: feat: add new feature
[SUCCESS] Committed successfully
[SUCCESS] Pushed to all upstreams

[INFO] === Verifying all upstreams received commits ===
[INFO] Verifying github...
[SUCCESS]   github: OK
[INFO] Verifying gitflic...
[SUCCESS]   gitflic: OK
[INFO] Verifying submodule1 -> github...
[SUCCESS]   submodule1 -> github: OK
[INFO] Verifying submodule1 -> gitflic...
[SUCCESS]   submodule1 -> gitflic: OK

[SUCCESS] === ALL COMMITS VERIFIED ON ALL UPSTREAMS ===
[SUCCESS] commit-fully completed successfully!
```

## Integration with llmctl

The llmctl project uses commit-fully for all commits:

```bash
# From llmctl root
./commit-fully "feat: add new feature"

# Or from any directory
COMMIT_FULLY_ROOT=/path/to/llmctl /path/to/project_toolkit/commit-fully "feat: add new feature"
```

All validation/verification submodules in `constitution/submodules/` are automatically processed.

## Anti-Bluff Guarantee

The verification step is paired with a meta-test mutation (`meta_test_verification.sh`) that proves the verification gate catches regressions:

1. **Mutation 1**: Remove a required submodule → verification must FAIL
2. **Mutation 2**: Corrupt verification logic → verification must FAIL

Both mutations are caught and correctly fail the gate, proving it's not a bluff gate.
