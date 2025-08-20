---
allowed-tools: Bash, Read, Grep, Glob, TodoWrite
argument-hint: [optional: specific file or directory to check]
description: Check codebase for TypeScript type errors and report findings
---

Check the codebase for TypeScript type errors: $ARGUMENTS

## Approach
Systematically check the codebase for TypeScript compilation errors and provide detailed reporting of any issues found.

## Steps to follow:

1. **Detect project configuration**
   - Check for tsconfig.json and project setup
   - Identify TypeScript files in the codebase
   - Determine the appropriate type checking strategy

2. **Run comprehensive type checking**
   - Execute `npx tsc --noEmit` for full type checking
   - If specific files/directories provided, focus on those areas
   - Capture all type errors and warnings

3. **Analyze and categorize errors**
   - Group errors by file and type
   - Identify the most common error patterns
   - Prioritize critical vs. minor issues

4. **Report findings**
   - Provide summary of total errors found
   - List errors by file with line numbers
   - Suggest potential fixes for common issues
   - Recommend next steps for resolution

5. **Additional checks**
   - Verify package.json scripts for type checking
   - Check for any eslint or other linting errors that might be related
   - Ensure all dependencies have proper type definitions