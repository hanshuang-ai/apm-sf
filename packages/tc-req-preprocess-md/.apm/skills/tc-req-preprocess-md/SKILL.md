---
name: tc-req-preprocess-md
description: 在将包含嵌入式 UI 原型、流程图图像和表格的 `.docx` 需求文档转换为下游工具和 LLM 可以直接消费的干净 Markdown 文档时使用。它按顺序提取结构、表格和图像，编写一个自包含的 `.md` 以及一个资源文件夹，并为每个图像附加结构化说明，以便读者无需打开原始文件即可理解。
---

# Converting DOCX Requirements to Markdown

## Overview

A `.docx` is an OOXML package — XML for structure, a media folder for images, a rels file that wires them together. To produce a Markdown document that is actually useful downstream, walk that package in document order, convert each block to its Markdown equivalent, write images to a sibling folder with stable filenames, and attach a structural caption to every image so it can still be understood when the image cannot be rendered (LLM input, plain text view, etc.).

**Core principle:** Fidelity first. Convert what is there in the order it appears. Do not paraphrase, summarize, or invent requirement content.

This is a converter, not a reviewer. Quality assessment of the requirements belongs in [[tc-req-preprocess]], not here.

## Inputs and Outputs

**Input:**

- `INPUT.docx` — the requirements document.

**Outputs (written next to the input by default):**

- `INPUT.md` — the converted Markdown document.
- `INPUT.assets/` — extracted media (PNG, JPG, EMF, …), one file per image, stable filenames.
- `INPUT.md.report.md` — short conversion report: counts, low-confidence captions, blocks rendered as HTML, anything skipped.

Never edit `INPUT.docx`. Never write anything into the input directory other than the three artifacts above.

## Workflow

### 1. Open the package safely

- Open the `.docx` with shared-read access; the source may already be open in Word.
- Read directly from the OOXML package. Do not depend on Word COM or shell-out extraction.
- Locate at minimum:
  - `/word/document.xml`
  - `/word/_rels/document.xml.rels`
  - `/word/media/*`
  - `/word/numbering.xml` and `/word/styles.xml` when present (needed to map list types and custom heading styles).

### 2. Extract images first

- Copy every file in `/word/media/` into `INPUT.assets/`, keeping the original filename when unique, falling back to `image{index}{ext}` when names collide.
- Build a map `rId → assetPath` from `document.xml.rels`.
- For non-web formats (EMF/WMF) convert to PNG **only if** a converter is available; otherwise keep the original and flag it in the report.

### 3. Walk `w:body` in order

Iterate `w:body` children sequentially — each child becomes one Markdown block. Track block index so the report can point at problems precisely.

Classify each block as:

- heading (paragraph whose `pStyle` maps to a heading level)
- plain paragraph
- list item (paragraph with `numPr`)
- table
- paragraph or table cell containing an image

### 4. Convert blocks to Markdown

| DOCX block | Markdown form |
|---|---|
| Heading 1–6 | `#` … `######` |
| Plain paragraph | one paragraph with blank lines above/below |
| Bullet list | `-`, two-space indent per nesting level |
| Numbered list | `1.` per item; rely on auto-renumber, do not preserve the source number |
| Bold / italic / inline code | `**bold**`, `*italic*`, `` `code` `` |
| Hyperlink | `[text](url)` |
| Simple table | GFM pipe table |
| Complex table (merged cells, nested, contains images or lists) | HTML `<table>` block, kept verbatim |
| Image | see §5 |

Collapse runs of whitespace inside a paragraph to single spaces. Map `w:br` to a real paragraph break, except inside a table cell where it becomes `<br>`.

### 5. Render images with captions

For every image-bearing block, emit **two** Markdown elements in this exact order, with no blank line between them:

1. The image reference: `![{shortName}]({INPUT.assets/...})`
2. A blockquote caption:

   ```
   > **Image — {classification}:** {short structural description}
   ```

