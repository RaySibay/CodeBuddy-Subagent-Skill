---
name: codebuddy-subagent
description: Use when Codex should delegate bounded, low-risk, token-heavy, independent, parallelizable, structured-output, or background work to the local CodeBuddy CLI (`codebuddy` or `cbc`) as a subagent. Trigger for CodeBuddy CLI, cbc, subagent delegation, token-heavy chores, broad repo search, log analysis, CI failure analysis, diff review, documentation migration, structured extraction, parallel exploration, or background worker tasks.
---

# CodeBuddy Subagent

## Inputs

- Receive the user goal, current project path, relevant files or directories, expected output, risk level, and success criteria.
- Identify the task type: read-only investigation, code review, log or test analysis, structured extraction, documentation migration, simple isolated edit, parallel exploration, or background work.
- State assumptions before delegation: target scope, allowed files, allowed commands, expected artifacts, verification command, and stop condition.
- Use CodeBuddy only when the delegated work is bounded enough to check and independent enough to run without continuous user clarification.

## Delegation Fit

Delegate these tasks to CodeBuddy:

- Summarize unfamiliar modules, entrypoints, call chains, data flow, public APIs, configuration layout, or package boundaries.
- Search broad codebases for references, conventions, duplicate patterns, dependency usage, dead code candidates, or ownership clues.
- Review diffs, PR patches, generated code, configuration files, migrations, tests, or docs for concrete defects and missing checks.
- Analyze build logs, CI logs, crash stacks, flaky test output, benchmark output, or large terminal transcripts.
- Extract structured data from issues, changelogs, DDL, schemas, OpenAPI files, markdown docs, release notes, dependency manifests, or code comments.
- Migrate or normalize documentation, README sections, API notes, examples, headings, terminology, links, and code block languages.
- Run independent second-pass validation on assumptions, edge cases, security risks, test gaps, or suspected regressions.
- Execute simple isolated edits with clear boundaries, such as adding a small test, updating boilerplate, renaming a local symbol, or fixing a narrow lint error.
- Launch parallel investigations across independent modules, services, packages, platforms, or keywords, then merge the findings manually.
- Run long bounded background work, such as full regression tests, dependency inventory, repo indexing, coverage comparison, or migration dry runs.
- Use CodeBuddy `WebFetch` or `WebSearch` for delegated web lookups only when the user requested current external information or the task clearly needs it.

Do not delegate these tasks to CodeBuddy:

- Do not delegate unclear requirements, product judgment, architecture ownership decisions, or tasks that need active user tradeoff discussion.
- Do not delegate destructive operations, credential handling, production changes, deployment, irreversible data changes, or `git push`.
- Do not delegate broad cross-cutting refactors unless the scope, allowed files, rollback plan, and verification commands are explicit.
- Do not run bare `codebuddy`; it starts the interactive REPL and can block the main agent.
- Do not trust CodeBuddy output as final. Treat it as a draft until Codex verifies the evidence, diffs, commands, and tests.

## Execution Steps

1. Define the success criteria before calling CodeBuddy. Write the smallest verifiable goal and name the expected output shape.
2. Choose the smallest working directory that contains the target project. Add extra directories only with `--add-dir` when required.
3. Prefer non-interactive Print mode for every subagent call: `codebuddy -p "<prompt>"`.
4. Bound the run with `--max-turns`. Use `1` for read-only summaries, `2-3` for investigation, and a higher value only for isolated implementation tasks.
5. Prefer `--output-format json` for machine-readable results. Add `--json-schema` when Codex must parse exact fields.
6. Use `--permission-mode plan` for read-only investigation, review, extraction, and scenario analysis.
7. Use `-y` only for trusted, bounded tasks where CodeBuddy may read, edit, or run commands without interactive approval.
8. Restrict the tool surface. Use `--allowedTools` for minimal permissions or `--disallowedTools` to block risky operations such as `Edit`, `Write`, `Bash(git push:*)`, deletion commands, deployment commands, or credential access.
9. Pass large logs, diffs, and generated output through stdin instead of pasting everything into the prompt.
10. Use `--bg --name <name>` only for long tasks with clear logs, stop conditions, and follow-up commands.
11. Use `/goal` only when the completion condition can be proven by CodeBuddy output, such as a test command exiting with code `0`.
12. Inspect CodeBuddy's result, session ID, tool use, modified files, and command outputs before acting on the answer.
13. Verify any delegated edit with targeted tests, lint, type checks, diff review, or direct file inspection.
14. Keep changes surgical. Do not let CodeBuddy format unrelated files, expand scope, or solve adjacent problems unless explicitly requested.
15. Report what CodeBuddy did, what Codex verified, what remains uncertain, and which files or commands support the conclusion.

