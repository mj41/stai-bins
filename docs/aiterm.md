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
