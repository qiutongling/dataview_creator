# Vault Adaptation

Use this reference when the user gives a vault, note, folder, or existing Obsidian context.

## Inspection Order

1. Ask the user to choose one mode on every invocation unless the prompt already explicitly chose it: generate a code block, insert into an existing note, or create a new view note.
2. Read the target note if direct insertion or revision is requested.
3. Inspect the destination folder if a new view note is requested.
4. Search the target folder for existing Dataview blocks and dashboard/index notes.
5. Inspect a small sample of nearby notes for frontmatter keys, inline fields, tags, folder names, date formats, and link style.
6. Reuse the vault's observed field names and labels.
7. Only ask additional questions when multiple plausible insertion points, note paths, or schemas remain.

Useful searches:

```powershell
rg -n "^---|^[A-Za-z0-9_-]+:|::|```dataview|```dataviewjs|#[\\p{L}\\w/-]+" "path\\to\\vault"
rg -n "status|type|due|created|updated|priority|tags|aliases|source" "path\\to\\vault"
```

For Chinese vaults, also search likely local field names:

```powershell
rg -n "状态|类型|截止|日期|优先级|来源|标签|别名|进度" "path\\to\\vault"
```

## Metadata Rules

Distinguish these metadata sources:

- YAML frontmatter: top-of-file fields between `---` fences.
- Inline fields: `Key:: Value` or inline bracket-style Dataview fields.
- File metadata: built-in fields such as `file.name`, `file.link`, `file.path`, `file.folder`, `file.tags`, `file.ctime`, and `file.mtime`.
- Task metadata: task-level fields such as task text, completion state, and any parsed task fields.

Do not assume a field exists just because it is common. Verify it from local notes when possible.

## Path and Link Conventions

- Match the vault's existing link style: wikilinks, Markdown links, encoded paths, aliases, and Chinese/English folder naming.
- Use quoted folder paths in DQL and `dv.pages('"Folder"')` in DataviewJS.
- Preserve real filenames and extensions when editing Markdown links if the vault already does so.
- Do not normalize folder names or move notes as part of a view request.

## Insertion Safety

Before editing a note, identify one exact insertion rule:

- Append under a named heading.
- Replace a specifically identified Dataview block.
- Insert at a marker comment.
- Append to the end of the note.

If no exact insertion rule exists, generate the block in chat instead of editing, or ask one short question.

After editing:

- Re-read the note or check the diff.
- Confirm the inserted block language is `dataview` or `dataviewjs`.
- Confirm no unrelated frontmatter, prose, links, or code blocks changed.

## New View Note Safety

Before creating a standalone view note, identify one exact destination:

- Vault root plus destination folder.
- Final note filename.
- Whether an existing same-name note should be left untouched, replaced, or handled by choosing a new name.

If the note path is not explicit, propose a path and wait for confirmation.

Default new-note structure:

```markdown
---
tags:
  - dataview
---

# View Title

Short purpose sentence.

```dataview
TABLE file.mtime AS "Updated"
FROM "Folder"
SORT file.mtime DESC
```
```

Adapt frontmatter and headings to nearby dashboard/index notes. Do not add frontmatter if local view notes do not use it.

After creating the note:

- Re-read the created file.
- Confirm the final path.
- Confirm whether the view uses DQL or DataviewJS and why.

## Field Naming with Chinese Notes

If notes use Chinese field names, keep them unless the user asks for English normalization.

Examples:

```dataview
TABLE 状态 AS "状态", 截止 AS "截止", 优先级 AS "优先级"
FROM "项目"
WHERE 状态 != "完成"
SORT 截止 ASC
```

```dataviewjs
const pages = dv.pages('"项目"')
  .where(p => p["状态"] !== "完成");

dv.table(
  ["项目", "截止"],
  pages.map(p => [p.file.link, p["截止"] ?? "-"])
);
```

Use bracket notation in DataviewJS for non-identifier field names, fields with spaces, and fields with punctuation.

## Failure Handling

- If a query depends on a missing field, state the missing field and offer a fallback using `file.*` metadata.
- If folder/tag scope is unclear, prefer a narrow explicit source over a global query.
- If DataviewJS is selected only because DQL is awkward, keep the script read-only and explain the selection briefly.
- If the local Dataview plugin version might not support a feature, flag that uncertainty and provide a simpler fallback.

## Sources

- Dataview metadata types: https://blacksmithgu.github.io/obsidian-dataview/annotation/types-of-metadata/
- Dataview query structure: https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
- Dataview JavaScript codeblock reference: https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/
