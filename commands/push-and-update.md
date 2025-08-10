---
allowed-tools: Bash(git:*), Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite
argument-hint: [commit message or PR description]
description: Push to GitHub and update all documentation
---

Push to github and update CLAUDE.md, README.md, and all documents under /docs directory if needed.

## Steps:

1. **Update documentation files**
   - Update CLAUDE.md with latest session information
   - Review and update README.md if project structure changed
   - Check all files under /docs directory for needed updates
   - Document any new features, configurations, or important changes

2. **Review current changes**
   - Run `git status` to see all modified files
   - Run `git diff` to review uncommitted changes
   - Ensure all necessary files are tracked

3. **Prepare documentation updates**
   - Add session summary to CLAUDE.md
   - Update README.md with:
     * New features or functionality
     * Updated setup instructions
     * Changed dependencies
   - Update /docs files:
     * Command documentation
     * API changes
     * Configuration updates
     * Architecture decisions

4. **Stage and commit changes**
   - Stage all relevant files with `git add`
   - Create descriptive commit message
   - Include context: ${ARGUMENTS:-"Update documentation and push changes"}

5. **Push to GitHub**
   - Check current branch with `git branch`
   - Pull latest changes with `git pull` (if needed)
   - Push to remote with `git push`
   - If on new branch, use `git push -u origin branch-name`

6. **Verify push success**
   - Confirm push completed successfully
   - Check remote repository status
   - Note any merge conflicts or issues

7. **Post-push documentation**
   - Document the push in CLAUDE.md
   - Include commit hash for reference
   - Note any important decisions or changes made