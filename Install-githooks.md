# Install Git hooks (VS Code)

## Purpose

This helps keep working trees clean after switching branches.

## Quick (VS Code)

1. Open this repository in VS Code.
2. Press Ctrl+Shift+P and choose "Tasks: Run Task".
3. Select "Install Git hooks".

The task is defined in `.vscode/tasks.json` and on Windows will run the PowerShell installer `scripts/install-githooks.ps1`.

## Description
- `post-checkout` — after a branch checkout this hook runs `git clean -fdX` to remove files and directories that are ignored by Git. 
This helps keep working trees clean after switching branches.
