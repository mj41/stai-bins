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