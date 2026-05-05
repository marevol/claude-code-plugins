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

Generate comprehensive tests for source files, following the project's existing test conventions. Convention analysis and test generation are delegated to the `test-engineer` subagent so framework-specific expertise (jest, pytest, JUnit, go test, etc.) is applied.

## Instructions

1. **Identify target files**
   - If the user specified files, use those
   - Otherwise, run `git diff --cached --name-only` to get staged files
   - Filter to source files only (exclude test files, config files, documentation)
   - If no target files found, inform the user and stop

2. **Delegate analysis and generation to `test-engineer`**
   Dispatch the `test-engineer` subagent with the resolved target file list from step 1. Instruct it to perform steps 3-6 below (detect framework, analyze targets, generate test files, place them) and return a summary of generated tests with their coverage areas.

   The remaining steps describe what the subagent must cover.

3. **Analyze project test patterns**
   - Detect the test framework by examining existing test files and configuration (e.g., `jest.config.*`, `pytest.ini`, `pom.xml` test dependencies, `build.gradle` test config)
   - Identify the test directory structure (e.g., `__tests__/`, `src/**/*.test.*`, `test/`, `tests/`)
   - Identify naming conventions (e.g., `*.test.ts`, `*_test.go`, `Test*.java`, `test_*.py`)
   - Review 2-3 existing test files to understand assertion style, setup/teardown patterns, and mocking approach

4. **Analyze target source files**
   - Identify public APIs, exported functions, class methods
   - Map dependencies and collaborators that may need mocking
   - Identify edge cases: null/empty inputs, boundary values, error conditions, async behavior

5. **Generate test files**
   - For each target file, generate a test file that covers:
     - **Happy path**: Normal expected behavior for each public function/method
     - **Error cases**: Invalid inputs, exceptions, error handling paths
     - **Boundary values**: Empty collections, zero values, max values, off-by-one scenarios
   - Follow the project's existing patterns for imports, assertions, and test organization
   - Use descriptive test names that explain the expected behavior

6. **Place test files**
   - Place test files according to the project's convention detected in Step 3
   - Show the user a summary of generated test files and their coverage before writing
   - Write the test files after user confirmation