## Command Patterns

Use a read-only investigation:

```powershell
codebuddy -p "Summarize src/auth: entrypoints, data flow, key risks, and files to inspect. Do not edit files." --output-format json --max-turns 1 --permission-mode plan
```

Use a structured extraction:

```powershell
codebuddy -p "Extract functions, side effects, and test gaps from src/auth/session.ts." --output-format json --max-turns 1 --permission-mode plan --json-schema '{"type":"object","properties":{"functions":{"type":"array","items":{"type":"string"}},"side_effects":{"type":"array","items":{"type":"string"}},"test_gaps":{"type":"array","items":{"type":"string"}}},"required":["functions","side_effects","test_gaps"]}'
```

Analyze a large log from stdin:

```powershell
Get-Content -Raw .\build-error.txt | codebuddy -p "Analyze this build failure. Return root cause, suspect files, and minimal next command." --output-format json --max-turns 1 --permission-mode plan
```

Review a diff from stdin:

```powershell
git diff main | codebuddy -p "Review this diff for correctness bugs, missing tests, security risks, and risky behavior changes. Do not suggest style-only issues." --output-format json --max-turns 1 --permission-mode plan
```

Run a bounded isolated edit:

```powershell
codebuddy -p "Add focused tests for the parser edge case in tests/parser. Modify only parser tests. Run the targeted test command and report the diff." --output-format json --max-turns 3 -y --allowedTools "Read,Grep,Glob,Edit,Bash" --disallowedTools "Bash(git push:*),Bash(rm:*),Bash(del:*),Bash(Remove-Item:*)"
```

Start background work:

```powershell
codebuddy --bg --name full-regression "Run the full regression suite, collect failures, and produce a concise failure summary. Do not edit files." -y --allowedTools "Bash,Read,Grep,Glob"
codebuddy ps
codebuddy logs full-regression
codebuddy attach full-regression
```

Use a goal-driven run:

```powershell
codebuddy -p "/goal npm test exits 0 for the auth package and the final response includes the exact command output summary" --output-format json --max-turns 5 -y --allowedTools "Read,Grep,Glob,Edit,Bash"
```

## Output Handling

- Extract the final result, `session_id`, tool-use summary, changed files, and verification evidence from JSON output.
- Resume an explicit session with `codebuddy -r "<session-id>" -p "<follow-up>"` only when continuing the same bounded task.
- Prefer fresh sessions for independent reviews, parallel exploration, and second opinions.
- Merge parallel findings manually. Deduplicate file references, rank risks by severity, and discard unsupported claims.
- Re-run or locally verify commands that matter to the user outcome.
- Stop delegation when CodeBuddy asks for clarification, expands scope, loops, touches unrelated files, or cannot produce evidence.

## Outputs

- Provide a concise summary of the delegated task, CodeBuddy command pattern used, and session ID when available.
- List concrete findings with file paths, commands, logs, or diffs as evidence.
- Mark unverified CodeBuddy claims as unverified until Codex checks them.
- Include verification results: tests run, lint/type checks run, files inspected, or reason verification was skipped.
- Deliver the smallest final change or recommendation that satisfies the original success criteria.
