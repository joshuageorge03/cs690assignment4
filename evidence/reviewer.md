# Part 7 — Reviewer Subagent Evidence

Agent: Claude Code reviewer
Model: Claude Opus 5.5
Date: 2026-11-09

Command:
claude --agent reviewer "Review runs/runB/workspace/discounts.py against runs/runB/workspace/spec.md. For each specification rule, explain whether the implementation satisfies it. Do not modify any files."

## Reviewer Output
The code follows all five rules in spec.md. I found no problems. I didn't run the code or change any files; this is from reading it.

Rule 1: an empty code leaves the total unchanged. Followed.
Lines 14–15 check code == "" and return total_cents unchanged.

Rule 2: codes are not case-sensitive. Followed.
Line 16 turns the code into upper case (code = code.upper()) before looking it up. So "save10" and "Save10" both become "SAVE10" and match the entry on line 7. All keys in CODES are upper case, so this works for every code.

Rule 3: an unknown code raises ValueError. Followed.
Lines 17–18 raise ValueError when the code isn't in CODES. This happens after the case change, so only codes that really don't exist are rejected.

Rule 4: a code works up to and including its last day. Followed.
Line 20 uses today > last_day. On the last day, today == last_day, so the code still applies. From the next day on, line 21 raises ValueError. The check uses > rather than >=, which is what makes the last day count.

Rule 5: the discount is whole cents, rounded down. Followed.
Line 22 calculates total_cents * percent // 100. It multiplies before dividing and uses whole-number division, so there's no decimal rounding error and the result is always rounded down. The spec's example works: 1999 * 10 // 100 = 199, and 1999 - 199 = 1800.

Notes outside the spec (not violations):
- A code with spaces around it, like " SAVE10 ", is rejected as unknown because spaces aren't trimmed. The spec doesn't ask for trimming.
- The code doesn't check total_cents. A negative total would get a discount rounded toward the more negative number. The spec doesn't say what should happen with negative totals.
- The empty-code check on line 14 runs before the expiry check. So "" returns the total unchanged on any date, which matches rule 1.
