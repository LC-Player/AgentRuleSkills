---
name: unity-gi-knowledge-logger
description: Log Unity knowledge Q&A into the Obsidian vault at C:\Users\ziyan.wang\Documents\Unity文档. Use when the user asks questions about TuanjieGI/Unity GI source code, rendering, SDF, Facet, ray tracing, or any Unity engine-related topic, and the answers should be persisted as knowledge notes. The skill automatically selects the appropriate subfolder and file (or creates new ones) under the vault root.
---

# Unity Knowledge Logger

When the user asks a question about Unity/TuanjieGI and you provide an answer, persist that knowledge in the Obsidian vault. This skill must be used proactively and automatically — do not wait for the user to ask you to log the answer.

## Vault Root

`C:\Users\ziyan.wang\Documents\Unity文档`

All knowledge notes are saved **somewhere under this root**. The skill automatically determines which subfolder and file to use.

## Workflow

1. **Scan the vault root** — use `list_directory` on `C:\Users\ziyan.wang\Documents\Unity文档` to see all subfolders.
2. **For each subfolder** that could be relevant to the question's topic, use `list_directory` to enumerate its files. Start with the most likely subfolder (see "Subfolder Selection" below) and read candidate file titles.
3. **Read candidate files** to find the best match. If a question is strongly related to an existing document, read its content to determine whether the question stems from an unclear explanation of a specific concept in that document.
4. **Choose the subfolder** — decide which subfolder under the vault root is the best home for this knowledge (see "Subfolder Selection").
5. **Choose the write strategy** within that subfolder (see below).
6. **Write the content** using `write_file` (new files) or `replace` (editing existing files).
7. **Inform the user** which file was created/updated, in which subfolder, and what was changed.

## Subfolder Selection

The skill must **automatically** choose the best subfolder under `C:\Users\ziyan.wang\Documents\Unity文档`. The following guidelines help decide:

1. **Scan all existing subfolders** via `list_directory` on the vault root.
2. **Match by topic keywords** — read subfolder names and use your judgment:
   - Subfolder names containing "GI", "光照", "渲染", "ray tracing", "SDF", "Facet" → likely for GI/lighting/rendering topics.
   - Subfolder names containing "Shader", "Compute", "Material", "Pipeline" → likely for shader/render-pipeline topics.
   - Subfolder names containing "Asset", "Editor", "Tool" → likely for editor/tooling topics.
   - Use general semantic matching for any other subfolder names.
3. **If no existing subfolder fits**, create a new subfolder with a concise, descriptive name (e.g., `Unity Shader 学习`, `APS系统`, `Game Play`). Create the folder implicitly by writing the first file into it (`write_file` to a path like `C:\Users\ziyan.wang\Documents\Unity文档\<new subfolder>\<file>.md`).

## Write Strategy: In-Place Supplement vs. New Section vs. New File

### Strategy A: In-Place Supplement (preferred when applicable)

Use when ALL of the following are true:
- The question is strongly related to an existing document
- The question stems from a concept in that document being explained unclearly or incompletely
- The answer clarifies, expands, or corrects that specific concept

**Action**: Read the file, find the specific section/paragraph that was unclear, and use `replace` to supplement it directly — add clarifying text, examples, diagrams, or callout blocks inline. Do NOT create a separate Q&A section.

Example: The user asks "我不明白蒙特卡洛方向采样策略" and the document `Quad上的球面蒙特卡洛采样.md` has a section about it that doesn't clarify the detail. → Edit that section directly to add a clarifying note.

### Strategy B: New Section (when question is a new sub-topic)

Use when:
- The question is related to an existing document's topic
- But the question introduces a new sub-topic not covered in the document
- The answer doesn't modify any existing explanation

**Action**: Append a new `##` section at the end of the existing file using the Q&A entry format below.

### Strategy C: New File (when no existing file matches)

Use when no
- existing file in any subfolder is topically related to the question
- adding any content into the proper file may cause "god file" (see below).

**Action**: Create a new `.md` file with a descriptive name in the chosen subfolder.

## Topic Jump Awareness and File Selection Discipline

### Re-Evaluate on Every Question

**The file you just wrote to is NOT the default for the next question.** Each new user question must go through the full file-selection workflow (scan vault → match subfolder → read candidate files → choose strategy). This applies even when:

- The user is asking a follow-up question that seems related to the previous one.
- The user references a concept from the previous answer.
- The conversation feels like a continuous thread.

**Anti-pattern**: "I just wrote to `SDF 加速结构详解.md`, so the next SDF-adjacent question should also go there." — Wrong. Read the file first; the answer may belong in a different file, or may warrant a new file entirely.

