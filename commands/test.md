---
allowed-tools: Bash, Read, Write
argument-hint: [test file or directory]
description: Run tests and create test files if needed
---

Run tests for $ARGUMENTS:

1. Check for existing test files
2. Run appropriate test commands (npm test, pytest, etc.)
3. Create test files if they don't exist
4. Ensure tests pass