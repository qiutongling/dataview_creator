# Dataview View Builder

Dataview View Builder is a Codex skill for creating Obsidian Dataview views. It helps an assistant generate DQL or DataviewJS views, insert them into existing notes, or create standalone Dataview dashboard notes.

## What It Does

- Builds `dataview` and `dataviewjs` code blocks for Obsidian notes.
- Chooses DQL for simple `TABLE`, `LIST`, `TASK`, and `CALENDAR` views.
- Chooses DataviewJS for complex grouping, derived columns, task aggregation, multiple sections, or reusable `dv.view()` workflows.
- Inspects vault conventions when paths are provided, so generated views can reuse existing fields, folders, tags, links, and heading style.
- Uses a safety-first workflow for editing notes and creating new dashboard notes.

## Skill Layout

```text
dataview-view-builder/
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── dataviewjs-patterns.md
    ├── dql-patterns.md
    └── vault-adaptation.md
```

## Installation

Copy or link the skill folder into your Codex skills directory.

On Windows, the default user skills directory is usually:

```powershell
C:\Users\<you>\.codex\skills
```

Example:

```powershell
Copy-Item -Recurse .\dataview-view-builder C:\Users\<you>\.codex\skills\
```

After installation, invoke it explicitly with:

```text
Use $dataview-view-builder to create a Dataview or DataviewJS view for my Obsidian notes.
```

## Workflow Modes

The skill asks which output mode to use each time it is invoked, unless the prompt already specifies one.

1. Code block mode: generate a fenced `dataview` or `dataviewjs` block for manual copying.
2. Insert existing note mode: insert or replace a Dataview block in a specified existing note.
3. New view note mode: create a standalone dashboard/index/view note in a specified vault location.

For edit modes, the skill requires an explicit target path or confirmation before changing files.

## Examples

Generate a project task view:

```text
Use $dataview-view-builder to create a task view for unfinished tasks in my Projects folder.
```

Insert a dashboard into an existing note:

```text
Use $dataview-view-builder to insert a DataviewJS project dashboard into D:\note\obsidian_note\myvault\Dashboard.md.
```

Create a standalone view note:

```text
Use $dataview-view-builder to create a new Dataview note at D:\note\obsidian_note\myvault\Views\Active Projects.md.
```

## Validation

Validate the skill structure with the Codex skill validator:

```powershell
python "C:\Users\qiuto\.codex\skills\.system\skill-creator\scripts\quick_validate.py" "D:\code\dataview_creator\dataview-view-builder"
```

Expected result:

```text
Skill is valid!
```

## References

The skill references the official Obsidian Dataview documentation for syntax and API behavior:

- Dataview query structure: https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
- Dataview query types: https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/
- DataviewJS codeblock API: https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/
