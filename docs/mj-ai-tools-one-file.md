# MJ AI Tools - Complete Documentation

This file contains the complete documentation for all tools.

## Table of Contents

- [aicmd](#aicmd)
- [aicont](#aicont)
- [aienv](#aienv)
- [aiterm](#aiterm)

---

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


---

# AiCont - AI Container Wrapper

AiCont is a containerized task execution system that creates Docker environments for AI development workflows. It sets up isolated containers with mounted workspace directories and manages task execution through aicmd and aicont-runner.

Tool name: `aicont`.

## Parameters
- `--ws-json-file` ... path to workspace file (referenced as `ws-json` later). Example file content is in `ws-json-example.json`.
- `--suffix` ... simple string [a-zA-Z0-9_-] minimum length 4, maximum length 32.
- `--no-container-rm` ... optional flag to disable automatic container removal when it exits. By default, containers are automatically removed with the `--rm` Docker/Podman option to prevent accumulation of stopped containers.

## Directory Structure

The tool will prepare this directory structure inside the `ai-temp` workspace folder. The folder name `ai-temp` is mandatory and must exist in the workspace.

Example: `./bin/aicont --ws-json-file ws-json-example.json --suffix xhdyT4Rd` will create `../ai-temp/aicont-xhdyT4Rd-run` subdirectory inside the existing `ai-temp` directory. This subdirectory is later referenced as `aicont-subdir`. The `aicont-subdir` must not exist at the start of the tool execution, meaning the `--suffix` used must be a unique string not used since the last `ai-temp` prune/cleanup.

Inside `aicont-subdir`, the following structure will be created:
- `super-aicont/`
- `aicont/ai-tasks-defs/` 
- `aicont/ai-tasks-runs/`

## Docker Image Preparation

`aicont` will create a Dockerfile inside `../ai-temp/aicont-$suffix$-prep` (e.g., `ai-temp/aicont-xhdyT4Rd-prep` for the suffix `xhdyT4Rd` from the example above) to build a new image.

The image will be based on the latest official Go Docker image.

### Container Directory Structure

Inside the container, there will be these directories:
- `/bin/` - for included binaries with both `aicmd` and `aicont-runner` binaries

### Mounted Directories

From the local directory, the following directories will be mounted:
- `/super-aicont-rw` - local workspace directory `super-aicont` from `aicont-subdir` mounted in read-write mode
- `/aicont/tasks-defs-ro` - local workspace directory `aicont/ai-tasks-defs` from `aicont-subdir` mounted in read-only mode
- `/aicont/tasks-runs-rw` - local workspace directory `aicont/ai-tasks-runs` from `aicont-subdir` mounted in read-write mode
- `/ws/$workspace-folder-name$-ro` - local workspace folders mounted in read-only mode for each workspace folder (where `$workspace-folder-name$` is a placeholder for the real workspace folder name)

Example: In case of `ws-json-example.json` (with five folders), it will create `/ws/go-mc`, `/ws/go-mc-mj`, `/ws/stai-tools`, and `/ws/stai-tools-src` (four mounts in read-only mode).

**Note:** The workspace folder `ai-temp` will be skipped as its subdirectories are already mounted elsewhere.

## Container Execution

`aicont` will:
1. Build the container image
2. Start the container with directories mounted as specified above
3. Name the container `aicont-$suffix$`

**Constraints:**
- No such container should already be running
- Maximum number of containers running/started with prefix `aicont-` is 3

After starting, the Docker container will:
1. Run environment analysis using `aienv`
2. Start `aicont-runner` in the background to watch for task definitions
3. Keep the container alive using `tail -f /dev/null`

The container runs `aicont-runner` directly (not wrapped by aicmd) in watch mode, monitoring the `/aicont/tasks-defs-ro` folder for new `.task` files.

## Task Runner (`aicont-runner`)

The second tool `aicont-runner` (aka `runner`) will:
1. Watch the `/aicont/tasks-defs-ro` folder
2. Ensure each new task is started exactly once
3. Save task progress as a new JSON file inside `/aicont/tasks-runs-rw` (referenced as `task-run-status-file`) including assigned `task-run-num`
4. Track `task-run-status` statuses: `starting`, `running`, `finished`, `failed`

## Task Management

The process outside that started `aicont` will be able to add new files with `.task` suffix to the `aicont/ai-tasks-defs` subdirectory of `aicont-subdir`. Tasks will be started by `runner` using `aicmd` with `--work-dir` set to `/aicont/tasks-runs-rw/$task-dir$/`. `$task-dir$` is created by `aicmd`. The `$task-dir$` name follows the pattern `tr$task-run-num$-$suffix$` (e.g., `tr001-xhdyT4Rd`). `$task-run-num$` is a sequence starting from one with leading zeros added to make the length 3. `runner` will start tasks as new definitions come in using `aicmd` background mode. That means they will finish at different times. `runner` will need to parse `aicmd` standard JSON output. `aicmd` should support the `--json` parameter for background mode. Paths and info provided will be added to `task-run-status-file`. `runner` will also get the `aicmd` `task-run-process-id` and add it to `task-run-status-file`. `runner` will remember all running `task-run-process-id` values and check statuses to be able to update `task-run-status`. `task-run-status-file` will not contain any output or logs as these will be referenced.

## Task Definition File Format

**Allowed:**
- `echo ABCD`
- `-- echo ABCD`
- `--timeout=10s -- echo "ABCD"`

**Error:**
- `--timeout=10s --work-dir=/tmp/abc -- echo "ABCD"`
- `--timeout=10s --whatever-else -- echo "ABCD"`

Only the `timeout` parameter is allowed. The value is matched by regexp `^\d+[smh]$`. The rest of the validation is done by `aicmd`.

## Task Status Format

```json
{
  "task-run-num": "001",
  "task-run-status": "running",
  "task-run-process-id": 12345,
  "aicmd-log-path": "/path/to/log",
  "aicmd-outs-path": "/path/to/outs",
  "start-time": "2024-01-01T12:00:00Z",
  "end-time": null
}
```

## Command Execution Chain

- User uses aicont to start container
- aicont starts docker container  
- aicont ends execution and provides list of paths to the user
- Docker container starts aienv for environment analysis
- Docker container starts aicont-runner in watch mode
- aicont-runner waits for the first task definition
- User creates task definition by e.g., `echo '--timeout=10s -- echo "ABCD"' > my-taskX.task` 
- aicont-runner detects new task and starts aicmd inside docker for the provided task definition

## Errors 

- aicont fails immediately due to three containers already running

## Container Cleanup

By default, containers are started with the `--rm` option and will be automatically removed when they exit. This helps prevent accumulation of stopped containers. If you need to preserve the container for debugging or inspection, use the `--no-container-rm` flag.

When using `--no-container-rm`, the container runs indefinitely using `tail -f /dev/null` to keep it alive. Users can stop containers manually using `docker stop aicont-<suffix>` or `docker kill aicont-<suffix>`. The container can also be removed with `docker rm aicont-<suffix>` after stopping.

## Example Usage

```bash
./bin/aicont --ws-json-file ws-json-example.json --suffix xhdyT4Rd
```

## Implementation Details

### Docker Image Naming
The Docker image follows the pattern `aicont:<suffix>` (e.g., `aicont:xhdyT4Rd`).

### Task File Processing
- Tasks execute in parallel (all at once) as they are detected
- Multiple `.task` files added simultaneously will be processed concurrently
- Task files remain in the definitions directory after processing for audit purposes

### Status Monitoring
- `aicont-runner` checks for new tasks every 2 seconds
- Process status is monitored by checking if the PID is still running
- Status updates are written immediately to the task status files

### Task Directory Creation
- Task directories are created by aicmd before starting command execution
- Directory names follow the pattern `tr<task-run-num>-<suffix>` (e.g., `tr001-xhdyT4Rd`)

### Container Resource Management
- No specific resource limits are applied by default
- Container runs with user namespace mapping (`--userns=keep-id`) for proper file permissions
- Maximum of 3 aicont containers can run simultaneously
- Containers are automatically removed when they exit (`--rm` flag) unless `--no-container-rm` is specified

### Error Handling
- If `aicont-runner` crashes, the container continues running
- Container logs can be checked using `docker logs aicont-<suffix>`
- Failed tasks are marked with status "failed" in their status files

### User Output Information
When aicont finishes successfully, it provides:
- Path to add task files (ai-tasks-defs directory)
- Path to monitor status files (ai-tasks-runs directory) 
- Container name for Docker commands
- Log file location


---

# AiEnv - AI Environment Scanner

AiEnv scans and lists directory tree structures with detailed filesystem permissions in JSON format. It's designed to run inside Docker containers to provide comprehensive environment analysis for AI development workflows.

## Description

Tool will be run inside Docker container. Will list whole directory tree structure with filesystem permissions (in some rwx format for user, group, others) as json.

## Usage

```bash
aienv [options]
```

## Features

- Recursive directory scanning
- Detailed permission analysis
- JSON output format
- Container-optimized operation

---

# AiTerm - AI-Enhanced Terminal Shell

AiTerm is a custom terminal shell written in Go that serves as a replacement for bash in VS Code terminals. It intercepts all user commands and executes them through `aicmd` for enhanced logging and monitoring capabilities.

## Features

- **Command Interception**: All user commands are intercepted and executed through `aicmd`
- **Enhanced Logging**: Comprehensive logging with JSON metadata for all command executions
- **VS Code Integration**: Designed to work seamlessly as a custom terminal shell in VS Code
- **Environment Management**: Automatic creation and management of working and log directories
- **Interactive Shell**: Provides a familiar shell experience with built-in commands

## Installation

Build the binary:
```bash
go build -o aiterm ./cmd/aiterm
```

## Configuration

Replace your default shell in VS Code with aiterm:

1. Build the binary: `go build -o aiterm ./cmd/aiterm`
2. Configure VS Code terminal to use aiterm as default shell
3. Set required environment variables

## How It Works

```
User Input → AiTerm Shell → aicmd wrapper → Original Command
```

Every command you type is executed through `aicmd` with proper logging and metadata collection.

## Directory Structure

```
$AITASK_TEMP/
├── aiwd/           # Working directory for command execution
└── ailog/          # Log directory for aicmd outputs
```

## Environment Variables

- `AITASK_TEMP`: Base directory for aicmd working directories (required)
- `AICMD_PATH`: Path to aicmd binary (optional, searches PATH if not set)

## Example Usage

```bash
# Set environment variables
export AITASK_TEMP=/tmp/aitask

# Run aiterm
./aiterm
```

## VS Code Configuration

Add to your VS Code settings.json:

```json
{
    "terminal.integrated.profiles.linux": {
        "aiterm": {
            "path": "/path/to/aiterm",
            "args": [],
            "env": {
                "AITASK_TEMP": "/tmp/aitask"
            }
        }
    },
    "terminal.integrated.defaultProfile.linux": "aiterm"
}
```

## Built-in Commands

- `help` - Show help message
- `exit` - Exit the shell

All other commands are executed through aicmd with enhanced logging capabilities.


---

