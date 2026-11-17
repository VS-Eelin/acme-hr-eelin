# Prompts — copy and paste

Every prompt used on the slides and in the labs, in lab order. Step numbers (1.2, 4.5 …) match the lab guides in `labs/`.

Each lab prompt says the exact values to use, what to check, what never to do, the result to expect and where to stop and ask. If Claude's result does not match the expected result, stop and ask a trainer instead of asking Claude to try harder.

**Machine notes**
- Windows: stop the API (Ctrl+C) before `dotnet test` or a build. A running API locks `src/Api/bin` and the build fails with MSB3027 (the test hook then says "Tests failing" with no test names).
- Run the API and the web app in two terminals of your own, not through Claude: servers Claude starts in the background stop after about 10 minutes.
- `npm run build` overwrites the web dev server's files. Restart `npm --prefix web run dev` after a build, or every page goes blank.
- Never paste a key, password, database URL or deploy hook into Claude. Put them in `.env` with your editor, or use `gh secret set` in your own terminal.

## Setup

### Explain the repo (S.5)
```text
Explain this repo and how to run it. Point me to the files for the API, the web app, the tests and the Claude Code settings. Do not change anything, and do not open .env.
```

## Module 2 — Claude Code

### Plan first (any change)
```text
Read <file or spec> and plan the change. List the files to change and the tests to add first. Do not edit yet.
```

### Code review
```text
Review my changes against main for bugs, missing tests and security issues. List findings by severity with file and line. Don't fix yet.
```

### Debugging
```text
Here is the error and the steps to reproduce: <paste>. Write a failing test that shows it, find the root cause, then fix it. Show me the test going red, then green.
```

### Refactoring
```text
Refactor <class> for readability without changing behaviour. Keep all tests passing after each step. Stop and ask before changing any public API.
```

### Done check
```text
Check the Definition of done in CLAUDE.md and confirm each item with evidence: which commands you ran and their results.
```

## Module 3 — API warm-up

### Add a second tool
```text
Read tools/Warmup/Program.cs. Add a second tool, get_public_holidays(state), that returns a small fake list of 2026 Malaysian public holidays for SEL, KUL and PNG, and an error for any other state. Follow the TODO and keep the loop unchanged. Build it, then run: dotnet run --project tools/Warmup -- "Can I take leave on 1 Feb 2026 in KL, and how many days do I have left?" Show me the tool_use lines from the output.
```

## Lab 1 — Leave API

### Plan prompt (1.2, plan mode)
```text
Read docs/feature-spec-leave.md, src/Api/Services/LeaveService.cs, src/Api/Auth/AccessService.cs and the failing test. Propose a plan: files to change, tests to add first, and risks. Include fixing the failing weekend test. Ask me if anything in the spec is unclear. Don't write code yet.
```

### Build it (1.3)
```text
Go ahead with the plan. Write the tests first, then the code, including API tests in src/Api.Tests that another employee gets 403 on approve. Keep controllers thin. When the API is done, add the create form and Approve / Reject buttons to web/app/leave/page.tsx using web/lib/api.ts; show each request's days from the API and the available balance. Then run npm --prefix web run build to check the page compiles.
If a build fails with MSB3027, stop and tell me to stop the API; do not work around it.
Never: open .env, change the planted tests to make them pass, push.
Expected: every test passes, including the weekend test that failed at the start, and the web build succeeds. If a test fails for a reason outside this lab, stop and tell me.
```

### Try to break the rules (1.4)
```text
Test the secret protection without touching my real .env. src/.env does not exist, so nothing can leak.
1. From the repo root: use the Read tool on src/.env. Then run the shell command: echo hook-test .env
2. Run cd web. From there: use the Read tool on ../src/.env, then run: echo hook-test .env. Then cd back to the repo root.
For each of the 4 attempts, tell me exactly what stopped it: "denied by your permission settings" (a deny rule), "Blocked by hook", or nothing. Show the results as a table: folder, attempt, what stopped it.
Never: open, read or print the real .env, or retry a blocked attempt with another command.
```

## Lab 2 — HR helpdesk agent

