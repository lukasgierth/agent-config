---
name: add-tests
description: Use when asked to add, write, or generate tests for a function, module, or file. Follows the existing test style and framework in the repo.
---

# Add Tests

## Goal

Write tests for specified code that fit naturally into the existing test suite.

## Workflow

1. Identify the target (function, class, or module) the user wants tested.
2. Find the existing test directory and a representative test file to understand:
   - Test framework in use (pytest, unittest, jest, vitest, cargo test, etc.)
   - Naming conventions (file names, test function names, fixture patterns)
   - How imports and setup are structured
3. Read the target code thoroughly to understand its behaviour, edge cases, and error paths.
4. Plan the test cases:
   - Happy path(s)
   - Edge cases (empty input, boundary values, etc.)
   - Error/exception paths
5. Write the tests following the existing style exactly — same indentation, same assertion style, same fixture approach.
6. Run the test suite to confirm the new tests pass (`pytest`, `npm test`, `cargo test`, etc.).
7. Report which tests were added and their results.

## Rules

- Do not introduce new test dependencies unless the existing suite already uses them or the user explicitly approves.
- Prefer simple, readable tests over clever ones.
- Each test should test one thing.
