# MJ AI Tools - Internal Tools Complete Documentation

This file contains the complete documentation for all tools.

## Table of Contents

- [aicont-runner](#aicont-runner)

---

# AiCont Runner - AI Container Task Executor

AiCont Runner monitors task definition directories and executes commands through aicmd with comprehensive logging and status tracking. It's designed to run inside containerized environments for automated AI development workflows.

Tool name: `aicont-runner`.

## Description

AiCont Runner is a task executor that watches for task definition files and executes them using aicmd.
It runs inside AI containers and processes task definitions to execute commands with proper logging and status tracking.

## Parameters

- `--task-defs-dir=<path>` - Path to task definitions directory (default: /aicont/tasks-defs-ro)
- `--task-runs-dir=<path>` - Path to task runs directory (default: /aicont/tasks-runs-rw)
- `--suffix=<string>` - Unique suffix string [a-zA-Z0-9_-] (4-32 chars) (required)
- `--watch` - Watch for new task definition files continuously
- `--version` - Show version information
- `--help` - Show usage information
- `--readme` - Show this documentation

## Examples

```bash
aicont-runner --suffix=test123
aicont-runner --suffix=test123 --watch
aicont-runner --task-defs-dir=./tasks --task-runs-dir=./runs --suffix=test123
```

## Task Definition File Format

Task definition files contain simple commands that will be executed using aicmd:

```
echo ABCD
-- echo ABCD  
--timeout=10s -- echo "ABCD"
```

## Task Status Format

Status is tracked in JSON files:

```
task-run-001.json, task-run-002.json, etc.
```

## Use Cases

- Processing AI task queues in containerized environments
- Automated task execution with structured logging
- Container-based CI/CD pipeline execution
- Batch processing of AI development tasks


---

