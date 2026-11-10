# Assignment 4 Report

Name:Joshua George
Commit graded:d3578de1604c07011f4e8d5492c1ebd509f80aec

Answer from your own traces. Give step numbers as `attempt 1, step 4`. Keep to the word
limits; a short answer with the right evidence earns full points.

## 1. The runs

Run `python -m agent summary` and paste its table here exactly as printed.

| Run | Tools | Judge | Stopped (per attempt) | Model calls | Tool calls | Input tokens | Output tokens | Cost | Judge verdict |
| --- | --- | --- | --- | ---: | ---: | ---: | ---: | ---: | --- |
| runA | local | after | 1: done | 8 | 7 | 10810 | 741 | $0.0055 | 1: PASS |
| runB | mcp | reflexion | 1: done | 10 | 9 | 17116 | 1078 | $0.0086 | 1: PASS |

## 2. A rejection is feedback (Run A)

At step 4, my agent proposed an edit to make the discount codes case-insensitive. I rejected the edit because it had only looked at the tests and source code without reading `spec.md`. I told it to review the specification first so it could make sure the change followed all the requirements. At step 5, it read `spec.md` and noticed that the expiration date also needed to be checked. At step 6, it made another proposal that addressed both problems. I approved that edit. So the agent did follow my feedback instead of just trying the same change again.

## 3. The visible tests and the judge (Run A)

The last `run_tests` call was at step 7, where the agent reported that all 4 visible tests passed. The agent then stopped at step 8 because it had finished the task and responded without another tool call. Afterward, the independent judge reported that all 6 acceptance tests passed as well. This showed that the agent didn't just fix the problem with lowercase discount codes, but also handled the expiration requirement that wasn't covered by the visible tests. Since the judge passed, there weren't any remaining failures to explain.

## 4. The lessons (Run B)

Run B didn't write any lessons to `NOTES.md` because the agent passed the judge on its first attempt. At step 9, it ran the visible tests and all 4 passed. It then finished at step 10, and the judge reported that all 6 acceptance tests passed. Because of this, there was no reason for Reflexion to generate a lesson or start another attempt.

I did reject the first proposed edit because it handled expired codes incorrectly. The agent then read `spec.md` and eventually corrected the issue. So even though Reflexion wasn't needed, the agent still used my feedback to improve its solution.

## 5. Memory grows (Run A)

At step 1, the first tool event recorded 482 input tokens. By step 7, the last tool event recorded 2,109 input tokens.

The number increased because the agent kept adding previous messages and tool results to its conversation history. Each time it called the model, it had more information from earlier steps to include, which increased the number of input tokens.

## 6. MCP

One of the tool descriptions returned by `check-mcp` was:

"List every file in the project, one path per line, relative to the project folder.

Takes no inputs. Use it first to see what the project contains; the paths it returns
are the paths read_file and edit_file expect."

One risk of installing an MCP server from someone else is that it could access files or perform actions that I didn't intend. My agent uses `gate_for()` to control which tools can run and which ones require approval. However, these restrictions only apply to tools being called through my agent. If a malicious MCP server performs actions outside of that, my agent wouldn't necessarily be able to stop it.

## 7. The reviewer and the judge

The reviewer checked Run B's final `discounts.py` against all five rules in `spec.md` and didn't find any problems. This matched the judge's result, where all 6 acceptance tests passed. The reviewer did mention some possible edge cases involving spaces and negative totals, but those weren't requirements in the specification.

The reviewer was limited to Read, Grep, and Glob through its agent configuration. This restricted which tools it could use, but it wasn't a filesystem-level lock. It also explicitly stated that it didn't modify any files or run the code.

## 8. Your agent and a vendor's agent

The vendor fixer added `code = code.upper()` to handle lowercase discount codes. All 4 visible tests passed, but the judge only passed 5 of 6 because the fixer didn't handle expired codes. It didn't change anything in `tests/`.

My Run A agent also fixed the lowercase code issue, but it checked the specification after my rejection and fixed the expiration problem too. As a result, it passed all 6 judge tests.

With my own agent, I could directly control approval gates, budgets, and stopping rules in Python. The vendor's fixer configuration couldn't define those same loop controls. However, Claude Code was easier to use because its coding tools and execution environment were already built in.