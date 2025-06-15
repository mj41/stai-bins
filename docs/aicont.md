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
