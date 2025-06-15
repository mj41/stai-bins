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
