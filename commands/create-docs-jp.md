---
allowed-tools: Read, Write, Edit, MultiEdit, Glob, Grep, TodoWrite, Task
argument-hint: [optional: specific section to focus on]
description: Create Japanese developer documentation with flowcharts
---

With reading CLAUDE.md, README.md, and all documents under /docs directory and check codebase, create helpful, stylish document pages for developer including flowchart of user and data. All content should be written in Japanese keeping the same style and color scheme with the app.

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
   - Create `/docs/ja/` directory for Japanese docs
   - Prepare following documents:
     * `overview.md` - プロジェクト概要
     * `architecture.md` - アーキテクチャ説明
     * `user-flow.md` - ユーザーフロー図
     * `data-flow.md` - データフロー図
     * `api-reference.md` - API リファレンス
     * `setup-guide.md` - セットアップガイド
     * `contributing.md` - コントリビューションガイド

3. **Create flowcharts using Mermaid**
   - User flow diagram:
     ```mermaid
     graph TD
       A[ユーザーアクセス] --> B{認証済み？}
       B -->|Yes| C[ダッシュボード]
       B -->|No| D[ログイン画面]
       D --> E[認証処理]
       E --> F{認証成功？}
       F -->|Yes| C
       F -->|No| D
     ```
   - Data flow diagram:
     ```mermaid
     graph LR
       A[フロントエンド] --> B[API Gateway]
       B --> C[ビジネスロジック]
       C --> D[データベース]
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

5. **Create Japanese content**
   - Write clear, professional Japanese
   - Keep code examples in English
   - Include both Japanese and English for technical terms
   - Example format: 認証 (Authentication)

6. **Add developer-friendly features**
   - **クイックスタート** - 5-minute setup guide
   - **トラブルシューティング** - Common issues and solutions
   - **パフォーマンス最適化** - Optimization techniques
   - **ベストプラクティス** - Best practices collection
   - **コードスニペット** - Useful code snippets
   - **デバッグガイド** - Debugging strategies

7. **Enhance with interactive elements**
   - Copyable code blocks with syntax highlighting
   - Collapsible sections for detailed information
   - Internal navigation links
   - Searchable index/table of contents
   - Tabs for different code examples (JavaScript/TypeScript)

8. **Visual enhancement**
   - Use CSS classes for visual distinction (info, warning, error, success)
   - Color-coded importance levels with background highlights
   - Progress indicators for multi-step processes
   - Status badges (Stable, Beta, Deprecated)
   - Code diff highlighting for changes
   - Box shadows and borders for section separation
   - Custom bullet points and numbered lists
   - Highlighted key terms and important notes

9. **Generate comprehensive documentation**
   - Include real examples from the codebase
   - Add screenshots or diagrams where helpful
   - Create decision trees for common scenarios
   - Document error codes and their meanings

Focus area (if specified): $ARGUMENTS