### Avoid God Files

A "god file" is a single document that accumulates every loosely-related topic until it becomes an unstructured dumping ground. Prevent this by enforcing these rules:

1. **Scope check before appending**: Before appending a new section to an existing file, ask: *"Does this new content belong within this file's scope, or does it deserve its own file?"* If the answer leans toward "its own file," create a new file.
2. **Title alignment check**: If the new content's natural title doesn't fit under the existing file's title, it likely doesn't belong in that file. E.g., adding "Probe 采样优化" content to a file titled `SDF 加速结构详解.md` is a poor fit — consider a new `APS 探针采样优化.md`.
3. **Prefer multiple focused files over one large file**: Three well-scoped files of 200 lines each are better than one 600-line file. Obsidian's `[[wikilinks]]` and search make cross-referencing trivial.
4. **Split signal**: If an existing file has grown past ~400 lines or covers more than 3 distinct sub-topics, proactively suggest splitting it to the user, or create the next related topic as a new file with a `[[wikilink]]` pointing back.

## Markdown Formatting Rules

The following rules ensure documents remain clean, readable, and maintainable in Obsidian. **Violating these rules is a bug in the output — always check your generated Markdown before writing.**

### No Numbered Headings

- Do NOT prefix headings with numbers or auto-numbering (e.g., `## 1. Overview`, `## 2.1 SDF 原理`). Obsidian and Markdown have no built-in numbering — manual numbers break when sections are inserted, reordered, or removed.
- Use plain semantic headings: `## Overview`, `### SDF 原理`.
- If sequential ordering matters for understanding, let the narrative flow or numbered list items (within a section) convey order — not the heading itself.

### Sparing Use of Horizontal Rules (`---`)

- Do NOT insert `---` before every `##` section. This creates visual clutter and noise.
- Use `---` **only** in these specific cases:
  1. Separating YAML frontmatter from body content (required).
  2. Separating distinct Q&A entries within the same file when using Strategy B (one `---` between entries, not before the first entry).
  3. Separating major topic boundaries that are genuinely unrelated (use sparingly — if sections are related, they don't need a rule).
- When in doubt, omit the `---`. Headings alone provide sufficient visual separation.

### General Markdown Hygiene

- Use `[[wikilinks]]` to cross-reference other notes in the vault (e.g., `[[Facet 系统源码导读]]`, `[[TuanjieGI Ray Tracing 流程详解]]`). Wikilinks work across subfolders within the same Obsidian vault.
- Use `> [!note]`, `> [!tip]`, `> [!info]` callout blocks for key insights.
- Use `> [!question]` for the user's original question (only in new sections, not in-place supplements).
- Use standard Markdown tables, code blocks, and mermaid diagrams.
- **Diagrams**: When you need to draw flowcharts, class diagrams, sequence diagrams, state diagrams, or any other kind of diagram, **prefer Mermaid syntax** (`mermaid` code blocks). Obsidian renders Mermaid natively. Do not use ASCII art, unless Mermaid cannot express the needed structure.
  - note: this is only for this skill and obsidian. don't apply this rule when you want to output diagrams to user in console directly. (use ASCII/arrow symbol or other displayable characters instead)
  - but when the user wants you to render diagram via ASCII or other tool (like external tools, image link, do so).
- Add YAML frontmatter to new files only:
  ```yaml
  ---
  tags:
    - unity
    - <subfolder-topic-tag>
  aliases:
    - <short description>
  ---
  ```
  Use `unity-gi` / `tuanjiegi` as tags when saving to the `unity GI 光照学习` subfolder. For other subfolders, use appropriate tags based on the topic.
- For in-place supplements and appended sections, do NOT add frontmatter.
- Use `**bold**` for key terms, `==highlight==` for critical points.
- Keep code blocks language-tagged (```cpp, ```mermaid, etc.).

## Q&A Entry Format (for Strategy B and C only)

When appending a new section to an existing file:

```markdown
## <Short Question Title>

> [!question] <Full user question>

<Answer content with code blocks, diagrams, tables as needed>

> [!tip] Key Takeaway
> <One-sentence summary of the key insight>
```

> **Note on `---` placement**: When appending multiple Q&A entries to the same file via Strategy B, insert a single `---` between entries to separate them. Do NOT add `---` before the first entry or after the last one.

When creating a new file:

```markdown
---
tags:
  - unity
  - <topic-tag>
aliases:
  - <short description>
---

# <Topic Title>

## <Short Question Title>

> [!question] <Full user question>

<Answer content>

> [!tip] Key Takeaway
> <One-sentence summary>
```