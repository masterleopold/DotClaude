---
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, Task
argument-hint: <ASIN> <BookTitle> [--horizontal]
description: Capture Kindle book via Playwright, extract text, create organized markdown
---

# Kindle Book Capture Workflow

You are orchestrating a 3-phase Kindle book capture: Playwright screenshots, text extraction, and markdown organization. The goal is to produce **maximally detailed, structurally rich markdown** that captures every idea, example, quote, framework, and nuance from the original book.

## Argument Parsing

Parse `$ARGUMENTS` into:
- **ASIN** (required): First argument, alphanumeric string (e.g., `B01GPCKJWK`)
- **BookTitle** (required): Second argument, Japanese or English book title (e.g., `なぜあなたの仕事は終わらないのか`)
- **--horizontal** (optional): Flag for horizontal (left-to-right) books. Default is vertical (right-to-left, Japanese 縦書き)

If ASIN or BookTitle is missing, ask the user to provide them.

Set these variables for use throughout:
- `VAULT_ROOT` = the current working directory (the Obsidian vault root)
- `CAPTURE_DIR` = `Books/files/kindle-captures/<ASIN>/`
- `SCRIPT_PATH` = `Books/files/kindle-capture.mjs`

---

## Phase 1: Playwright Capture

### 1a. Check for existing captures

First, check if screenshots already exist:

```
Glob: Books/files/kindle-captures/<ASIN>/page_*.png
```

If captures already exist, report the count and ask the user:
- **Skip capture** — proceed directly to Phase 2 with existing screenshots
- **Re-capture** — delete existing and run fresh capture

### 1b. Run the capture script

Run the Playwright capture script in the background:

```bash
node "<VAULT_ROOT>/Books/files/kindle-capture.mjs" <ASIN> [--horizontal]
```

Use `run_in_background: true` so you can guide the user while it runs.

Tell the user:
1. A browser window will open to Kindle Cloud Reader
2. Log in to Amazon if needed
3. Navigate to the **first page** they want to capture (e.g., skip cover/TOC if desired)
4. Signal readiness: `touch /tmp/kindle-ready`

### 1c. Monitor progress

After the user signals ready, periodically check the background task output. When the script finishes:
- Report total pages captured
- Report the capture directory path
- Proceed to Phase 2

---

## Phase 2: Text Extraction (Max Precision)

Extract text from captured screenshots using Claude's vision capabilities. This phase must capture **every word, every structural element, every nuance** from each page.

### 2a. Count and partition screenshots

```
Glob: Books/files/kindle-captures/<ASIN>/page_*.png
```

Sort the results numerically. Divide into 3 roughly equal batches for parallel processing. Each batch should contain consecutive pages to preserve context across page boundaries.

### 2b. Launch parallel extraction agents

Launch 3 Task agents (subagent_type: `general-purpose`) in parallel. Each agent receives:

**Prompt template for each agent:**

```
You are performing ultra-precise text extraction from Kindle book screenshots. Your output must be a COMPLETE, VERBATIM transcription — every sentence, every word, every piece of punctuation. Do NOT summarize. Do NOT skip passages. Do NOT paraphrase. Transcribe EVERYTHING.

Book: <BookTitle>
ASIN: <ASIN>

Your batch: page_XXX.png through page_YYY.png (inclusive)
Files are at: <VAULT_ROOT>/Books/files/kindle-captures/<ASIN>/

## Extraction Rules

For each page, read the PNG file with the Read tool and extract:

### Text Content (verbatim, complete)
- Transcribe ALL body text exactly as written — every sentence, every paragraph
- Preserve the original language faithfully (Japanese, English, or mixed)
- Maintain paragraph breaks as they appear on the page
- Include ALL examples, anecdotes, case studies, and illustrative stories in full
- Include ALL numerical data, statistics, percentages, and figures mentioned
- Never truncate, condense, or skip "obvious" or "repetitive" passages

### Structural Elements (annotate precisely)
- **Chapter titles**: Mark with `[CHAPTER_START: <title>]`
- **Section headings**: Mark with `[SECTION: <heading>]`
- **Subsection headings**: Mark with `[SUBSECTION: <heading>]`
- **Bold/emphasized text**: Wrap in `**bold**` markers
- **Quotes/blockquotes**: Prefix with `> `
- **Lists** (numbered or bulleted): Preserve as-is with proper indentation
- **Footnotes/endnotes**: Mark with `[NOTE: <content>]`
- **Page headers/footers**: Mark with `[HEADER: <text>]` or `[FOOTER: <text>]`

### Special Content (capture with context)
- **Tables/charts**: Reproduce as markdown tables. If complex, describe structure in `[TABLE: rows x cols, <description>]` then reproduce content
- **Diagrams/figures**: Describe in `[FIGURE: <detailed description of visual content, labels, arrows, relationships>]`
- **Illustrations with captions**: `[ILLUSTRATION: <description>] Caption: <caption text>`
- **Formulas/equations**: Reproduce in readable text form
- **Code snippets**: Wrap in triple backticks with language identifier if visible
- **URLs/references**: Capture exactly as printed

### Context Signals (for downstream chapter splitting)
- When you detect a new chapter beginning (title page, large heading, page with only a chapter title), add:
  `[CHAPTER_BOUNDARY detected — title: "<title>", page: XXX]`
- When you detect a major section shift within a chapter, add:
  `[SECTION_SHIFT — new section: "<heading>", page: XXX]`
- When text flows across a page break mid-sentence, add:
  `[CONTINUES_FROM_PREVIOUS_PAGE]` at the top of the new page

## Output Format

Return a single structured response with ALL pages in order:

## Page XXX
[complete transcribed content with all annotations]

## Page XXX
[complete transcribed content with all annotations]

... (continue for every page in your batch)

## Quality Checklist (verify before returning)
- [ ] Every page in my batch has been read and transcribed
- [ ] No sentences were skipped or summarized
- [ ] All structural markers (chapters, sections, subsections) are annotated
- [ ] Tables are reproduced with all cells
- [ ] Quotes are properly marked
- [ ] Figures/diagrams are described in detail
- [ ] Page continuations are marked
```

