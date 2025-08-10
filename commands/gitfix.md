---
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git commit:*), Read, Edit, MultiEdit, Grep, Glob, TodoWrite
argument-hint: [issue description]
description: Fix a git issue and commit changes with careful planning
---

Fix the following issue in the codebase: $ARGUMENTS

## Approach
Check codebase and read documents carefully and plan refactoring. The refactoring MUST preserve the existing functionality and style without causing any bugs/errors.

## Steps to follow:

1. **Analyze and understand the codebase**
   - Read relevant documentation (README, CONTRIBUTING, etc.)
   - Study the existing code structure and patterns
   - Understand the current functionality before making changes
   - Identify the coding style and conventions used

2. **Plan the fix carefully**
   - Use TodoWrite to create a detailed plan
   - Identify all files that need to be modified
   - Consider potential side effects and dependencies
   - Ensure the fix aligns with existing architecture

3. **Implement the fix**
   - Make necessary code changes following existing patterns
   - Preserve the current coding style and conventions
   - Maintain backward compatibility
   - Add proper error handling if needed

4. **Verify the changes**
   - Test the functionality if possible
   - Check that no existing features are broken
   - Ensure code style consistency
   - Review changes for potential bugs

5. **Commit the changes**
   - Create a descriptive commit message
   - Include details about what was fixed and why
   - Reference any related issues if applicable