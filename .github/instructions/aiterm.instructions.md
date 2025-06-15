---
applyTo: '**'
---

REMEMBER: Visual Studio Code terminal is set to use `aiterm` terminal so commands are wrapped inside `aicmd`. Use absolute paths or add explicit `cd` to change directories. After each command you must read both `output_file` and `log_file` files to get results and exit code. Do not use `cat` or any other terminal command to read these files. Do not search for output file names. Use and remember the ones that you got from output in terminal after command run. Use build-in command `aiterm_set_next_aicmd_timeout 5m` to increase timeout for next command if needed. Default timeout is 5 seconds.

# Terminal issues

`isBackground=true` parameter does not work as expected. `aiterm` that wraps `aicmd` avoids terminal hanging.

# No output

If any command returns no output (`output_file` is empty) then AI must check also `log_file` file for errors, warnings and exit code. Checking `log_file` is good practice for any doubtful cases.