### 2c. Collect and merge results

Gather the output from all 3 agents. Combine them in page order into a single text stream. At merge boundaries:
- Reconnect any sentences split across agent batches
- Verify chapter boundary annotations are consistent
- Resolve any overlapping or duplicate page transcriptions

---

## Phase 3: Markdown Organization (Maximum Detail)

Transform the raw extraction into richly structured markdown files. The output quality must match or exceed the reference files in this vault (see `Books/entries/なぜあなたの仕事は終わらないのか/` for the quality standard).

### 3a. Analyze book structure

From the combined extracted text:
1. Map ALL chapter boundaries using `[CHAPTER_BOUNDARY]` markers
2. Within each chapter, map ALL sections and subsections
3. Identify the author name (from title page, preface, or colophon)
4. Extract the book's full title (including subtitle if any)
5. Note any foreword, preface, afterword, or appendix sections
6. Collect ALL key quotes (blockquoted passages, memorable statements)
7. Identify ALL frameworks, models, diagrams, and tables

### 3b. Create the hub file

Check if `Books/entries/<BookTitle>.md` already exists. If it does, update it. If not, create it.

**Hub file template:**

```markdown
---
tags:
  - source/book
  - type/guide
  - <add 2-4 relevant tags from TAG_TAXONOMY.md based on content>
Category: "<appropriate category>"
Rating: ""
author: "<author full name>"
Location: "Books/entries/<BookTitle>"
Chapters: <number of chapters>
Language: "JP"
URL: "https://www.amazon.co.jp/dp/<ASIN>"
---

# <Full Book Title including subtitle>

**著者**: <author name with brief credential if mentioned in the book>
**出版**: <publisher and year if visible in captures>
**ASIN**: <ASIN>

## 概要

<Comprehensive 4-6 sentence summary covering: what the book is about, who it's for, the core thesis, and the key methodology/framework. This should be dense with information, not generic.>

## 核心メッセージ

- **<concept>** — <one-line explanation>
- **<concept>** — <one-line explanation>
- **<concept>** — <one-line explanation>
- **<concept>** — <one-line explanation>
<List the 4-8 most important ideas/principles from the entire book. Each should be a named concept with a dash explanation.>

## チャプター

| # | タイトル | テーマ | ページ |
|---|---------|-------|--------|
| 1 | [[01_<ChapterTitle>]] | <2-3 word theme> | p.XXX-YYY |
| 2 | [[02_<ChapterTitle>]] | <2-3 word theme> | p.XXX-YYY |
...
<Include page ranges from the capture for each chapter>

## 関連

- [[<relevant Stack or Knowledge file>]] — <why it's related>
- [[<relevant Stack or Knowledge file>]] — <why it's related>
<Search Knowledge/ and Information/ Stack folders for genuinely related content. Include 3-5 wikilinks with brief relationship notes.>
```

### 3c. Create chapter files (MAXIMUM DETAIL)

Create a directory: `Books/entries/<BookTitle>/`

Each chapter file must be **exhaustively detailed**. The goal is that someone reading ONLY the chapter file gets the FULL value of reading those pages in the original book. Nothing is lost.

**Chapter file template:**

