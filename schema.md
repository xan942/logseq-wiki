# Wiki Schema

This document defines the structure of the wiki, ingest rules, and lint rules.
Claude reads this on every Ingest and Lint operation.

---

## Wiki Structure

```
wiki/
├── index.md          # Auto-maintained table of contents + cross-reference map
├── hardware/         # Physical devices, specs, interfaces, connectivity
├── services/         # Running services, ports, config paths, health status
├── configs/          # Canonical configuration snippets (source of truth)
└── issues/           # Known problems, workarounds, resolution status
```

### Page Categories

| Category | What goes here | Example pages |
|---|---|---|
| `hardware/` | Physical device specs, interfaces, roles | `raspberry-pi-4.md`, `router.md` |
| `services/` | Service name, port, config path, dependencies, last-verified date | `nginx.md`, `nextcloud.md` |
| `configs/` | Canonical config blocks — the authoritative version | `nginx-ssl.md`, `pihole-custom-dns.md` |
| `issues/` | Problems with status: OPEN / RESOLVED / WONT-FIX | `ssl-cert-renewal.md` |

---

## Page Format

Every wiki page must have this frontmatter:

```markdown
# Page Title

**Last updated:** YYYY-MM-DD
**Summary:** One sentence describing what this page covers.
**Related:** [[page-name]], [[page-name]]

---

[body content]
```

Pages missing any of these fields should be flagged during Lint.

---

## Ingest Rules

When ingesting a new source:

| Source type | Pages to update |
|---|---|
| New hardware/device doc | Create or update `hardware/<device>.md`; cross-ref any affected `services/` pages |
| New service doc | Create or update `services/<service>.md`; update `configs/` if config is included |
| Config file or snippet | Create or update `configs/<name>.md`; update any `services/` pages that reference it |
| Troubleshooting notes | Create or update `issues/<name>.md` with status and resolution |
| General notes | Identify best-fit category; if unclear, place in `wiki/` root with a note for human review |

After every ingest:
1. Update `wiki/index.md` with any new pages or changed cross-references
2. Commit with message format: `wiki: ingest [source filename] → [affected pages]`

---

## Lint Rules

Run during the Lint operation. Flag (do not auto-fix unless marked safe):

| Rule | Severity | Auto-fix? |
|---|---|---|
| Page not linked from `index.md` | WARNING — orphan page | No — ask human |
| Page missing required frontmatter fields | WARNING | No |
| `services/` page missing "last-verified" date | INFO — stale after 30 days | No |
| Same port number assigned to two different services | ERROR — conflict | No |
| Contradictory information between two pages | ERROR — conflict | No |
| Issue page with status OPEN older than 90 days | INFO — stale | No |

Lint output format:
```
LINT REPORT — YYYY-MM-DD
[ERROR] configs/nginx-ssl.md ↔ services/nginx.md: port conflict (443 vs 8443)
[WARNING] hardware/old-router.md: orphan — not linked from index.md
[INFO] services/pihole.md: last-verified 2026-01-01 (>30 days)
```

---

## Commit Convention

All Claude write-backs use this format:
```
wiki: [operation] [short description]

Examples:
wiki: ingest raspberry-pi-hardware.md → hardware/raspberry-pi-4.md, services/
wiki: lint — 2 warnings, 1 error flagged
wiki: query filed new page wiki/services/vaultwarden.md
```
