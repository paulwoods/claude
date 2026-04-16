---
name: woods-windows-conventions
description: >
  Instructs Claude Code to use Windows-compatible paths, shell syntax,
  and environment conventions in all tool calls and code generation.
version: 1.0.0
tags: os, windows, paths, shell
---

# Windows OS Conventions

## File Paths
- Always use backslash `\` as the path separator.
- Use Windows-style absolute paths (e.g., `C:\Projects\myapp`).
- Do not generate Unix-style paths (e.g., `/home/user/myapp`).

## Shell Commands
- Default to PowerShell syntax for all shell commands.
- Do not use bash or Unix shell syntax.
- Do not use Unix commands (`ls`, `grep`, `cat`, `rm`, `touch`).

## Windows equivalents

| Unix command     | Windows / PowerShell equivalent  |
|------------------|----------------------------------|
| `ls`             | `dir` or `Get-ChildItem`         |
| `cat`            | `type` or `Get-Content`          |
| `rm`             | `del` or `Remove-Item`           |
| `cp`             | `copy` or `Copy-Item`            |
| `mv`             | `move` or `Move-Item`            |
| `touch`          | `New-Item`                       |
| `export VAR=`    | `$env:VAR =`                     |
| `./mvnw`         | `.\mvnw`                         |

## Environment Variables
- Use `$env:VARIABLE` syntax in PowerShell.
- Use `%VARIABLE%` syntax in cmd.

## Line Endings
- Be aware that Windows uses `\r\n` line endings.
- Do not enforce Unix `\n` line endings in generated scripts or config files.
