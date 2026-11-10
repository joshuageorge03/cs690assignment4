# Provenance Ledger

Write one entry per reviewable change or experiment, when the work is done. Use exactly
the schema in HANDOUT.md. Put the prompts you typed into AI tools in `prompts/` and name
the file in the entry's `prompts` field.

For a run, most of the entry comes straight from its trace: what the agent did and in
what order, the commands that checked it, and the result. You add what only you know:
how you reviewed it, what you rejected, and what is still uncertain.

## Worked example (not graded; leave it here and add your entries under "My entries")

This shows the level of detail expected. The commit SHAs, dates, steps and numbers are
made up, and so is the practice run.

```
## Entry 2
artifact:  agent/loop.py, run_agent, at commit 9c3a71e
tool:      Claude Code, model Claude Sonnet 5.5, 2026-11-01
prompts:   asked for the loop body from the comment above run_agent;
           prompts/loop-01.md
review:    read every line against the comment. It returned as soon as is_stuck was
           true, which skipped the tool results for the other calls in the same turn;
           moved the stuck check after the for loop as the comment says. Rejected its
           suggestion to import a framework's message classes; kept the dicts from
           core.py
checks:    pytest tests/test_loop.py: 9 passed
evidence:  HANDOUT Part 3; the comment above run_agent, steps 4 and 5
risk:      no test of mine has three tool calls in one turn

## Entry 4
artifact:  runs/try1, a practice run, agent code at commit 4b9e2d1
tool:      my agent (python -m agent run try1), anthropic claude-haiku-5-5, 2026-11-02
prompts:   the default task and BASE_SYSTEM in agent/loop.py; prompts/try1-task.md
review:    read every event in trace.jsonl. At attempt 1, steps 4 and 6, the model
           copied old_text with the read_file line numbers, and edit_file refused both
           times. At step 7 it made an edit that changed nothing the tests check, and
           at step 8 the stuck rule stopped it. I approved the step 7 edit without
           reading it closely
checks:    python -m agent check-trace try1: complete
evidence:  HANDOUT Part 1 (tool descriptions) and Part 2 (is_stuck)
risk:      one practice run; a better description may still not stop the copying
dataset:   fixture/ at commit 4b9e2d1, task "Make the tests pass."
result:    stopped stuck after 8 model calls, 14,880 input tokens, about $0.002;
           runs/try1/trace.jsonl
changed:   wrote "copy it without the line numbers" into the read_file and edit_file
           descriptions, and read every diff before approving
```

## My entries

## Entry 1
artifact:  agent/tools.py, read_file
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement read_file using safe_path, handle invalid
           line ranges and missing files, and format the output with line
           numbers; prompts/read_file.txt
review:    reviewed how the function checks the file path and handles
           line ranges. Checked that it limits the number of lines
           returned and tells the agent when more lines are available
checks:    pytest tests/test_tools.py -v: 16 passed
evidence:  HANDOUT Part 1; comments above read_file in agent/tools.py
risk:      edge cases with unusual file contents may not be covered

## Entry 2
artifact:  agent/tools.py, edit_file
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement edit_file so it only replaces text when
           there is exactly one match, while handling missing files and
           invalid replacements; prompts/edit_file.txt
review:    reviewed how the function counts matches before replacing
           text and checks that the file is within the workspace.
           Checked that it rejects zero or multiple matches
checks:    pytest tests/test_tools.py -v: 16 passed
evidence:  HANDOUT Part 1; comments above edit_file in agent/tools.py
risk:      replacements involving unusual whitespace or encodings
           may require additional testing

## Entry 3
artifact:  agent/tools.py, run_tests
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement run_tests using the existing
           run_unittests helper, apply the timeout, and summarize
           results through format_report; prompts/run_tests.txt
review:    reviewed how the function calls the test runner and handles
           timeouts. Checked that failing tests return a summary
           rather than raising an error
checks:    pytest tests/test_tools.py -v: 16 passed
evidence:  HANDOUT Part 1; comments above run_tests in agent/tools.py
risk:      timeout behavior and unusual test failures may need
           additional verification


## Entry 4
artifact:  agent/policy.py, gate_for()
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement gate_for using the existing GATES mapping,
           require approval for edits, and deny unknown tools and edits to
           the tests folder; prompts/gate_for.txt
review:    reviewed how the function uses GATES to determine permissions
           and normalizes paths before checking whether edits target
           the tests folder. Checked that unknown tools are denied
checks:    pytest tests/test_policy.py -v: 7 passed
evidence:  HANDOUT Part 2; comments above gate_for in agent/policy.py
risk:      unusual path formats or symbolic links may require additional
           validation

## Entry 5
artifact:  agent/loop.py, run_agent
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement the ReAct loop using the requirements
           provided in the starter code, including tool calls, approval
           checks, token budgets, and stopping conditions;
           prompts/run_agent.txt
review:    reviewed how the function communicates with the model and
           processes tool calls. Checked that edit_file requires approval
           and denied tools are not executed. Also checked that the loop
           tracks token usage, records tool results, and processes all
           tool calls before checking if the agent is stuck
checks:    pytest tests/test_loop.py -v: 9 passed
evidence:  HANDOUT Part 3; comments above run_agent in agent/loop.py
risk:      multiple tool calls in one turn and unexpected model responses
           may require additional testing

