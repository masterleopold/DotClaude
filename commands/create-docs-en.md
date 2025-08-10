---
allowed-tools: Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
argument-hint: [optional: specific section to focus on]
description: Create English developer documentation with flowcharts
---

With reading CLAUDE.md, README.md, and all documents under /docs directory and check codebase, create helpful, stylish document pages for developer including flowchart of user and data. All content should be written in English keeping the same style and color scheme with the app.

## Implementation Steps:

1. **Analyze existing documentation and codebase**
   - Read CLAUDE.md, README.md thoroughly
   - Scan all documents under /docs directory
   - Extract from codebase:
     * Application color scheme (CSS variables, theme settings)
     * UI component styles and design patterns
     * Architecture patterns and structure
     * Data flow and state management
     * User authentication/authorization flow

2. **Design document structure**
   - Create `/docs/en/` directory for English docs
   - Prepare following documents:
     * `overview.md` - Project Overview
     * `architecture.md` - Architecture Guide
     * `user-flow.md` - User Flow Diagrams
     * `data-flow.md` - Data Flow Diagrams
     * `api-reference.md` - API Reference
     * `setup-guide.md` - Setup Guide
     * `contributing.md` - Contributing Guide

3. **Create flowcharts using Mermaid**
   - User flow diagram:
     ```mermaid
     graph TD
       A[User Access] --> B{Authenticated?}
       B -->|Yes| C[Dashboard]
       B -->|No| D[Login Page]
       D --> E[Authentication Process]
       E --> F{Auth Success?}
       F -->|Yes| C
       F -->|No| D
     ```
   - Data flow diagram:
     ```mermaid
     graph LR
       A[Frontend] --> B[API Gateway]
       B --> C[Business Logic]
       C --> D[Database]
       D --> C
       C --> B
       B --> A
     ```

4. **Extract and apply app styling**
   - Identify color scheme from:
     * CSS/SCSS files
     * Tailwind config
     * Theme configuration files
   - Create consistent styling:
     ```html
     <style>
       :root {
         --primary-color: #xxx;
         --secondary-color: #xxx;
         --accent-color: #xxx;
         --bg-color: #xxx;
         --text-color: #xxx;
       }
     </style>
     ```

5. **Create clear English content**
   - Write concise, technical documentation
   - Use consistent terminology throughout
   - Include code examples with explanations
   - Follow documentation best practices

6. **Add developer-friendly features**
   - **Quick Start** - Get running in 5 minutes
   - **Troubleshooting** - Common issues and solutions
   - **Performance Optimization** - Optimization techniques
   - **Best Practices** - Recommended patterns and approaches
   - **Code Snippets** - Reusable code examples
   - **Debugging Guide** - Debugging strategies and tools

7. **Enhance with interactive elements**
   - Copyable code blocks with syntax highlighting
   - Collapsible sections for detailed information
   - Internal navigation links and anchors
   - Searchable index/table of contents
   - Tabs for different code examples (JavaScript/TypeScript)
   - Version selectors for different API versions

8. **Visual enhancement**
   - Use CSS classes for visual distinction (info, warning, error, success)
   - Color-coded importance levels with background highlights
   - Progress indicators for multi-step processes
   - Status badges (Stable, Beta, Deprecated, Experimental)
   - Code diff highlighting for changes
   - Box shadows and borders for section separation
   - Custom bullet points and numbered lists
   - Highlighted key terms and important notes
   - Callout boxes for tips and warnings

9. **Generate comprehensive documentation**
   - Include real examples from the codebase
   - Add architecture diagrams and screenshots
   - Create decision trees for common scenarios
   - Document error codes with explanations and solutions
   - Include performance benchmarks where relevant
   - Add migration guides for version updates

Focus area (if specified): $ARGUMENTS