```markdown
---
tags:
  - source/book
  - type/framework
  - <1-2 topic-specific tags>
  - <action tag: action/apply, action/reference, or action/review>
---

# 第N章：<Chapter Title>

> 本書: [[<BookTitle>]] by <author>

## 要旨

<Dense 3-5 sentence summary of THIS chapter's argument, thesis, and contribution to the book's overall message. Not generic — capture the specific logic chain.>

## セクション構成

### <Section Heading from the book>

<FULL content of this section. Include:
- Complete argument/narrative (every key point, not just summaries)
- All examples and anecdotes with their full context and punchlines
- All statistics, data points, and numerical evidence
- Direct quotes from the author as blockquotes (> )
- Comparisons and contrasts with other ideas/approaches>

**サブセクション: <Subsection heading if any>**
<Full content of subsection>

### <Next Section Heading>

<Continue with same exhaustive detail for every section...>

<For sections that contain comparative or structured information, ADD TABLES:>

| <Column1> | <Column2> | <Column3> |
|------------|-----------|-----------|
| <data>     | <data>    | <data>    |

<For sections that describe processes, mental models, or frameworks, ADD ASCII DIAGRAMS:>

```
<ASCII diagram representing the framework, process flow, cycle,
 or mental model described in this section.
 Use boxes, arrows, labels to visualize the concept.>
```

<For memorable or pivotal author statements, USE BLOCKQUOTES:>

> <Exact quote from the author — these are high-value passages that
>  capture a key insight in the author's own words>

## フレームワーク

<If the chapter introduces any framework, model, method, or structured approach, create a visual representation:>

```
<Framework name>:
  <Visual ASCII representation showing the components,
   relationships, flow, or hierarchy of the framework.
   Use indentation, arrows (→, ←, ↑, ↓), boxes (┌┐└┘│─),
   and labels to make it clear and scannable.>
```

<If the chapter has multiple frameworks, include ALL of them with separate code blocks.>
<If the chapter has no explicit framework, OMIT this section entirely.>

## キーメッセージ

> <The single most important quote or insight from this chapter,
>  in the author's original words>

<If there are additional crucial quotes (2-3 max), include them too:>

> <Another pivotal quote>
```

### Critical quality rules for chapter files:

1. **NEVER summarize away detail** — If the author spends 2 pages on an anecdote, include the full anecdote with setup, conflict, and punchline
2. **EVERY section heading from the book becomes a `###` heading** — Do not merge or skip sections
3. **EVERY subsection becomes a bold subheading** — `**サブセクション: <title>**`
4. **ALL examples and case studies in full** — Names, companies, numbers, outcomes
5. **ALL direct quotes as blockquotes** — If the author emphasizes something, it goes in `> `
6. **Tables for ANY comparative or structured information** — Side-by-side comparisons, criteria lists, feature matrices, timeline items
7. **ASCII diagrams for ANY visual concept** — Processes, cycles, hierarchies, mental models, frameworks. Use box-drawing characters (┌┐└┘│─├┤┬┴┼) and arrows (→←↑↓)
8. **Preserve numerical specificity** — "18分" not "短い仮眠", "20倍界王拳" not "全力集中"
9. **Link back to hub file** — Every chapter starts with `> 本書: [[<BookTitle>]] by <author>`
10. **Include page references** where useful for particularly important frameworks or quotes

### File naming:
- `01_<ChapterTitle>.md`, `02_<ChapterTitle>.md`, etc.
- Use the book's own chapter titles in Japanese
- Keep filenames under 80 characters
- Replace `/\:*?"<>|` with `･` (U+FF65) per vault conventions

### 3d. Handle special sections

- **Foreword/Preface (はじめに/まえがき)**: Create as `00_はじめに.md` — include full content, author's stated purpose
- **Afterword (おわりに/あとがき)**: Create as the last numbered file — include full content
- **Appendix (付録)**: Create as `XX_付録.md` — include all reference material, lists, resources

### 3e. Final report

After creating all files, report:
- Hub file path with complete chapter count
- List of ALL created chapter files with their line counts
- Total extracted content volume (approximate word/character count)
- Any pages where extraction quality was uncertain — flag specific page numbers
- Relevant `[[wikilinks]]` to existing vault content discovered during organization

---

## Error Handling

- If the capture script fails, show the error and suggest: check if Playwright is installed (`npx playwright install chromium`), verify ASIN is correct
- If text extraction quality is poor for specific pages, flag those page numbers and suggest manual review
- If chapter boundaries are ambiguous, create a best-effort split and include `<!-- CHAPTER_BOUNDARY_UNCERTAIN: pages XXX-YYY -->` comments
- If the book has an unusual structure (no clear chapters, continuous essay, etc.), adapt the organization while maintaining maximum detail — use logical section breaks instead
