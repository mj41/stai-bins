# MJ AI Tools

A collection of GitHub Copilot tools and utilities for Visual Studio Code AI-related development.

## Tools Overview

### aicmd

AiCmd is a Go-based command wrapper designed specifically for VS Code Copilot AI interactions. It addresses the limitation that VS Code Copilot can only use one terminal session and hanging commands break AI progress.

[Full documentation](./aicmd.md)

### aicont

AiCont is a containerized task execution system that creates Docker environments for AI development workflows. It sets up isolated containers with mounted workspace directories and manages task execution through aicmd and aicont-runner.

[Full documentation](./aicont.md)

### aienv

Tool will be run inside Docker container. Will list whole directory tree structure with filesystem permissions (in some rwx format for user, group, others) as json.

[Full documentation](./aienv.md)

### aiterm

- **Enhanced Logging**: Comprehensive logging with JSON metadata for all command executions

[Full documentation](./aiterm.md)

## Usage

All tools are available in the `stai-tools/bin/` directory and support:

- `--version` - Show version information
- `--help` - Show usage information
- `--readme` - Show full documentation

