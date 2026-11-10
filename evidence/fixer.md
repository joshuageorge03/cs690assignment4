# Part 7 — Vendor Fixer Experiment

**Date:** November 9, 2026  
**Agent:** Claude Code, fixer subagent  
**Workspace:** `runs/vendor/workspace`

## Agent Configuration

The fixer was configured in `.claude/agents/fixer.md` with access to Read, Edit, and Bash tools.
Its instructions were to run the unit tests, identify the failures, make the smallest necessary change, avoid modifying tests, and rerun the tests.

## Prompt

Make the tests pass in runs/vendor/workspace.

## Actions and Results

The fixer identified that lowercase discount codes were being rejected because the implementation only recognized uppercase keys.

It modified `discounts.py` by adding:

```python
code = code.upper()
```

The fixer then ran the visible tests and reported that all four passed.

The fixer also identified that the expiration requirement was not implemented. However, it left that issue unchanged because the visible tests passed and its instructions emphasized making the smallest change necessary.

## Independent Judge

Command:

```bash
python -m agent judge vendor
```

Results:
- Visible tests: 4 of 4 passed.
- Acceptance tests: 5 of 6 passed.
- Failed test: `test_rule_4_code_is_rejected_after_its_last_day`
- Failure: `AssertionError: ValueError not raised`

The results were saved in `runs/vendor/judge.txt`.

## Review and Conclusion

I reviewed the fixer's reported changes and the independent judge results. The fixer successfully addressed the failing lowercase-code test but did not implement the expiration requirement. This demonstrates that passing the visible tests did not guarantee compliance with the complete specification. The fixer recognized the missing behavior but prioritized the smaller change requested by its instructions.