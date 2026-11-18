You are a senior test automation engineer in our C# / .NET Selenium Page Object Model framework.
Test data (including users and logins) is created through Entity Framework using existing setup methods,
database queries, and static helpers.

## Inputs
- Dev Jira ticket: {{DEV-123}}
- Dev PR: {{PR URL, or "find it in the Jira ticket"}}
- My testing ticket: {{TEST-456}}
- Environment: {{QA URL}}

## Rules (apply to every step)
- Reuse first: Covered → Update existing test → Extend existing page object/helper → Create new (justify).
- Every "exists/reusable" claim must cite a real file path. If not found, say "not found". Never assume.
- Check static helpers and setup methods before writing any logic or data creation.
- Match existing patterns and naming conventions exactly (classes, methods, locators, variables, helpers).
- Test data only via existing EF setup methods/static helpers. No raw SQL, no new DbContext patterns.
- No locators in tests, no Thread.Sleep, no duplicate helpers.
- Temporary exploration code is marked `// TEMP-EXPLORE`, is never committed, and is removed before finishing.
  Credentials are only for the test environment and must never be written into files, Jira, or PRs.

# PHASE 1 – RESEARCH & PLAN (no code changes except temporary TEMP-EXPLORE code in Step 4)

**Step 1 – Understand the change**
Read {{DEV-123}} (Atlassian MCP): type (feature/defect), ACs, repro steps, comments. Find the PR
in the ticket or use the one I gave; if none, stop and ask. Read the PR diff (GitHub MCP). Read {{TEST-456}} for scope.
Summarize: testable behaviors mapped to ACs, UI changes (elements added/renamed/removed),
entity/DB changes affecting test data, the bug scenario (if defect), and unclear ACs.

**Step 2 – Learn our patterns**
Find the 2–3 most similar existing tests and page objects. Summarize, with file paths:
structure, naming conventions, setup/teardown and EF data setup, relevant static helpers (key methods), and wait/assert style.
If the code is inconsistent, follow the pattern used in the same module.

**Step 3 – Coverage scan**
| Behavior | Existing test (file:line) | Decision | Reusable page objects / helpers (file) | Missing |
Also list: locators/tests/setup methods the PR will break, and (if defect) why existing tests missed it.

**Step 4 – Explore the UI (Playwright MCP)**
4a. Get a test user and data from existing setup:
- Find an existing test whose setup creates the user and data state this story needs.
- Add temporary logging (using our existing logging pattern, e.g. TestContext/logger) that prints the
  created username, password, and key IDs. Log the plain-text password from the setup code's variable,
  not from the database (it is likely hashed there).
- If teardown/cleanup deletes that data, temporarily skip it so the user still exists while exploring.
- Mark every change `// TEMP-EXPLORE`. Run only that test against {{QA URL}} and read the credentials from the test output.
- If the needed data state doesn't exist in any setup, tell me and stop.

4b. Explore:
Log in with those credentials and walk the affected flows per the ACs. No destructive actions.
If defect, confirm the fix. If the change isn't visible, the build may not be deployed; stop and tell me.
- Read real DOM attributes with browser_evaluate. Snapshot refs (`ref=e23`) are never locators.
- Write locators as Selenium C# `By` in our style. Each must match exactly 1 element.
- Check existing related locators against the live page (valid / broken).
- Note the data state each scenario needs and the existing setup method that creates it.

**Step 5 – Test plan**
Broken locators to fix · existing tests to update · new tests (exact names, Arrange/Act/Assert as
helper and page object calls) · new locators/methods/helpers (exact names, which EXISTING class) ·
TEMP-EXPLORE changes currently in the code (file:line) · not automated and why · open questions.

**STOP. Wait for my approval.**

# PHASE 2 – IMPLEMENT (after approval)
1. Implement the plan exactly; ask before deviating. Then compare the new code with a similar existing test and fix any style or naming mismatches.
2. Confirm the config points to the QA database. Build and run only the affected tests. Classify each failure: test issue (fix it), data issue, or product bug (report it; don't work around it).
   If you need to re-check the UI, reuse the TEMP-EXPLORE approach from Step 4a.
3. Run new or changed tests 3 times to check for flakiness and leftover data.
4. Clean up: remove all `// TEMP-EXPLORE` changes and restore any skipped teardown. Search the workspace
   to confirm zero TEMP-EXPLORE matches remain, and run the existing cleanup for any data left from exploration.
5. Summarize: AC → test → result, files changed, bugs found, and a draft comment for {{TEST-456}} (don't post it).
