---
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Task, AskUserQuestion
argument-hint: [source] [BookID/Title] [--horizontal]
description: Capture book from Kindle Cloud Reader, Mac Kindle, or Apple Books → OCR → structured Markdown
---

# Book Capture Workflow

Orchestrate the full pipeline: screenshot capture → OCR text extraction → structured Markdown generation.

All AI-powered steps (OCR re-reading, content analysis, markdown generation) use **Claude Code agents** — no external API keys required.

Supports 3 sources:
- **cloud**: Kindle Cloud Reader (via Playwright)
- **kindle**: Mac Kindle app (via CGWindowList + screencapture + Page Down navigation)
- **books**: Apple Books app (via AppleScript + screencapture)

## Setup

Set working variables:
- `VAULT_ROOT` = the root of the Obsidian vault (parent of Books/)
- `FILES_DIR` = `Books/files/`
- `CAPTURES_BASE` = `Books/files/kindle-captures/`

## Step 1: Gather Information

Parse `$ARGUMENTS` for optional pre-filled values. Then ask the user for any missing information:

**Required:**
1. **Source app**: cloud / kindle / books
2. **Book ID**: ASIN (for cloud/kindle) or Book Title (for books)
3. **Book Title**: Full title in Japanese (for all sources)
4. **Author**: Author name
5. **Category**: One of the vault categories (AI, Marketing, Strategy, Finance, etc.)
6. **Location**: Where topic files should live (e.g., `Knowledge/Marketing/トラクション`)

**Optional (with defaults):**
7. **Direction**: `--horizontal` for LTR books, default is RTL (縦書き)
8. **Language**: JP (default) or EN
9. **URL**: Amazon URL (auto-generated from ASIN if available)

If the user provides arguments like `cloud B0746JCN8B`, parse them. For anything missing, use AskUserQuestion.

## Step 2: Capture Screenshots

### Check for existing captures

```
Glob: Books/files/kindle-captures/<BookID>/page_*.png
```

If captures exist, ask the user: **Skip capture** (use existing) or **Re-capture** (fresh).

### Run capture based on source

**cloud** (Kindle Cloud Reader):
```bash
node "Books/files/kindle-capture.mjs" <ASIN> [--horizontal]
```
Run in background. Guide user through login + signal (`touch /tmp/kindle-ready`).

**kindle** (Mac Kindle app):
```bash
node "Books/files/capture-kindle-mac.mjs" <BookID> [--horizontal]
```
Remind user: open the book in **Amazon Kindle** app to the first page, ensure Accessibility permission for Terminal.

