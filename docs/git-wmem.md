# git-wmem - Git Working Memory

`git-wmem` is a suite of tools designed to help developers manage work-in-progress across multiple git repositories. It allows you to snapshot the state of your workspace, including uncommitted changes, and restore or analyze them later.

## Overview

`git-wmem` acts as a "working memory" for your development environment. It solves the problem of context switching when working with multiple repositories. Instead of manually stashing changes in each repo or creating temporary "wip" commits, `git-wmem` captures the entire state of your workspace in a single operation.

## Features

- **Multi-Repo Support**: Tracks and manages state across multiple git repositories simultaneously.
- **Snapshot Capability**: Captures uncommitted changes (staged and unstaged), untracked files, and current branch information.
- **Efficient Storage**: Uses a dedicated storage mechanism to keep snapshots organized and retrievable.
- **History Tracking**: Maintains a log of snapshots, allowing you to review past states.
- **Performance Profiling**: Built-in CPU and memory profiling for performance analysis.

## Usage

### Basic Commands

```bash
# Initialize a new wmem repository
git-wmem init <directory>

# Save the current state of tracked repositories
git-wmem commit

# View the history of saved states
git-wmem log
```

### Version Information
```bash
git-wmem --version
```

### Show Documentation
```bash
git-wmem --readme   # Show full documentation
```

## Command Line Options

- `--cpuprofile=<file>`: Write cpu profile to the specified file.
- `--memprofile=<file>`: Write memory profile to the specified file.
- `--readme`: Show full documentation.
- `--version`: Show version information.
- `--help`: Show usage information.

## Examples

```bash
# Initialize wmem in the current directory
git-wmem init .

# Save a snapshot of your current work
git-wmem commit

# Check the log of saved snapshots
git-wmem log

# Run with profiling enabled
git-wmem --cpuprofile=cpu.prof commit
```

## Documentation

See `docs` directory in the source repository for more details:
- [use-cases](https://github.com/mj41/git-wmem/blob/main/docs/use-cases.md)
- [data-structures](https://github.com/mj41/git-wmem/blob/main/docs/data-structures.md)
- [dictionary](https://github.com/mj41/git-wmem/blob/main/docs/dictionary.md)
- [boundaries](https://github.com/mj41/git-wmem/blob/main/docs/boundaries.md)
- [validations](https://github.com/mj41/git-wmem/blob/main/docs/validations.md)
- [optimizations](https://github.com/mj41/git-wmem/blob/main/docs/optimizations.md)

