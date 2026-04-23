---
name: dataview-view-builder
description: "Create, revise, insert, or create standalone Obsidian Dataview view notes. Use when the user asks for a Dataview, DQL, DataviewJS, TABLE, LIST, TASK, CALENDAR, dashboard, index, project/task view, note query, reusable dv.view, or dedicated Dataview dashboard note for Obsidian; supports three explicit modes: generate a code block, insert into an existing note, or create a new view note."
---

# Dataview View Builder

## Overview

Build Obsidian Dataview views that fit the user's vault conventions. Prefer concise DQL for ordinary views, and choose DataviewJS when the requested output needs multi-step logic, derived fields, complex grouping, or custom rendering.

## Workflow

1. Ask for the output mode every time this skill is invoked, before building or editing anything.
   - Code block mode: return a fenced `dataview` or `dataviewjs` code block plus any short setup notes.
   - Insert existing note mode: edit an existing note, only after the user provides a target vault/note path or an unambiguous current-file context.
   - New view note mode: create a standalone Dataview dashboard/index/view note, only after the user provides the vault path, destination folder, and note name or accepts a proposed note path.
   - If the user's prompt already explicitly chooses one of these modes, restate the selected mode briefly and continue; otherwise ask a short question offering exactly these three modes.

2. Inspect local context before writing.
   - In code block mode, inspect vault context when the user provides it; otherwise state any field/path assumptions in the answer.
   - In insert existing note mode, read the target note first.
   - In new view note mode, inspect the destination folder and nearby dashboard/index notes before choosing frontmatter, heading style, and source scope.
   - Search nearby notes for matching frontmatter keys, inline fields, tags, folders, existing Dataview blocks, and naming conventions.
   - Reuse confirmed field names instead of inventing schema.

3. Choose DQL or DataviewJS.
   - Use DQL for standard `TABLE`, `LIST`, `TASK`, and `CALENDAR` views with ordinary `FROM`, `WHERE`, `SORT`, `GROUP BY`, `FLATTEN`, or `LIMIT`.
   - Use DataviewJS for multi-step computation, nested arrays, dynamic columns, conditional rendering, multiple sections/tables, cross-field derived values, complex task aggregation, or `dv.view()` reuse.

4. Build the view.
   - Prefer a bounded `FROM` source such as a folder, tag, or link when the user intent allows it.
   - Preserve Obsidian link style and path conventions already present in the vault.
   - Keep generated code readable enough that the user can modify it later.

5. Validate manually.
   - Check code block language, command order, field spelling, quotes around folder sources, and whether date/number/string assumptions match observed metadata.
   - For inserted or newly created notes, re-read the edited/created note or check the diff before reporting completion.

## Selection Rules

Use DQL when the view is a straightforward query:

- `TABLE` with fixed columns.
- `LIST` with zero or one display expression.
- `TASK` filtered by completion, tag, due date, folder, or project status.
- `CALENDAR` from one date field.
- Simple filtering, sorting, limiting, grouping, or flattening.

Use DataviewJS when DQL would become hard to read or cannot express the result cleanly:

- Compute derived columns with branching or fallback values.
- Group rows and render a heading/table per group.
- Combine several queries into one dashboard.
- Filter or map nested arrays beyond simple `FLATTEN`.
- Render custom task lists with non-default grouping.
- Call `dv.view()` for reusable view scripts.

Security default: do not generate DataviewJS that writes, deletes, renames, rewrites, or networks from the vault unless the user explicitly asks for that behavior and confirms the risk. Prefer read-only use of the Dataview API.

## Direct Edit and Note Creation Rules

- Never replace an existing Dataview block unless the user identifies that block or asks for replacement.
- Insert under the requested heading, marker, or cursor-equivalent location when known.
- If multiple insertion points match, stop and ask briefly.
- Preserve frontmatter, surrounding prose, and unrelated code blocks.
- If the target note uses Chinese headings, aliases, or field names, match that style rather than translating by default.
- For new view notes, do not create a note until the vault path and final note path are explicit.
- For new view notes, use a minimal structure: optional frontmatter matching local conventions, one title, one short purpose sentence when useful, then the Dataview code block.

## References

Load only the reference needed for the current request:

- `references/dql-patterns.md`: DQL syntax, command order, and common `TABLE`/`LIST`/`TASK`/`CALENDAR` patterns.
- `references/dataviewjs-patterns.md`: DataviewJS API patterns, complex grouping, task aggregation, and `dv.view()` reuse.
- `references/vault-adaptation.md`: how to inspect a vault, discover fields safely, and edit notes without breaking local conventions.

Primary upstream sources for syntax are the official Dataview documentation:

- https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
- https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/
- https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/
