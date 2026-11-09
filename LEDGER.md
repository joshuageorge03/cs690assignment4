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