**Known Kindle Mac quirks:**
- App name is `Amazon Kindle` (not `Kindle`)
- Window ID detection uses CGWindowList via inline Swift (AppleScript `id of window` not supported)
- Page navigation uses **Page Down** key (arrow keys and spacebar don't work reliably)
- Dismiss any startup dialogs first (script sends Escape automatically, but check)

**books** (Apple Books):
```bash
node "Books/files/capture-books-app.mjs" "<BookTitle>" [--horizontal]
```
Remind user: open the book in Apple Books to the first page, ensure Accessibility permission.

Monitor progress. When complete, report page count.

## Step 3: OCR Text Extraction

### 3a. Run Vision OCR (local, fast)

```bash
node "Books/files/extract-text.mjs" "Books/files/kindle-captures/<BookID>" --concurrency 5 --claude-threshold 0.6
```

This runs macOS Vision framework on all pages. Output: `raw_text.json` with per-page text and confidence scores. Pages below the confidence threshold are flagged as `vision-low`.

**IMPORTANT**: macOS Vision **cannot read 縦書き (vertical Japanese text)** — it only captures horizontal headers/footers. For 縦書き books, most pages will be flagged as low-confidence or empty.

### 3b. Re-read low-quality pages with Claude Code agents

After Vision OCR completes, check the stats. If there are `vision-low` or `failed` pages (or if the book is 縦書き), dispatch Claude Code agents to re-read those pages using the Read tool (multimodal image reading).

**For 縦書き books**: Skip the Vision results entirely. Re-read ALL pages with agents.

**For 横書き books**: Only re-read pages where `method` is `vision-low` or `failed`.

Workflow:
1. Read `raw_text.json` to identify pages needing re-reading
2. Dispatch 4-8 parallel Task agents (subagent_type: `general-purpose`), each handling ~20-30 pages
3. Each agent:
   - Reads assigned page images via the Read tool
   - Extracts ALL text faithfully from each page
   - Writes output to `/tmp/ocr_batch_NN.json`
4. Merge batch results back into `raw_text.json`

**Agent prompt template:**

```
You are extracting text from book page screenshots. Read each image with the Read tool and transcribe ALL text faithfully.

Book: "<BookTitle>"
Your pages: page_XXX.png through page_YYY.png
Directory: <VAULT_ROOT>/Books/files/kindle-captures/<BookID>/

For each page:
1. Read the PNG file with Read tool
2. Transcribe ALL visible text exactly as written
3. Preserve paragraph breaks and structure
4. Note chapter boundaries with [CHAPTER: <title>]
5. Note section headings with [SECTION: <heading>]

Output: Write a JSON file to /tmp/ocr_batch_NN.json with format:
[
  { "page": N, "text": "full text...", "confidence": 1.0, "method": "claude-vision" },
  ...
]
```

**Merge script** (run after all agents complete):
```javascript
// Read all /tmp/ocr_batch_*.json files, merge into raw_text.json
// For re-read pages, replace the vision-low entries with claude-vision entries
// Keep existing vision (good) entries unchanged
```

Report stats when complete (total pages, vision vs agent-read breakdown).

## Step 4: Generate Structured Markdown (Claude Code Agents)

This step uses Claude Code agents to analyze the OCR text and generate structured Obsidian markdown. No external API key needed.

### 4a. Plan thematic structure

Read `raw_text.json` and concatenate all page text. Then analyze the full text yourself (or via a Task agent) to identify **8-14 thematic categories** that best organize the book's knowledge. Group by information type (methodology, case studies, frameworks, principles, etc.), NOT by original chapter order.

Produce a plan:
```json
{
  "themes": [
    { "id": "01", "title": "テーマ名（日本語）", "description": "内容説明", "pageRanges": "5-20, 45-60" },
    ...
  ],
  "bookSummary": "本の概要（2-3文）",
  "tags": ["source/book", "type/framework", "theme/innovation"]
}
```

Save this to `Books/files/kindle-captures/<BookID>/structured.json`.

### 4b. Generate topic files (parallel agents)

Dispatch 3-5 parallel Task agents (subagent_type: `general-purpose`), each handling 2-4 themes. Each agent:

1. Receives the full text (or relevant page ranges) and assigned theme details
2. Generates richly structured Markdown content for each theme
3. Writes files directly to `<Location>/NN_テーマ名.md`

**Topic file format:**
```markdown
---
tags:
  - source/book
  - type/framework
  - theme/<relevant>
parent: "[[<BookTitle>]]"
---

<Rich markdown content with ##/###/#### headings, **bold** key terms,
> blockquotes for notable statements, tables, lists, etc.>
```

**Agent content guidelines:**
- Be SUPER DETAILED — this is a reference document, not a summary
- Use ## for main sections, ### for subsections
- Use **bold** for key terms and important phrases
- Use > blockquotes for direct quotes and notable statements
- Use tables for comparisons, frameworks, matrices
- Include specific names, companies, numbers, statistics
- Preserve all frameworks, models, and step-by-step processes
- Do NOT include OCR artifacts ([章扉], [目次], --- PAGE X --- etc.)

### 4c. Generate hub file

After all topic files are created, write the hub file at `Books/entries/<BookTitle>.md`:

```markdown
---
tags:
  - source/book
  - type/framework
  - theme/<relevant>
Category: "<Category>"
Rating: ""
author: "<Author>"
Location: "<Location>"
Chapters: <number of topics>
Language: "<Language>"
URL: "<URL>"
---

# <BookTitle>

<Book summary (2-3 sentences)>

## Topics

- [[01_テーマ名]] — テーマの説明
- [[02_テーマ名]] — テーマの説明
...
```

**Filename rules:**
- Sanitize: replace `/\:*?"<>|` with `･` (U+FF65), max 80 chars
- Topic files: `NN_テーマ名.md` (NN = 01, 02, ...)
- Hub file: `<BookTitle>.md` in `Books/entries/`

## Step 5: Report Results

After all phases complete, report:
- Total pages captured
- OCR stats (Vision good / Vision low / agent-read)
- Number of topic files generated with their paths
- Hub file path: `Books/entries/<BookTitle>.md`
- Topic files directory: `<Location>/`
- Any issues or warnings

Suggest the user:
1. Open the hub file in Obsidian to verify
2. Review topic files for content quality and completeness
3. Add a Rating to the hub file frontmatter
4. Check wikilinks are working correctly

## Error Handling

- **npm packages not installed**: Run `cd "Books/files" && npm install`
- **Playwright not installed**: Run `npx playwright install chromium`
- **Accessibility permission denied**: Guide to System Settings → Privacy & Security → Accessibility
- **Vision OCR compilation fails**: Check Xcode Command Line Tools (`xcode-select --install`)
- **Vision OCR misses vertical text**: Expected for 縦書き — dispatch Claude Code agents for those pages
- **Kindle window not found**: Ensure book is open and visible in Amazon Kindle app. Check that no dialogs are blocking
- **Kindle page advance fails**: Script uses Page Down key. If stuck, try restarting Kindle app
- **Capture stuck/timeout**: Use `--max-pages` limit or kill process + resume OCR on existing captures
- **Agent text extraction incomplete**: Re-dispatch agents for specific page ranges
