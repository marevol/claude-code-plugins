---
name: create-my-test
description: Generate tests for staged or specified source files
user-invocable: true
allowed-tools:
  - Bash
  - Read
  - Grep
  - Glob
  - Write
  - Edit
  - Agent(test-engineer)
---

# Generate Tests

Generate comprehensive tests for source files, following the project's existing test conventions.

## Delegation Policy (MANDATORY)

This skill MUST delegate test analysis and generation to the `test-engineer` subagent. The main session's only job is to:

- Identify target source files (step 1)
- Invoke the subagent via the Agent tool (step 2)
- Confirm the generated test plan with the user and surface results

The main session MUST NOT read source files to design tests, draft test cases, or write test code inline. If you start drafting `describe(...)` / `it(...)` / `@Test` blocks in the main session, STOP and dispatch the subagent.

## Instructions

1. **Identify target files** (main session)
   - If the user specified files, use those
   - Otherwise, run `git diff --cached --name-only` to get staged files
   - Filter to source files only (exclude test files, config files, documentation)
   - If no target files found, inform the user and stop

2. **REQUIRED — invoke `test-engineer` subagent**

   You MUST call the Agent tool with `subagent_type: test-engineer`. Do NOT analyze or write tests yourself.

   Build the subagent prompt using this template:

   ```
   You are dispatched by the create-my-test skill to generate tests.

   Repository root: {pwd}
   Target source files: {target_file_list}

   Steps you must perform:

   1) Detect the test framework and conventions:
      - Examine config (jest.config.*, pytest.ini, pom.xml, build.gradle, etc.)
      - Identify test directory layout (__tests__/, src/**/*.test.*, test/, tests/)
      - Identify naming conventions (*.test.ts, *_test.go, Test*.java, test_*.py)
      - Read 2-3 existing test files to capture assertion style, setup/teardown, mocking approach

   2) Analyze each target source file:
      - Public APIs, exported functions, class methods
      - Dependencies / collaborators that may need mocking
      - Edge cases: null/empty inputs, boundary values, error conditions, async behavior

   3) Generate test files covering:
      - Happy path for each public function/method
      - Error cases (invalid inputs, exceptions)
      - Boundary values (empty collections, zero, max, off-by-one)
      Follow the project's existing patterns for imports, assertions, and organization.
      Use descriptive test names that explain the expected behavior.

   4) Place files following the convention detected in step 1.

   Before writing any files to disk, return a plan to the main session:
   - For each target source file: planned test file path + bullet list of cases.

   The main session will obtain user confirmation and re-invoke you with approval to write.
   ```

3. **Confirm plan with the user, then re-invoke** (main session)

   Show the test plan returned by the subagent to the user. After confirmation, re-invoke `test-engineer` with the same context plus instruction "User has approved — write the test files now and return the list of files written."

   The main session MUST NOT write the test files itself.
