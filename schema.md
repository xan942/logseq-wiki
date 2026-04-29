# Wiki Schema

Claude reads this file on every Ingest and Lint operation.
Edit this file to change how the wiki is structured and what rules Claude enforces.

---

## Wiki Structure

The `wiki/` directory is Logseq's pages directory (configured in `logseq/config.edn`).
Subdirectories appear as **namespaced pages** in Logseq's graph view.

```
wiki/
├── index.md          # Auto-maintained table of contents — Claude keeps this updated
├── hardware/         # Physical devices: specs, roles, interfaces, connectivity
├── services/         # Running services: ports, config paths, dependencies, health
├── configs/          # Canonical config snippets — the authoritative version of each config
└── issues/           # Known problems: status (OPEN / RESOLVED / WONT-FIX), workarounds
```

Adapt these categories to your use case. The schema is the only file you need to edit
to change how Claude organizes knowledge.

---

## Page Format

Every wiki page must include this frontmatter block at the top:

```markdown
# Page Title

**Last updated:** YYYY-MM-DD
**Summary:** One sentence describing what this page covers.
**Related:** [[wiki/category/page]], [[wiki/category/page]]

---

[body content]
```

Use `[[wiki/category/page]]` format for Logseq bidirectional links so they appear
correctly in the graph view.

Pages missing any required field should be flagged as a WARNING during Lint.

---

## Ingest Rules

| Source type | Pages to update |
|---|---|
| Hardware / device doc | Create or update `wiki/hardware/<device>.md`; cross-ref affected `wiki/services/` pages |
| Service doc | Create or update `wiki/services/<service>.md`; update `wiki/configs/` if config is included |
| Config file or snippet | Create or update `wiki/configs/<name>.md`; update `wiki/services/` pages that reference it |
| Troubleshooting notes | Create or update `wiki/issues/<name>.md` with status and resolution notes |
| General / uncategorised | Best-fit category; if unclear, create in `wiki/` root and add a `<!-- review needed -->` comment |

After every ingest:
1. Update `wiki/index.md` — add new pages, update cross-reference map
2. Confirm all new pages have the required frontmatter

---

## Lint Rules

| Rule | Severity | Auto-fix? |
|---|---|---|
| Page not linked from `wiki/index.md` | WARNING — orphan | No |
| Page missing required frontmatter fields | WARNING | No |
| `wiki/services/` page has no **Last updated** date | INFO — check if stale | No |
| Same port number assigned to two different services | ERROR — conflict | No |
| Contradictory facts between two pages | ERROR — conflict | No |
| `wiki/issues/` page status OPEN, last updated > 90 days ago | INFO — stale | No |

Lint report format:

```
LINT REPORT — YYYY-MM-DD
[ERROR]   wiki/services/nginx.md ↔ wiki/configs/nginx-ssl.md: port conflict (443 vs 8443)
[WARNING] wiki/hardware/old-device.md: orphan — not linked from index.md
[INFO]    wiki/services/pihole.md: last updated 2026-01-01, may be stale (>30 days)
```
