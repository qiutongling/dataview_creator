# DataviewJS Patterns

Use this reference when DQL would be too limited or hard to maintain.

## Safety Boundary

Default to read-only DataviewJS. Do not write files, delete files, rename files, call network APIs, or mutate the vault from generated JavaScript unless the user explicitly requests it and confirms the risk.

## Basic Page Query

Use `dv.pages(source)` with the same source syntax as DQL. Folder sources need quotes inside the string.

```dataviewjs
const pages = dv.pages('"Projects"')
  .where(p => p.status !== "done")
  .sort(p => p.due ?? p.file.mtime, "asc");

dv.table(
  ["Project", "Status", "Due"],
  pages.map(p => [p.file.link, p.status ?? "-", p.due ?? "-"])
);
```

Use `dv.pages()` only for deliberate global dashboards. Prefer a folder, tag, or link source.

## Derived Columns

Use DataviewJS when columns need fallback logic, formatting, or branching.

```dataviewjs
const pages = dv.pages("#book")
  .where(p => p.rating || p.status);

dv.table(
  ["Book", "Score", "State"],
  pages.map(p => [
    p.file.link,
    p.rating ? `${p.rating}/10` : "unrated",
    p.status ?? "unknown"
  ])
);
```

Keep derived expressions short. If a helper function improves readability, define it above the query.

## Grouped Sections

Use DataviewJS for one heading and one table/list per group.

```dataviewjs
const groups = dv.pages('"Notes"')
  .where(p => p.type)
  .groupBy(p => p.type);

for (const group of groups) {
  dv.header(3, group.key);
  dv.table(
    ["Note", "Updated"],
    group.rows
      .sort(p => p.file.mtime, "desc")
      .map(p => [p.file.link, p.file.mtime])
  );
}
```

Use DQL `GROUP BY` instead when one grouped table is enough.

## Task Aggregation

Use `dv.taskList()` for task rendering. Pass `false` as the second argument to avoid grouping by source file.

```dataviewjs
const tasks = dv.pages('"Projects"')
  .file.tasks
  .where(t => !t.completed && t.text.includes("#next"));

dv.taskList(tasks, false);
```

Use custom sections when the user asks for "overdue", "today", and "later" buckets.

```dataviewjs
const tasks = dv.pages('"Projects"').file.tasks.where(t => !t.completed);
const today = dv.date("today");

const overdue = tasks.where(t => t.due && t.due < today);
const dueToday = tasks.where(t => t.due && t.due.equals(today));

dv.header(3, "Overdue");
dv.taskList(overdue, false);
dv.header(3, "Today");
dv.taskList(dueToday, false);
```

Confirm task fields before relying on them. Some vaults store due dates in text instead of task metadata.

## Multiple Views in One Block

Use DataviewJS when one dashboard needs several independent sections.

```dataviewjs
const source = '"Projects"';
const pages = dv.pages(source);

dv.header(3, "Active Projects");
dv.table(
  ["Project", "Due"],
  pages.where(p => p.status === "active").map(p => [p.file.link, p.due ?? "-"])
);

dv.header(3, "Recently Updated");
dv.list(
  pages.sort(p => p.file.mtime, "desc").limit(10).file.link
);
```

## Reusable `dv.view()`

Use `dv.view(path, input)` only when the user wants reusable custom view code or many notes should share the same rendering logic.

```dataviewjs
await dv.view("views/project-dashboard", {
  source: '"Projects"',
  status: "active",
  limit: 20
});
```

Important constraints:

- The view path is resolved from the vault root.
- `dv.view()` is asynchronous, so use `await`.
- Do not place view scripts in dot-prefixed directories such as `.views`; Dataview cannot read them.
- A folder view may contain `view.js` and optional `view.css`.

## Output Rules

- Return fenced `dataviewjs` blocks.
- Use `const` and small helper functions.
- Prefer `??` for display fallbacks.
- Avoid excessive abstraction inside a note-level code block.
- Add a short comment only when it explains a non-obvious choice.

## Sources

- Dataview JavaScript codeblock reference: https://blacksmithgu.github.io/obsidian-dataview/api/code-reference/
- Dataview query structure and source syntax: https://blacksmithgu.github.io/obsidian-dataview/queries/structure/
