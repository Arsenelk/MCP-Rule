# PowerShell Script Rules

## Purpose

This file records PowerShell-specific script generation and editing rules. Read it only when creating, reviewing, or modifying `.ps1` files.

## Rules

1. Separate adjacent hashtable elements in PowerShell arrays with commas, including multi-line literals.
2. Avoid fragile one-line nested literals when generating scripts; choose explicit multi-line structure.
3. Do not point `RedirectStandardOutput` and `RedirectStandardError` to the same file in `Start-Process`.
4. After creating or modifying a script, verify the key edited section.
5. Run a PowerShell syntax parse check when feasible before handing the script back to the user.
