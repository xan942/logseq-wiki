# LLM Wiki — Claude Operating Instructions

## Vault Paths

> Edit these to match your actual paths before first use.

```
Wiki:    ~/logseq-wiki/wiki/
Schema:  ~/logseq-wiki/schema.md
Sources: ~/logseq-wiki/sources/
```

---

## Operations

### Ingest

Triggered by: `Ingest <source-file>`

1. Read `schema.md` — understand wiki structure and ingest rules
2. Read the source file
3. Identify which `wiki/` pages are affected (per schema ingest rules)
4. For each affected page: read the existing page (if it exists), then update with new information
   - Preserve existing accurate content
   - Integrate new information into the appropriate sections
   - Add or update the `Related:` field for cross-references
5. Update `wiki/index.md` — add new pages, update changed cross-refs
6. Commit all changed files via GitHub MCP using the schema commit convention

### Query

Triggered by: any question about the knowledge base

1. Read `wiki/index.md` to identify relevant pages
2. Read the relevant pages
3. Answer the question
4. If the answer reveals information worth persisting (a new fact, a resolved ambiguity,
   a synthesis not yet in the wiki), file a new page or update an existing one, then commit

### Lint

Triggered by: `Lint` or `Run lint`

1. Read `schema.md` — load lint rules
2. Read all files in `wiki/` recursively
3. Check each lint rule against each page
4. Output a lint report (format defined in schema.md)
5. Do NOT auto-fix errors or conflicts — flag them for human review
6. Commit the lint report as `wiki/lint-YYYY-MM-DD.md`

---

## Write-back

Use the GitHub MCP server to commit updates directly to the repository.

- Repository: `xan942/logseq-claude-memory`
- Branch: `main`
- Commit message format: see `schema.md` — Commit Convention section

Do not modify files outside `wiki/` unless explicitly asked.
Do not modify `schema.md` or `CLAUDE.md` unless explicitly asked.
Raw sources in `sources/` are immutable — never modify them.
