# AiCmd - AI Command Wrapper

AiCmd is a Go-based command wrapper designed specifically for VS Code Copilot AI interactions. It addresses the limitation that VS Code Copilot can only use one terminal session and hanging commands break AI progress.

## Overview

AiCmd acts as a controller/supervisor process that spawns and monitors child processes, handles I/O redirection, enforces timeouts, and provides structured logging. It ensures no orphaned processes and provides robust process management for AI workflows.

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

# Running other stai-tools binaries
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

AiCmd supports JSON output for programmatic integration:

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

AiCmd creates timestamped log files in the `logs/` directory (or custom `--log-dir`) with the format:
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

**Silent by Default**: AiCmd runs silently by default. All output is logged to files but not displayed on console unless `--verbose` flag is used.

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

AiCmd provides comprehensive file organization options:

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

AiCmd supports nested invocation - running aicmd within another aicmd process. The child aicmd processes automatically have access to a clean environment without the parent's security restrictions, enabling seamless nested command execution.

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
cd $HOME/work/stai-tools-src/cmd/aicmd
go build -o aicmd
```

## AI Integration

For AI usage instructions, see: `$HOME/work/stai-tools/ai-instructions/terminal-commands.md`
