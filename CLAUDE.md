# logseq-wiki — Claude Operating Instructions

## Vault Paths

> Edit these to match your actual Nextcloud sync path before first use.

```
Vault:   ~/Nextcloud/logseq-wiki/
Wiki:    ~/Nextcloud/logseq-wiki/wiki/
Schema:  ~/Nextcloud/logseq-wiki/schema.md
Sources: ~/Nextcloud/logseq-wiki/sources/
```

---

## Operations

### Ingest

Triggered by: `Ingest <source-file>` or `Ingest sources/<file>`

1. Read `schema.md` — load wiki structure and ingest rules
2. Read the source file
3. Identify which `wiki/` pages are affected (per schema ingest rules)
4. For each affected page:
   - If the page exists: read it, then update with new information, preserving accurate content
   - If the page does not exist: create it using the page format defined in `schema.md`
   - Update the `Related:` field with relevant cross-references
5. Update `wiki/index.md` — add new pages, update cross-reference map
6. Write all changed files directly to disk

### Query

Triggered by: any question about the knowledge base

1. Read `wiki/index.md` to identify relevant pages
2. Read those pages
3. Answer the question
4. If the answer reveals something worth persisting (new fact, resolved ambiguity, useful
   synthesis not yet captured), write it as a new or updated wiki page

### Lint

Triggered by: `Lint` or `Run lint`

1. Read `schema.md` — load lint rules
2. Read all files under `wiki/` recursively
3. Check each lint rule against each page
4. Output a lint report using the format defined in `schema.md`
5. Do NOT auto-fix errors or conflicts — flag them for human review
6. Write the report to `wiki/lint-YYYY-MM-DD.md`

---

## Write-back

Write files directly to the local filesystem using your built-in file tools.
No external APIs, services, or network calls are needed or expected.

Nextcloud syncs the vault to all devices automatically after files are written.
Logseq reflects changes the next time it syncs or the vault is refreshed.

---

## Boundaries

- Only modify files inside `wiki/` unless explicitly asked to do otherwise
- Never modify `schema.md`, `CLAUDE.md`, or `logseq/config.edn` unless explicitly asked
- Files in `sources/` are immutable — never edit them; ingest them, don't modify them
- Do not create files outside the vault path defined above
