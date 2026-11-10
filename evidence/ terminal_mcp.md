# Part 7 — MCP Server Integration Evidence

**Date:** November 9, 2026  
**Tool:** Claude Code  
**MCP Server:** a4-tools  
**Workspace:** `runs/terminal/workspace`

## MCP Server Configuration

I configured Claude Code to use the MCP server created in Part 5.

Command:

```bash
claude mcp add --scope project --transport stdio a4-tools -- "$PWD/.venv/bin/python" "$PWD/server.py" --workspace runs/terminal/workspace
```

I verified the connection using:

```bash
claude mcp get a4-tools
```

The server reported `Status: Connected` and used the project configuration in `.mcp.json`.

Inside Claude Code, I also ran `/mcp` and confirmed that `a4-tools` was connected.

## Testing the MCP Tools

I first asked Claude:

"Use the a4-tools MCP server to call list_files and show me which files are available in the workspace. Do not modify any files."

Claude successfully called `a4-tools.list_files` and returned:

- discounts.py
- spec.md
- tests/__init__.py
- tests/test_discounts.py

This confirmed that Claude could communicate with my MCP server.

## Read-Only Review

I then instructed Claude to use the MCP tools to inspect `spec.md`, `discounts.py`, and `tests/test_discounts.py` without modifying any files.

Claude used `a4-tools.read_file` to inspect the files.

It identified two implementation problems:

1. Discount codes were case-sensitive, violating Rule 2.
2. Expired discount codes were still accepted, violating Rule 4.

Claude also identified missing visible test coverage for expiration dates, the last valid day, and mixed-case discount codes.

## Fixing the Implementation Through MCP

After the review, I asked Claude to fix the identified issues using only the a4-tools MCP server.

Claude proposed two separate edits to `discounts.py`.

**Edit 1:** Added `code = code.upper()` to support case-insensitive discount codes.

**Edit 2:** Added an expiration check that raises `ValueError` when `today > last_day`.

I reviewed and approved each edit individually through Claude Code's permission prompts.

Claude used `a4-tools.edit_file` to apply the changes.

## Test Results

After completing the edits, Claude used `a4-tools.read_file` to inspect the implementation and `a4-tools.run_tests` to run the visible tests.

The MCP tool returned:

`PASSED: 4 of 4 tests passed.`

## Review and Conclusion

The MCP server successfully exposed the tools I implemented in Part 1 to Claude Code.

Claude was able to list files, read source code, edit files, and run tests using my MCP server.

The approval prompts allowed me to review each proposed modification before it was applied.

This experiment demonstrated that my custom tools could be used by an external coding agent without requiring changes to their original implementations.