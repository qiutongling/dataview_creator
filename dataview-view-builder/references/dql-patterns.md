# DQL Patterns

Use this reference when the user's request can be expressed as ordinary Dataview Query Language.

## Command Order

A DQL block starts with exactly one query type. `FROM`, when present, must appear immediately after the query type. Other data commands can then be chained in the order needed by the result.

```dataview
<QUERY-TYPE> <fields>
FROM <source>
WHERE <condition>
SORT <field> ASC
GROUP BY <field>
FLATTEN <array> AS item
LIMIT 20
```

Prefer this practical order unless a specific query needs a different execution order:

1. Query type: `TABLE`, `LIST`, `TASK`, or `CALENDAR`.
2. `FROM`: folder, tag, link, incoming/outgoing link source, or source expression.
3. `WHERE`: field existence and filtering.
4. `FLATTEN`: only when an array/list needs row-level handling.
5. `GROUP BY`: only when grouped output is desired.
6. `SORT`: before or after grouping depending on whether rows or groups should be sorted.
7. `LIMIT`: usually last, unless the user explicitly wants "top N before sorting again".

Source: Dataview "Structure of a Query" and "Data Commands" official documentation.

## Safe Source Defaults

Avoid broad full-vault queries unless the user explicitly wants a global dashboard. Prefer one of these bounded forms:

```dataview
FROM "Projects"
FROM #project/active
FROM [[Some MOC]]
FROM outgoing([[Some Dashboard]])
FROM #tag AND "Folder"
FROM #tag AND -"Archive"
```

Folder sources must be quoted. Tags are unquoted. Links use Obsidian wikilink syntax.

## TABLE

Use `TABLE` when the user wants columns, dashboards, indexes, review sheets, or metadata comparison.

```dataview
TABLE status AS "Status", due AS "Due", priority AS "Priority"
FROM "Projects"
WHERE status != "done"
SORT due ASC
LIMIT 20
```

Useful variants:

```dataview
TABLE WITHOUT ID file.link AS "Note", summary AS "Summary", file.mtime AS "Updated"
FROM "wiki"
WHERE summary
SORT file.mtime DESC
```

```dataview
TABLE rows.file.link AS "Notes"
FROM #source
GROUP BY type
SORT type ASC
```

## LIST

Use `LIST` for simple note lists. DQL supports at most one additional display expression after `LIST`.

```dataview
LIST
FROM #idea
SORT file.mtime DESC
LIMIT 15
```

```dataview
LIST "Updated: " + string(file.mtime)
FROM "Notes"
WHERE contains(file.tags, "#review")
SORT file.mtime DESC
```

If the list needs multiple computed display fields, switch to `TABLE` or DataviewJS.

## TASK

Use `TASK` for interactive task results. Task queries operate at task level, not page level.

```dataview
TASK
FROM "Projects"
WHERE !completed
SORT due ASC
GROUP BY file.link
```

Common filters:

```dataview
TASK
FROM #project/active
WHERE !completed AND due <= date(today) + dur(7 days)
SORT due ASC
```

```dataview
TASK
WHERE !completed AND contains(text, "#next")
GROUP BY file.link
```

If the request needs custom task grouping, multiple task sections, or task-derived rollups, use DataviewJS.

## CALENDAR

Use `CALENDAR` when each page has one relevant date field.

```dataview
CALENDAR due
FROM "Projects"
WHERE due
```

```dataview
CALENDAR file.mtime
FROM "Journal"
```

If the calendar source date is derived from multiple candidate fields, use DataviewJS or ask the user which field should win.

## Metadata Checks

Use existence checks before comparing optional fields:

```dataview
WHERE due AND due <= date(today)
```

Use observed field types. Date comparisons require Dataview-recognized dates, normally ISO-like values such as `YYYY-MM-DD`.

## Sources

- Dataview query structure: https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
- Dataview query types: https://blacksmithgu.github.io/obsidian-dataview/queries/query-types/
- Dataview data commands: https://blacksmithgu.github.io/obsidian-dataview/queries/data-commands/