`classification` is one of:

- `UI prototype` — a screen, dialog, form, list view.
- `Flowchart` — a process, approval, or state diagram.
- `Other` — schematic, screenshot, decorative.

The structural description follows the image type:

- **UI prototype** — name the page or dialog, the top filter/tool area, the list or form body, and the primary action buttons. 2–4 short sentences.
- **Flowchart** — name the process, list the major nodes in order, call out branch conditions and end states. 2–4 short sentences.
- **Other** — one sentence on what the image is.

If the DOCX text around the image already describes it well, reuse that wording — do not invent new requirements. If neither the image nor the surrounding text gives enough to write a confident caption, write `> **Image — {classification} (low confidence):** {what you can say}` and record the block index in the report.

### 6. Keep image-text association tight

The single biggest failure mode is breaking the link between an image and its explanation. Rules:

- The image and its caption are adjacent — no blank line, no other block between them.
- Text that originally followed the image in the DOCX goes **after** the caption, unchanged.
- If an image lives inside a table cell, do not flatten the table. Render the table as HTML and put `<img>` plus the caption inside the cell.

### 7. Sanity-check before declaring done

Verify all of these — if any fails, fix it or log it in the report; do not silently drop content:

- `INPUT.assets/` file count ≥ image count in `/word/media/`.
- Every `![…](…)` in `INPUT.md` resolves to an existing asset file.
- Heading count in Markdown matches DOCX heading count (off-by-one is acceptable when the source has untitled sections).
- No raw OOXML tag (`<w:…`) leaked into the Markdown output.

### 8. Write the report

A short Markdown file with:

- Source path, output paths, timestamp.
- Counts: paragraphs, headings, tables, images, list items.
- A table of every image with: block index, asset path, classification, caption confidence (`ok` / `low`).
- Blocks rendered as HTML or skipped, with the reason.
- TODOs for the user: low-confidence captions, unconverted EMF/WMF files, complex tables kept as HTML, equations / SmartArt / text boxes that have no clean Markdown equivalent.

## Decision Rules

### When to render a table as HTML instead of GFM

Use HTML for the **entire** table when any of these hold:

- a cell spans multiple rows or columns
- a cell contains an image
- a cell contains a nested table
- a cell contains a list or multiple paragraphs

Otherwise use a GFM pipe table.

### When a caption is "low confidence"

Mark the caption low-confidence when:

- the image is a flowchart and you could not enumerate at least the entry node and one branch or end state
- the image is a UI prototype and you could not name the page and at least one control or field group
- the surrounding text contradicts what the image appears to show

### When to stop and ask

Stop and ask the user — do not guess — when:

- the `.docx` cannot be opened or appears corrupt
- `document.xml.rels` references images that are not in `/word/media/`
- the document relies on numbered lists or custom heading styles and `numbering.xml` or `styles.xml` is missing

## Common Mistakes

- Treating the `.docx` as flat text and losing image-paragraph order.
- Collecting all images first and dumping them at the end of the Markdown.
- Inlining base64 image data instead of writing files to `INPUT.assets/`.
- Rewriting requirement text into "cleaner" prose during conversion — this is a converter, not an editor.
- Inventing flowchart node names you did not actually see in the image.
- Flattening tables with merged cells or embedded images into mangled pipe tables.
- Silently dropping unsupported elements (text boxes, equations, SmartArt) — always log them in the report.

## Output Checklist

- [ ] `INPUT.md` written; no raw XML tags leaked.
- [ ] `INPUT.assets/` populated; every image referenced in the Markdown exists on disk.
- [ ] Every image immediately followed by a caption blockquote with classification and description.
- [ ] Tables: GFM when simple, HTML when complex; structure preserved either way.
- [ ] `INPUT.md.report.md` lists counts, low-confidence captions, and anything left unconverted.
- [ ] Source `.docx` untouched.