### Finish the agent loop (2.2)
```text
Read src/Api/Helpdesk/HelpdeskAgent.cs and HelpdeskTools.cs. Complete the TODOs: the agent loop must resend the full history every turn (send a copy: history.ToList()), check StopReason, return one tool_result for every tool_use (even when the tool throws), and stop at MaxTurns. Then implement the 3-working-day rule inside the apply_leave tool in C#, not in the prompt. Write unit tests with FakeClaude in src/Api.Tests (FakeClaude.Text, FakeClaude.ToolUses and FakeClaude.ToolResults).
Never: put the rule only in the prompt, change tickets.json, push.
Expected: all tests pass, including the 5 helpdesk tests that failed at the start.
```

### Run the tickets (2.3)
```text
The API is running with my key in .env. Run dotnet run --project tools/TicketRunner. Show a table: ticket, expected, outcome, turns used. For any ticket that failed, show its reply and tool outputs and explain why.
Expected: 10 tickets get an outcome; ticket 3 returns needs_manager_approval from apply_leave; ticket 6 cites POL-05 and ticket 10 cites POL-11; ticket 7 (prompt injection) is refused and creates no leave; no run goes past MaxTurns.
If every ticket fails with 401 or "invalid key", stop and tell me: the key is missing from .env or the API was not restarted.
```

## Lab 3 — Policy research agents

### Finish the worker (3.2)
```text
Read src/Api/Research/PolicyResearch.cs. Implement RunWorker(subtopic): a separate Claude conversation with only the search_policies and read_policy tools, limited by MaxTurnsPerWorker, that returns findings with policy ids. If it stops at the turn limit or the token budget, return text that starts with "Stopped:" so the lead marks that tool_result as an error. Keep the token counting per agent. Add tests with FakeClaude for: a normal answer, the turn limit, and the token budget.
Never: give workers more tools, raise the limits to make a test pass, push.
Expected: all tests pass, including the 2 research tests that failed at the start.
```

### Ask a research question (3.4)
```text
The API is running. Signed in as farah.tan6@acme-hr.example (password in README.md), call the policy research endpoint with: "Can I carry forward leave and take it as half days?" Show the answer, the policy id cited for each claim, citationsValid, stoppedBy, the number of workers and turns against MaxWorkers and MaxTurnsPerWorker, and tokens per agent. Do not print the token.
Expected: citationsValid true, stoppedBy end_turn, inside the limits, every claim cites POL-xx. If the lead's first answer had an uncited sentence, the code gives it one repair turn (the lead shows one extra turn): that is expected.
```

## Lab 4 — Team productivity

### Create the database (4.2)
```text
Use the render MCP server. List my Render workspaces and ask me to confirm one. Then call create_postgres with name acme-hr-db, plan free, region singapore, version 16, in that workspace. Do not request or show any connection string, password or key. Check the status with get_postgres until it is available (this takes a few minutes), then tell me the id, plan, region, version and expiry date, and tell me what I must add to the IP allow list to connect from my laptop.
Stop and ask me if: there is more than one workspace, a free database already exists, or the plan is not free.
```

### Check the database (4.5)
```text
I have put DATABASE_URL in .env myself. Do not open .env. Restart the API and call http://localhost:5080/health. Tell me the provider and the database line.
Then check that signing in as farah.tan6@acme-hr.example with the public password Acme@2026 returns 401.
Expected: "provider":"Npgsql.EntityFrameworkCore.PostgreSQL", database connected, and 401 for Acme@2026. If it says Sqlite, stop: .env was not saved or the API did not restart. If it says NOT connected, stop: check the IP allow list. If the API does not start and says SEED_USER_PASSWORD must be set, set your private password in .env.
```

### MCP question (4.5)
```text
Using the render MCP server: select my workspace and find my acme-hr-db Postgres database. With get_postgres, show its status, plan, region, version and expiry date. With get_metrics, show its recent CPU or active connections, whichever it offers. Do not run SQL and do not change anything.
```
Why no SQL: Render's MCP server queries from Render's own machines, which have no fixed IP, so your database's IP allow list (only your laptop) blocks it. That is the allow list working.

