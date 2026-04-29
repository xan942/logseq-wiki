# LLM Wiki — Claude Code Memory Pattern

A personal knowledge base where Claude Code acts as the **maintainer**, not just a reader.

Inspired by [Andrej Karpathy's LLM Wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

---

## The Problem with Most AI Memory Setups

Most approaches treat AI memory as a static context dump — you paste in notes, Claude reads them,
nothing persists. Knowledge has to be re-derived every session. The maintenance burden grows faster
than the value, so wikis get abandoned.

## The LLM Wiki Pattern

Instead of retrieving raw documents at query time, the LLM **incrementally builds and maintains**
a structured, interlinked wiki. Knowledge compounds over time. When you add a source, Claude
integrates it into existing pages, updates cross-references, and flags contradictions.

### Three Layers

| Layer | What it is | Who maintains it |
|---|---|---|
| **Raw sources** | Immutable original docs you curate | You |
| **Wiki** | Structured markdown pages, interlinked | Claude |
| **Schema** | Config defining structure and rules | You (initially), Claude (on lint) |

### Three Operations

| Operation | Trigger | What Claude does |
|---|---|---|
| **Ingest** | You add a source file to `sources/` | Updates relevant wiki pages, updates cross-refs, commits back |
| **Query** | You ask a question | Reads wiki, answers, optionally files a new page |
| **Lint** | Periodic or on-demand | Flags contradictions, orphans, stale entries |

**The key difference from RAG or a context dump: Claude writes back to the wiki.** It doesn't just read.

---

## Quickstart (15 minutes)

### 1. Clone and set vault path

```bash
git clone https://github.com/xan942/logseq-claude-memory ~/logseq-wiki
cd ~/logseq-wiki
```

Edit `CLAUDE.md` — replace the vault path with your actual path:
```markdown
Wiki:    ~/logseq-wiki/wiki/
Schema:  ~/logseq-wiki/schema.md
Sources: ~/logseq-wiki/sources/
```

### 2. Configure MCP write-back

```bash
cp setup/mcp-settings.json ~/.claude/settings.json
```

Store your GitHub token securely (not in `~/.bashrc`):
```bash
mkdir -p ~/.config/claude
echo 'export GITHUB_TOKEN="ghp_your_token_here"' > ~/.config/claude/secrets
chmod 600 ~/.config/claude/secrets
echo 'source ~/.config/claude/secrets' >> ~/.profile
source ~/.profile
```

Edit `~/.claude/settings.json` and replace `<your-github-personal-access-token>` with your token.

### 3. First ingest

Open Claude Code in the wiki directory:

```bash
cd ~/logseq-wiki
claude
```

Then type:
```
Ingest sources/getting-started.md
```

Claude reads `schema.md`, creates or updates relevant `wiki/` pages, and commits back via GitHub MCP.

### 4. Cross-device sync

See `setup/nextcloud-sync.md` if you run Nextcloud (recommended). Otherwise see `guide.md`.

---

## Repository Structure

```
logseq-claude-memory/
├── README.md               # This file
├── schema.md               # Wiki structure, ingest rules, lint rules — edit this first
├── CLAUDE.md               # Claude's operating instructions — set your vault paths here
├── guide.md                # Full setup and usage guide
├── setup/
│   ├── mcp-settings.json   # Ready-to-use .claude/settings.json with GitHub MCP
│   └── nextcloud-sync.md   # Cross-device sync via Nextcloud (recommended)
├── sources/                # Raw source documents — immutable, you curate these
└── wiki/                   # LLM-maintained markdown — Claude writes here
    └── index.md            # Auto-maintained table of contents
```

---

## Why This Works

Humans abandon wikis because maintenance is tedious. LLMs excel at that tedium —
updating references, maintaining consistency, synthesizing across pages. You focus on
curation and asking meaningful questions. Claude handles the bookkeeping.

This echoes Vannevar Bush's 1945 Memex: a personal, curated knowledge store where
the connections between items matter as much as the items themselves.
