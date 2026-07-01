---
name: tc-req-preprocess
description: 在审查包含嵌入式 UI 原型、流程图图像或审批流图表的 `.docx` 需求文档时使用，特别是在 Codex 必须评估需求质量、确定文本是否充分描述了每个图像、补充缺失或薄弱的解释，并交付修改后的 `.docx` 副本和书面审查报告时。
---

# Reviewing DOCX Requirements

## Overview

Review the `.docx` as a package, not as a normal text file. Extract structure from `word/document.xml`, map embedded images through `word/_rels/document.xml.rels`, classify each image as a UI prototype or a process-flow image, judge whether the accompanying text is adequate for that image type, then write changes into copied outputs rather than mutating the original.

## Workflow

### 1. Build the artifact list

- Find the main `.docx` requirements file and any existing review notes.
- Treat `~$` temporary Office files as noise.
- Decide the output set up front:
  - review report `.md`
  - revised `.docx` copy
  - optional second-pass review report or repaired revision

### 2. Read the `.docx` safely

- Prefer shared-read access because the document may already be open in Word.
- Read the OOXML package directly with `System.IO.Packaging.Package` or equivalent.
- Start with:
  - `/word/document.xml`
  - `/word/_rels/document.xml.rels`
  - `/word/media/*`
- Do not assume Word COM is available or usable.
- Do not rely on archive extraction if policy or file locks may block it.

### 3. Reconstruct document structure

- Walk `w:body` in order.
- Distinguish:
  - headings/paragraphs
  - tables
  - paragraphs or cells that contain images
- Build a lightweight sequence view such as:
  - section heading
  - table index
  - image target
  - nearby text after the image

This is the key step. Most embedded prototypes sit inside table cells, not standalone paragraphs.

### 4. Map images to their business context

- Resolve image relationship IDs from `document.xml.rels`.
- For each image, record:
  - module or section name
  - page/dialog purpose
  - image type: UI prototype, flowchart/process diagram, or mixed
  - the text immediately after the image
  - whether that text is business-rule-only, UI-structure-only, or actually explains the image

When needed, export specific images to a temp folder and inspect them directly.

### 5. Review the requirements in two passes

#### Pass A: Document quality

Check for issues such as:

- scope says one thing but the body only covers a subset
- empty or placeholder function tables
- numbering drift
- contradictory status or approval rules
- typos that change meaning
- missing shared state definitions

Write these into a review report, not into the skill output itself.

#### Pass B: Image-text adequacy

Use this stricter rule:

- `Has text` is not enough.
- The text must let a reader understand the image without opening it.

For UI prototypes, adequate text should cover most of:

1. page or dialog name
2. query/filter area
3. list fields or form fields
4. primary action buttons
5. important state switches or approval actions
6. key constraints or visibility rules

For flowcharts or approval-flow diagrams, adequate text should cover most of:

1. process name or diagram purpose
2. start condition or trigger
3. major nodes or stages
4. branch conditions or approval/rejection paths
5. actors, owners, or systems involved
6. end states, outputs, or archival outcomes

Classify each image as:

- `complete`: text can independently describe the image
- `partial`: text exists but only captures business rules or fragments
- `missing`: no meaningful text exists

### 6. Write supplements into copied `.docx` files

- Never edit the source file directly.
- Create a named copy first.
- Append supplements directly after the image-bearing paragraph or cell content.
- Keep existing business-rule text.
- Add structural text in a consistent style, for example:
  - `Prototype supplement: this image shows ...`
  - query area
  - list/form area
  - buttons/actions
  - purpose
- For flowcharts, use a different pattern:
  - process or flow name
  - trigger or entry condition
  - main nodes in order
  - branch rules
  - final outcomes

If doing a second repair pass, create another copy from the previous revised file.

### 7. Re-review the revised version

After inserting supplements, run a second review focused on:

- whether every target image got a supplement
- whether supplements were inserted in the right location
- wording errors introduced during insertion
- uneven detail level across modules
- generic names that should be module-specific
- whether flowchart images were incorrectly described as UI pages

Typical repair items:

- duplicated wording such as `model-series-series`
- vague labels such as `agreement-like content`
- places where the supplement still does not mention visible controls

### 8. Deliverables

Produce separate artifacts:

- review findings report
- revised `.docx`
- optional second-pass review report
- optional repaired `.docx`

Keep the report explicit about:

- what standard was used
- which modules were complete, partial, or missing
- what was changed
- what still needs manual review

## Decision Rules

### When is text insufficient?

Treat text as insufficient when it only says things like:

- users can only see published items
- after unpublishing it becomes invisible
- supports add/edit/delete

That describes business outcome, not the image itself.

### When is text sufficient?

Treat text as sufficient when it captures both:

- the visible structure of the image
- the major business interaction or process expressed by that image

### When to inspect the image directly

Inspect the image directly when:

- the surrounding text is ambiguous
- multiple images exist in one module
- the page is a dialog with controls not mentioned in text
- the image is a flowchart and the text does not enumerate nodes or branches
- the supplement must mention exact visible fields or actions

## Writing Pattern for Supplements

Use short structural prose. Avoid long requirement dumps.

For UI prototypes, a good supplement normally contains 3-5 sentences:

1. identify the page or dialog
2. describe the top query/filter/tool area
3. describe the list or form body
4. describe action buttons or approval controls
5. state the page purpose if not obvious

For flowcharts, a good supplement normally contains 3-5 sentences:

1. identify the flow or approval process
2. state the entry trigger
3. describe the major nodes or steps in order
4. describe branch conditions or approval decisions
5. state the end states or resulting actions

## Common Mistakes

- Treating any text after an image as sufficient.
- Inspecting only paragraph-level images and missing images inside table cells.
- Treating every image as a UI page when some are process diagrams.
- Editing the original `.docx` instead of a copy.
- Using Word COM as the default path.
- Writing supplements that repeat business rules but still do not describe visible controls.
- Writing flowchart supplements that mention only "supports approval" without naming nodes, branches, or end states.
- Failing to run a second-pass review after insertion.

## Output Checklist

- Source `.docx` preserved
- Revised `.docx` copy created
- Review report created
- Image adequacy judged with the stricter standard for both UI prototypes and flowcharts
- Supplements inserted only where needed
- Re-review performed on the revised version