### Check the guardrail (4.6)
```text
Show me the Render deny and ask rules in .claude/settings.json and list the Render tools you have in this session. Is update_environment_variables among them? If it is, call it once to set TEST=1 on the service with id srv-does-not-exist. Tell me exactly what stopped you: the tool is not available (removed by a deny rule), a permission rule, or Render itself.
```

### Feature A (4.8, worktree 1)
```text
Read docs/feature-spec-calendar.md and build it. Plan first, tests where it applies. Use port 3000 for the web app only after the main folder's dev server is stopped.
```

### Feature B (4.8, worktree 2)
```text
Read docs/feature-spec-holidays.md and build it with /new-endpoint. Plan first, tests first.
```

### Commit the feature (4.8, in each worktree)
```text
Run dotnet test, then commit the changes for this feature with a short message that says what the feature does for HR users. Before you commit, show me git status. Do not commit CLAUDE.md changes, .claude/settings.json changes, or a package-lock.json that changed only because of my npm version. Do not push.
```

### Release notes (4.9)
```text
Write release notes for this release.
```

## Lab 5 — CI

### Open the pull request (5.4)
```text
Open a pull request from lab5-planted-bug to main with gh pr create, title "Lab 5: planted bug". Show me the files changed. Wait for the checks with gh pr checks --watch and tell me the result of ci and claude-review. Do not merge.
Expected: only CoreControllers.cs and LeaveService.cs changed; ci passes; claude-review fails and posts one comment in the Conversation tab.
If claude-review fails and there is no comment on the pull request, stop: the ANTHROPIC_API_KEY secret is wrong or on another repo. A correct review also fails in about 30 seconds: look for the comment, not the time.
```

### Fix the planted bug (5.6)
```text
The claude-review check failed on my pull request. Here is the review comment:

<paste the review comment from the pull request>

Fix every blocking finding in the comment, not only the first. Start with failing tests: one each for Approved, Pending and Rejected leave, one that another employee gets 403, and one that an unknown employee id gets 404. Run them and show me they fail, then fix the code until they pass. Run all tests and tell me which ran. Do not push.
```

### Check the deploy (5.7)
```text
Run gh run list --branch main --limit 5, then show the log of the latest deploy run with gh run view --log. Tell me if the deploy step printed a deploy id that starts with dep-. Do not print any secret.
Expected: {"deploy":{"id":"dep-..."}}. Look only for dep-: the log also shows the script's own text "RENDER_DEPLOY_HOOK not set", which means nothing. No dep- id means the deploy was skipped even though the run is green: stop and check the secret with gh secret list.
```

## Lab 6 — Document intake

### Finish the extractor (6.2)
```text
Read src/Api/Documents/DocumentExtractor.cs, DocumentValidator.cs and docs/feature-spec-documents.md. Implement the TODOs: JSON schema output with OutputConfig, return NeedsReview for a stop reason other than end_turn, parse and validate in C#, one retry with the exact validation problems, otherwise NeedsReview. The retry message must say: correct only reading mistakes; if the document itself is wrong, keep its values as written. Add tests with FakeClaude, including one for stop reason max_tokens (FakeClaude.Text(json, "max_tokens")).
Never: change DocumentValidator rules, the samples in seed/documents or expected.json to make a test or score pass; push.
Expected: all tests pass, including the 5 extractor tests that failed at the start.
```

### Improve the score (6.4)
```text
The API is running with my key in .env. Run dotnet run --project tools/ExtractScore. Look at the wrong-field lines and find the field that is wrong most often. Suggest ONE change to the prompt or the schema and wait for my OK. Then stop the API, make the change, run dotnet test, start the API again and run the score again. Show both scores.
Never: edit expected.json or the samples.
Expected: 20/20 end Valid or NeedsReview; samples 08, 10 and 17 stay NeedsReview with their values as written.
```

## Day 2 — Build-a-Skill challenge
```text
Create a Skill in .claude/skills/<name>/SKILL.md for this job: <your team's repeat job>. Write a description that says what it does and when to use it. Keep the steps short.
```
