# logseq-wiki

A self-hosted, private knowledge base where Claude Code maintains the content and Logseq
provides the human interface — no cloud services, no data leaving your network.

Inspired by [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

---

## Architecture

```
You curate sources
       ↓
Claude Code — reads sources, writes wiki/ pages directly to disk
       ↓
~/Nextcloud/logseq-wiki/wiki/   ← local filesystem (stays on your device)
       ↓
Nextcloud desktop client — syncs to your self-hosted Nextcloud server
       ↓
All your devices — same wiki available everywhere via Nextcloud
       ↓
Logseq — visual interface for reading and editing the wiki on any device
```

**Nothing leaves your network.** Claude writes files locally. Nextcloud syncs them to your
own server. Logseq reads them on any device.

Optional: nightly Git push to a **private** GitHub repo for off-site backup (see `setup/github-backup.md`).

---

## The LLM Wiki Pattern

Instead of pasting context into every Claude session, the wiki compounds over time.
Claude actively maintains it — it doesn't just read from it.

### Three Layers

| Layer | What it is | Who maintains it |
|---|---|---|
| **Raw sources** | Immutable docs you curate in `sources/` | You |
| **Wiki** | Structured markdown pages in `wiki/` | Claude |
| **Schema** | Structure and rules defined in `schema.md` | You (initially) |

### Three Operations

| Operation | Trigger | What Claude does |
|---|---|---|
| **Ingest** | `Ingest sources/<file>` | Updates relevant wiki pages, updates index |
| **Query** | Any question | Reads wiki, answers, optionally files new pages |
| **Lint** | `Lint` | Flags contradictions, orphans, stale entries |

---

## Quickstart

### 1. Clone into your Nextcloud folder

```bash
# Clone directly into your Nextcloud sync directory so Nextcloud picks it up immediately
git clone https://github.com/xan942/logseq-wiki ~/Nextcloud/logseq-wiki
cd ~/Nextcloud/logseq-wiki
```

### 2. Open as a Logseq vault

In Logseq: **Add a graph** → point it at `~/Nextcloud/logseq-wiki/`

Logseq reads from `wiki/` (configured in `logseq/config.edn`) and the wiki pages appear
in the graph view. You can browse, edit, and add `[[links]]` like any Logseq vault.

### 3. First ingest

Open Claude Code in the vault directory:

```bash
cd ~/Nextcloud/logseq-wiki
claude
```

Then type:
```
Ingest sources/getting-started.md
```

Claude reads `schema.md`, writes new pages into `wiki/`, and updates `wiki/index.md`.
Nextcloud syncs the changes to your other devices automatically.

---

## Repository Structure

```
logseq-wiki/
├── README.md                  # This file
├── schema.md                  # Wiki structure, ingest rules, lint rules
├── CLAUDE.md                  # Claude's operating instructions
├── logseq/
│   └── config.edn             # Logseq vault configuration
├── setup/
│   ├── nextcloud-sync.md      # Nextcloud setup and conflict handling
│   └── github-backup.md       # Optional: private repo off-site backup
├── sources/                   # Raw source docs — immutable, you curate these
│   └── getting-started.md     # Example source to test your first Ingest
└── wiki/                      # LLM-maintained pages — Claude writes here
    └── index.md               # Auto-maintained table of contents
```

---

## Privacy Model

| Data | Where it lives |
|---|---|
| Wiki pages | Local disk + your Nextcloud server |
| Source documents | Local disk + your Nextcloud server |
| Schema and config | Local disk + your Nextcloud server |
| Claude conversations | Local only (Claude Code runs locally) |
| Optional backup | Private GitHub repo (your account, opt-in) |

No telemetry, no third-party sync services, no API keys required to run the wiki.
