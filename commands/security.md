---
allowed-tools: Read, Grep, Glob
argument-hint: [file or directory to scan]
description: Scan for security vulnerabilities
---

Perform security analysis on $ARGUMENTS:

1. Check for hardcoded secrets or API keys
2. Identify potential SQL injection vulnerabilities
3. Look for unsafe file operations
4. Check for exposed sensitive data
5. Review authentication and authorization patterns
6. Suggest security improvements