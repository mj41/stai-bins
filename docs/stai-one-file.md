# stai-bins - Complete Documentation

This file contains the complete documentation for all tools.

## Table of Contents

- [aicmd](#aicmd)
- [aiterm](#aiterm)
- [git-wmem](#git-wmem)

---

# aicmd - AI Command Wrapper

aicmd is a Go-based command wrapper designed specifically for VS Code Copilot AI interactions. It addresses the limitation that VS Code Copilot can only use one terminal session and hanging commands break AI progress.

## Overview

aicmd acts as a controller/supervisor process that spawns and monitors child processes, handles I/O redirection, enforces timeouts, and provides structured logging. It ensures no orphaned processes and provides robust process management for AI workflows.

## Features

- **Process Supervision**: Spawns and monitors child processes with PID tracking
- **Timeout Management**: Enforces execution time limits with graceful termination
- **Background Execution**: Daemonizes processes for long-running tasks
- **I/O Handling**: Dual output (console + timestamped log files) for stdout/stderr
- **Security**: Only binary executables allowed, shell commands rejected
- **Process Validation**: Ensures child processes are completely terminated before exit
- **Structured Logging**: Timestamped logs with PID tracking for debugging and audit

## Usage

### Basic Command Execution
```bash
./aicmd --timeout=<duration> --work-dir=<path> <command> [args...]
```

### With Stdin File
```bash
# First create input file, then use it
echo "test input" > /tmp/input.txt
./aicmd --timeout=2m --work-dir=./temp --stdin-file=/tmp/input.txt <command> [args...]
```

### Version Information
```bash
./aicmd --version
```

### Show Documentation
```bash
./aicmd --readme   # Show full documentation
```

## Command Line Options

- `--timeout=<duration>`: **Required** - Maximum execution time (range: 1s-1h)
- `--work-dir=<path>`: **Required** - Working directory for command execution
- `--stdin-file=<path>`: File to use as stdin input for the command
- `--log-dir=<path>`: Directory for log files (defaults to --work-dir if not specified)
- `--outs-dir=<path>`: Directory to capture stdout/stderr files with auto-generated names
- `--file-suffix=<suffix>`: Optional suffix for both log and output file names
- `--watch=<path>`: Watch file for changes and restart command automatically
- `--no-date-in-name`: Generate file names without date/time prefix
- `--verbose`: Enable verbose logging to console and run in foreground mode
- `--silent`: Suppress all output (no process information displayed)
- `--debug`: Enable debug output including child task JSON dump
- `--no-background`: Run in foreground mode (background is default)
- `--json`: Enable JSON-formatted logging
- `--wait`: Wait for process completion even in background mode
- `--json-meta`: Enable JSON format for metadata output (process info)
- `--version`: Display version information
- `--readme`: Show full documentation

## Examples

```bash
# Basic command execution (background by default)
./aicmd --timeout=5s --work-dir=./temp echo "Hello World"

# Command with timeout and working directory (background by default)
./aicmd --timeout=10s --work-dir=./temp ping google.com

# Command with stdin file (background by default)
echo "test input" > /tmp/input.txt
./aicmd --timeout=1m --work-dir=./temp --stdin-file=/tmp/input.txt cat

# Command with output capture to custom directory (background by default)
./aicmd --timeout=30s --work-dir=./temp --outs-dir=./temp/outputs go test ./...

# Command with file suffix for organization (background by default)
./aicmd --timeout=2m --work-dir=./temp --file-suffix=build go build

# Command with custom log directory and file suffix
./aicmd --timeout=2m --work-dir=./temp --log-dir=./custom/logs --file-suffix=test go test ./...

# Command with clean file names (no date/time prefix)
./aicmd --timeout=2m --work-dir=./temp --file-suffix=clean --no-date-in-name go build

# Running with verbose output in foreground
./aicmd --verbose --timeout=30s --work-dir=./temp go test ./...

# Background execution (process runs detached)
./aicmd --timeout=10m --work-dir=./temp go run ./server.go

# Background with output capture and organization
./aicmd --timeout=5m --work-dir=./temp --outs-dir=./temp/outputs --file-suffix=build go build ./...

# Running other stai-bins binaries
./aicmd --timeout=30s --work-dir=./temp ../aijson/aijson --help

# Output JSON metadata about the process
./aicmd --json-meta --timeout=30s --work-dir=./temp echo "Hello World"

# Output full JSON with process details and command output
./aicmd --json --timeout=30s --work-dir=./temp echo "Hello World"

# Enable debug logging for troubleshooting
./aicmd --debug --timeout=30s --work-dir=./temp echo "Debug mode"

# Explicitly run in silent mode (default behavior)
./aicmd --silent --timeout=30s --work-dir=./temp echo "Silent execution"

# Watch mode - monitor and restart if process exits early
./aicmd --watch --timeout=60s --work-dir=./temp sleep 10

# Wait for background process to complete before parent exits
./aicmd --wait --timeout=2m --work-dir=./temp go build
```

## JSON Output Modes

aicmd supports JSON output for programmatic integration:

### JSON Metadata (`--json-meta`)
Outputs process information in JSON format:
```bash
./aicmd --json-meta --timeout=30s --work-dir=./temp echo "Hello World"
```
Returns structured data about the process (PID, start time, file paths, etc.) without command output.

### Full JSON (`--json`)
Outputs complete process information including command output:
```bash
./aicmd --json --timeout=30s --work-dir=./temp echo "Hello World"
```
Returns comprehensive JSON with process metadata and captured stdout/stderr.

### JSON with File Capture
Combine JSON output with file organization:
```bash
./aicmd --json --timeout=30s --work-dir=./temp --outs-dir=./outputs --file-suffix=api echo "API test"
```

## Debug and Watch Modes

### Debug Mode (`--debug`)
Enable detailed debug logging for troubleshooting:
```bash
./aicmd --debug --timeout=30s --work-dir=./temp echo "Debug information"
```
Provides verbose internal logging for diagnosing issues.

### Watch Mode (`--watch`)
Monitor and automatically restart processes that exit early:
```bash
./aicmd --watch --timeout=60s --work-dir=./temp sleep 10
```
Useful for services that should remain running throughout the timeout period.

### Silent Mode (`--silent`)
Explicitly enable silent operation (default behavior):
```bash
./aicmd --silent --timeout=30s --work-dir=./temp echo "No console output"
```
Suppresses all console output except errors.

## Background Mode

Background mode is **enabled by default**. Commands run as daemon processes that don't block the terminal. Use `--no-background` or `--verbose` to disable.

The `--no-background` flag runs commands in foreground mode. The `--verbose` flag also automatically enables foreground mode. This is useful for:

- Quick informational commands (version, help)
- Commands where you need immediate output
- Debugging with --verbose flag
- Sequential commands that depend on previous output

### Background Mode Features

- Process is detached from terminal and runs independently
- All output is captured to log files in the working directory
- Parent process returns immediately with PID information
- Child process runs with timeout protection
- Automatic fallback for environments with restricted process controls

### Background Examples

```bash
# Start a development server in background (default)
./aicmd --timeout=30m --work-dir=./project npm run dev

# Run tests in foreground with output capture
./aicmd --no-background --timeout=10m --work-dir=./project --outs-dir=./project/test-results go test ./...

# Long-running build process in background (default)
./aicmd --timeout=1h --work-dir=./project make all

# Quick version check in foreground
./aicmd --no-background --timeout=10s --work-dir=./project go version
```

When using background mode (default), the parent process will display:
```
Background process started with PID: 12345
Logs will be written to: /path/to/workdir/logs/
```

When using foreground mode (`--no-background`), the command runs and waits for completion.

The `--wait` flag allows you to run in background mode but still wait for the process to complete before the parent process exits. This is useful when you want the process to be detached but need to know when it finishes:

```bash
# Background mode with wait - process runs detached but parent waits for completion
./aicmd --wait --timeout=5m --work-dir=./project go build

# Normal background mode - parent returns immediately
./aicmd --timeout=5m --work-dir=./project go build
```

## Logging

aicmd creates timestamped log files in the `logs/` directory (or custom `--log-dir`) with the format:
```
logs/aicmd_YYYY-MM-DD_HH-MM-SS.mmm_PID.log
logs/aicmd_YYYY-MM-DD_HH-MM-SS.mmm_PID_suffix.log (with --file-suffix)
logs/aicmd_PID_suffix.log (with --no-date-in-name --file-suffix)
logs/aicmd_PID.log (with --no-date-in-name)
```

Output files are created in the specified `--outs-dir` with matching naming:
```
outputs/aicmd_YYYY-MM-DD_HH-MM-SS.mmm_PID.out
outputs/aicmd_YYYY-MM-DD_HH-MM-SS.mmm_PID_suffix.out (with --file-suffix)
outputs/aicmd_PID_suffix.out (with --no-date-in-name --file-suffix)
outputs/aicmd_PID.out (with --no-date-in-name)
```

The PID ensures unique file names when multiple aicmd processes run simultaneously.

**Silent by Default**: aicmd runs silently by default. All output is logged to files but not displayed on console unless `--verbose` flag is used.

Log format:
```
YYYY-MM-DD HH:MM:SS.mmm [aicmd:<SUPERVISOR_PID>] <message>
YYYY-MM-DD HH:MM:SS.mmm [aicmd:<SUPERVISOR_PID>] chpid=<CHILD_PID> src=<stdout|stderr> log-line=<content>
```

To see output on console, use `--verbose`:
```bash
./aicmd --verbose --timeout=30s --work-dir=./temp go test ./...
```

## File Organization

aicmd provides comprehensive file organization options:

### Log Directory Control
Use `--log-dir` to specify a custom directory for log files (defaults to `--work-dir/logs/`):
```bash
./aicmd --timeout=2m --work-dir=./temp --log-dir=./custom/logs go build
```

### Output File Capture
Use `--outs-dir` to capture stdout/stderr to organized output files:
```bash
./aicmd --timeout=30s --work-dir=./temp --outs-dir=./temp/outputs go test ./...
```

### File Naming Options
Use `--file-suffix` to organize related commands:
```bash
./aicmd --timeout=2m --work-dir=./temp --file-suffix=build go build
./aicmd --timeout=1m --work-dir=./temp --file-suffix=test go test
```

Use `--no-date-in-name` for cleaner file names without timestamps:
```bash
./aicmd --timeout=2m --work-dir=./temp --file-suffix=clean --no-date-in-name go build
# Creates: logs/aicmd_clean.log and outputs/aicmd_clean.out (if --outs-dir specified)
```

### Combined Organization
All options can be combined for comprehensive file management:
```bash
./aicmd --timeout=5m --work-dir=./temp --log-dir=./logs --outs-dir=./outputs --file-suffix=integration --no-date-in-name go test ./...
```

## Security Restrictions

- Only binary executables are allowed
- Shell commands are rejected (no pipes `|`, redirections `<>`, semicolons `;`, etc.)
- Command validation using PATH lookup
- No shell interpretation or expansion

## Nested Invocation

aicmd supports nested invocation - running aicmd within another aicmd process. The child aicmd processes automatically have access to a clean environment without the parent's security restrictions, enabling seamless nested command execution.

### Example
```bash
# Parent aicmd can invoke child aicmd without issues
./aicmd --verbose -- ./aicmd --timeout=30s echo "Nested execution works"
```

This capability is useful for:
- Testing new version fo aicmd encapsulated by older aicmd
- Allow AI to follow instructions when parent aicmd could be started only in backgroud (and with other well speficied parameters) for auditing

## Build Instructions

```bash
cd $HOME/work/stai-tools/cmd/aicmd
go build -o aicmd
```


---

# aiterm - Terminal Shell for AI

aiterm is a custom terminal shell written in Go that serves as a replacement for bash in VS Code terminals. It intercepts all user commands and executes them through `aicmd` for enhanced logging and monitoring capabilities.

## Features

- **Command Interception**: All user commands are intercepted and executed through `aicmd`
- **Enhanced Logging**: Comprehensive logging with JSON metadata for all command executions
- **VS Code Integration**: Advanced VS Code terminal integration with 633 escape sequences for command tracking
- **Environment Management**: Automatic creation and management of working and log directories
- **Interactive Shell**: Provides a familiar shell experience with built-in commands
- **Timeout Management**: Support for setting custom timeouts for individual commands
- **Signal Handling**: Proper Ctrl+C handling to prevent accidental shell termination
- **Auto-detection**: Automatic detection of VS Code environment for optimal integration

## Installation

Build the binary:
```bash
go build -o aiterm ./cmd-bins/aiterm
```

## Usage

```bash
aiterm --aitask-temp <directory> [options]
```

### Required Parameters

- `--aitask-temp <directory>`: Directory for aiterm temporary files (must exist)

### Optional Parameters

- `--colors`: Enable colored terminal prompt (green)
- `--no-vscode-integration`: Disable VS Code terminal integration (enabled by default)
- `--help`: Show help message
- `--readme`: Show this documentation
- `--version`: Show version information

## How It Works

```
User Input → aiterm Shell → aicmd wrapper → Original Command
```

Every command you type is executed through `aicmd` with proper logging and metadata collection.

## Directory Structure

```
$AITASK_TEMP/
├── aiwd/           # Working directory for command execution
└── ailog/          # Log directory for aicmd outputs
```

## Environment Variables

- `AITASK_TEMP`: Base directory for aicmd working directories (set automatically from --aitask-temp parameter)
- `AICMD_PATH`: Path to aicmd binary (optional, searches PATH if not set)
- `TERM_PROGRAM`: Used to auto-detect VS Code environment
- `VSCODE_SHELL_INTEGRATION`: Used to auto-detect VS Code integration support

## Example Usage

```bash
# Create a temporary directory
mkdir -p /tmp/aitask

# Run aiterm with required parameter
./aiterm --aitask-temp /tmp/aitask

# Run with colored prompt
./aiterm --aitask-temp /tmp/aitask --colors

# Run without VS Code integration
./aiterm --aitask-temp /tmp/aitask --no-vscode-integration
```

## VS Code Configuration

Add to your VS Code settings.json:

```json
{
    "terminal.integrated.profiles.linux": {
        "aiterm": {
            "path": "/path/to/aiterm",
            "args": ["--aitask-temp", "/tmp/aitask", "--colors"],
            "env": {}
        }
    },
    "terminal.integrated.defaultProfile.linux": "aiterm"
}
```

Note: The `--aitask-temp` directory must exist before starting VS Code.

## Built-in Commands

- `help` - Show help message with available commands
- `exit` - Exit the shell
- `aiterm_set_next_aicmd_timeout <duration>` - Set timeout for the next command only
  - Examples: `aiterm_set_next_aicmd_timeout 5m`, `aiterm_set_next_aicmd_timeout 30s`

All other commands are executed through aicmd with enhanced logging capabilities.

## VS Code Integration Features

When running in VS Code, aiterm provides:

- **Command Tracking**: Uses VS Code 633 escape sequences to track command execution
- **Working Directory Reporting**: Automatically reports directory changes to VS Code
- **Command Nonce**: Generates unique identifiers for each command execution
- **Exit Code Reporting**: Reports command success/failure status to VS Code
- **Auto-detection**: Automatically detects VS Code environment using `TERM_PROGRAM` and `VSCODE_SHELL_INTEGRATION`

## Command Execution Format

All commands are executed through aicmd with the following format:
```
aicmd --work-dir=$AITASK_TEMP/aiwd --outs-dir=$AITASK_TEMP/ailog --log-dir=$AITASK_TEMP/ailog --json-meta [--timeout=<duration>] -- sh -c 'your-command'
```


---

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



---