## Entry 6
artifact:  agent/reflexion.py, reflexion()
tool:      ChatGPT, GPT-6
prompts:   asked AI to implement reflexion using the existing run_agent,
           judge, and write_lesson functions to handle multiple attempts
           and save feedback between runs; prompts/reflexion.txt
review:    reviewed how the function runs the agent and checks the results
           after each attempt. Checked that failed attempts generate
           lessons that are saved for the next attempt and that the
           function stops when the tests pass or a provider error occurs
checks:    pytest -q: 37 passed, 3 failed
evidence:  HANDOUT Part 4; comments above reflexion in agent/reflexion.py
risk:      repeated attempts with actual model responses may behave
           differently from the provided tests


## Entry 7
artifact:  runs/runA, local tools
tool:      my agent (python -m agent run runA), OpenAI gpt-4.1-mini
prompts:   default task "Make the tests pass." and BASE_SYSTEM in
           agent/loop.py
review:    reviewed the first proposed edit and rejected it because
           the agent had only read the tests and source code, not
           spec.md. I asked it to review the specification before
           making changes. It then read spec.md and proposed a
           second edit that handled both case-insensitive codes
           and expired discount codes. I approved that edit
checks:    run_tests: 4 of 4 passed
           judge: 6 of 6 passed
           check-trace and replay: pending verification
evidence:  runs/runA/trace.jsonl; HANDOUT Part 6
risk:      only one recorded run, and I relied on the provided
           tests and judge to verify the final changes
dataset:   fixture/ at commit acb0f57, task "Make the tests pass."
result:    stopped done after 8 model calls and 7 tool calls;
           10,810 input tokens, 741 output tokens, cost $0.0055;
           judge PASS
changed:   rejected the first edit and requested a review of spec.md;
           approved the revised edit to discounts.py that added
           code normalization and expiration checking

## Entry 8
artifact:  runs/runB, MCP tools with Reflexion
tool:      my agent (python -m agent run runB --tools mcp --judge),
           OpenAI gpt-4.1-mini
prompts:   default task "Make the tests pass." and BASE_SYSTEM in
           agent/loop.py; prompts/runB-task.md
review:    reviewed the first proposed edit and rejected it because
           expired discount codes returned the original total instead
           of raising ValueError. I asked the agent to read spec.md
           and check all five rules. The agent then attempted another
           edit, but edit_file rejected it because the old text did
           not match. It reread discounts.py and proposed a corrected
           edit, which I approved
checks:    run_tests: 4 of 4 passed
           judge: 6 of 6 passed
           check-trace and replay: pending verification
evidence:  runs/runB/trace.jsonl; HANDOUT Part 6
risk:      Reflexion did not need a second attempt because the judge
           passed on the first attempt. This run does not demonstrate
           whether saved lessons improve later attempts
dataset:   fixture/ at commit acb0f57, task "Make the tests pass."
result:    stopped done after 10 model calls and 9 tool calls;
           17,116 input tokens, 1,078 output tokens, cost $0.0086;
           judge PASS on attempt 1
changed:   rejected the initial incorrect expiration handling;
           approved the revised discounts.py edit after the agent
           reviewed spec.md and recovered from a failed text match


## Entry 9
artifact:  .claude/agents/reviewer.md, read-only reviewer subagent
tool:      ChatGPT, GPT-6
prompts:   asked AI to create a reviewer subagent that checks Python
           code against spec.md without making modifications;
           prompts/reviewer_agent.txt
review:    reviewed the reviewer configuration and checked that it
           only has access to Read, Grep, and Glob. Verified that
           its instructions require checking each rule in spec.md
           and explaining any violations without modifying files
checks:    claude --agent reviewer: completed read-only review of
           runs/runB/workspace/discounts.py against spec.md;
           all five specification rules reported as followed
evidence:  HANDOUT Part 7; .claude/agents/reviewer.md;
           evidence/review.md
risk:      the reviewer only inspected the code and did not execute
           tests; its conclusions depend on static analysis
evidence:  HANDOUT Part 7; .claude/agents/reviewer.md
risk:      the reviewer may overlook requirements or incorrectly
           interpret parts of the specification


## Entry 10
artifact:  .claude/agents/fixer.md, runs/vendor, vendor fixer experiment
tool:      Claude Code, fixer subagent
prompts:   asked the fixer to make the tests pass in the vendor
           workspace; instructions in .claude/agents/fixer.md;
           prompts/fixer_agent.txt
review:    reviewed the fixer's implementation and final response.
           It added code.upper() to handle lowercase discount codes.
           The fixer identified the missing expiration requirement
           but left it unchanged because all visible tests passed
checks:    python -m agent judge vendor:
           visible tests 4 of 4 passed;
           acceptance tests 5 of 6 passed
evidence:  HANDOUT Part 7; evidence/fixer.md;
           runs/vendor/judge.txt
risk:      the fix passes the visible tests but fails the expiration
           requirement checked by the independent judge
dataset:   runs/vendor/workspace, clean copy of fixture/
result:    visible tests passed; independent judge failed because
           expired discount codes did not raise ValueError
changed:   added code = code.upper() to discounts.py;
           did not modify the expiration handling