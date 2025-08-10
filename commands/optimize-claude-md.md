---
allowed-tools: Read, Write, Edit, MultiEdit, Bash(wc:*), Bash(du:*), Grep, Glob, TodoWrite
argument-hint: [optional: max size in KB for CLAUDE.md]
description: Optimize CLAUDE.md size while preserving information
model: claude-opus-4-1-20250805
---

We need to optimize CLAUDE.md because it's too large and could affect Claude Code performance. But we need to keep those info in README.md or separated document. Figure out the best strategy. Ultrathink.

## Deep Analysis & Strategy Development:

1. **Analyze current CLAUDE.md**
   - Check file size and line count
   - Identify content categories (commands, patterns, conventions, history, etc.)
   - Assess information criticality (essential vs. nice-to-have)
   - Evaluate redundancy and outdated information

2. **Develop optimization strategy**
   - Think deeply about the best structure for documentation
   - Consider these organization patterns:
     * Keep only critical, frequently-needed info in CLAUDE.md
     * Move historical/session logs to `docs/sessions/`
     * Move command references to `docs/commands.md`
     * Move coding patterns to `docs/patterns.md`
     * Move project-specific info to README.md
     * Create `.claude/templates/` for reusable prompts

3. **Information categorization**
   - **Essential for CLAUDE.md** (keep):
     * Current project context
     * Active conventions and patterns
     * Critical commands to run (lint, test, build)
     * Important warnings or gotchas
     * Quick reference for common tasks
   
   - **Move to README.md**:
     * Project overview and setup
     * Architecture decisions
     * Public-facing documentation
   
   - **Move to separate docs**:
     * Session histories → `docs/claude-sessions/`
     * Detailed patterns → `docs/development-patterns.md`
     * Command history → `docs/command-log.md`
     * Learning notes → `docs/learnings.md`

4. **Implement the reorganization**
   - Create necessary directories and files
   - Move content to appropriate locations
   - Update CLAUDE.md with links to moved content
   - Ensure CLAUDE.md stays under 100 lines (or specified limit)
   - Add a table of contents for easy navigation

5. **Create index system**
   - Add a brief index in CLAUDE.md pointing to other docs
   - Use clear naming conventions for split documents
   - Maintain cross-references between documents

6. **Optimization targets**
   - CLAUDE.md: < 100 lines or ${ARGUMENTS:-50}KB
   - Focus on actionable, current information
   - Remove duplicate or outdated content
   - Use concise language and bullet points

7. **Validate the optimization**
   - Ensure no critical information is lost
   - Verify all links and references work
   - Check that CLAUDE.md loads quickly
   - Confirm Claude Code can efficiently parse the optimized structure