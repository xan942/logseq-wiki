# Setup and Usage Guide

Complete walkthrough for the Logseq + Nextcloud + Claude Code stack.
Everything runs on your own hardware — no cloud services required.

---

## Prerequisites

| Tool | What it is | Install |
|---|---|---|
| **Claude Code CLI** | Anthropic's terminal AI agent — runs locally, writes files to disk | `npm install -g @anthropic/claude-code` |
| **Node.js** | JavaScript runtime required by Claude Code | https://nodejs.org |
| **Logseq** | Local-first note-taking app — reads and displays your wiki markdown files | https://logseq.com/downloads |
| **Nextcloud desktop client** | Syncs your Nextcloud folder between devices and your self-hosted server | https://nextcloud.com/install/#install-clients |
| **Git** | Version control — used for cloning this template and optional backup | Pre-installed on most Linux/macOS |

---

## Step 1 — Clone into your Nextcloud folder

Clone directly into your Nextcloud sync directory. The Nextcloud client watches this
folder, so anything written here — by you, Logseq, or Claude — syncs to your server
and to all other devices automatically.

```bash
# Replace ~/Nextcloud with your actual Nextcloud sync folder path
# Common paths: ~/Nextcloud, ~/nextcloud, ~/NextCloud
git clone https://github.com/xan942/logseq-wiki ~/Nextcloud/logseq-wiki
cd ~/Nextcloud/logseq-wiki
```

Update the vault paths in `CLAUDE.md` to match your path:

```
Vault:   ~/Nextcloud/logseq-wiki/
Wiki:    ~/Nextcloud/logseq-wiki/wiki/
Schema:  ~/Nextcloud/logseq-wiki/schema.md
Sources: ~/Nextcloud/logseq-wiki/sources/
```

---

## Step 2 — Open as a Logseq vault

**What this does:** Logseq reads all markdown files in the vault directory and builds a
graph of pages and their connections. The `logseq/config.edn` file (already in this repo)
tells Logseq to use `wiki/` as its pages directory and prefer markdown format.

1. Open Logseq
2. Click **Add a graph** (or **Open a local directory**)
3. Navigate to `~/Nextcloud/logseq-wiki/` and select it
4. Logseq opens the vault — `wiki/index.md` appears as the Index page

You can now browse and edit wiki pages in Logseq normally. Pages Claude creates in `wiki/`
appear in the graph automatically.

---

## Step 3 — Set up Nextcloud sync (if not already running)

If your Nextcloud server is already running and the desktop client is installed, skip this.
If not, see `setup/nextcloud-sync.md` for the full setup guide.

**Quick check:** If you cloned into `~/Nextcloud/logseq-wiki/` and you can see the files
in your Nextcloud web UI at `nextcloud.naspi.local`, sync is working.

---

## Step 4 — Customize the schema

Open `schema.md` and adapt it to your use case before your first ingest.

The default categories (`hardware/`, `services/`, `configs/`, `issues/`) suit a homelab
setup. If you want different categories — software projects, research topics, personal
knowledge — rename them here. The schema is the single lever that controls how Claude
organizes everything.

---

## Step 5 — First ingest

`sources/` holds your raw input documents. These are the inputs you curate — config files,
notes, READMEs, troubleshooting logs. Once added, they are **never edited** (add new files
for updated information instead). This keeps a clean audit trail of what Claude learned and when.

```bash
cd ~/Nextcloud/logseq-wiki
claude   # opens Claude Code with CLAUDE.md loaded automatically
```

Type in Claude Code:
```
Ingest sources/getting-started.md
```

Claude will:
1. Read `schema.md` — understand the wiki structure and ingest rules
2. Read the source file
3. Create wiki pages in the appropriate `wiki/` subdirectories
4. Update `wiki/index.md` with new pages and cross-references
5. Write all files directly to disk

Logseq will reflect the new pages on next sync or manual refresh (`Ctrl+Shift+R` in Logseq).

---

## Step 6 — Querying

With content in the wiki, ask questions in natural language inside Claude Code:

```
What services are running and what ports do they use?
Is there an open issue with the SSL certificates?
What is the hardware spec for the NASPi?
```

Claude reads `wiki/index.md` first to find relevant pages, then reads those pages to answer.
If the answer surfaces something worth keeping — a fact not yet recorded, a useful synthesis —
Claude writes a new or updated page to the wiki automatically.

---

## Step 7 — Lint

Run periodically to keep the wiki healthy:

```
Lint
```

Claude scans all pages in `wiki/` against the rules in `schema.md` and writes a report to
`wiki/lint-YYYY-MM-DD.md`. It flags:
- Orphaned pages (exist in `wiki/` but not linked from `index.md`)
- Pages missing required frontmatter fields
- Port conflicts between service pages
- Contradictions between pages
- Stale entries (services not verified in >30 days, open issues >90 days old)

Claude does **not** auto-fix errors or conflicts — those require your judgment.

---

## Step 8 — Optional: private GitHub backup

The wiki lives on your Nextcloud server. If you want an off-site backup to a private
GitHub repo, see `setup/github-backup.md`. This is entirely optional and adds no
dependencies to the daily workflow.

---

## Troubleshooting

**Logseq doesn't show Claude's new pages**
Press `Ctrl+Shift+R` in Logseq to re-index the vault. If pages still don't appear,
check that the files were actually written: `ls ~/Nextcloud/logseq-wiki/wiki/`.

**Logseq links show as unresolved**
Claude uses `[[wiki/category/page]]` link format. Logseq resolves these as namespaced
pages. If a link appears broken, check that the target file exists at the expected path.

**Nextcloud is not syncing changes**
Check the Nextcloud desktop client — it should show a green checkmark. If it shows
errors, check your server connection (`nextcloud.naspi.local` reachable?) and that
the client is watching the correct folder.

**Claude can't find the wiki**
Verify the paths in `CLAUDE.md` exactly match your filesystem. Run
`ls ~/Nextcloud/logseq-wiki/wiki/` to confirm the directory exists. Tilde (`~`)
expansion works; relative paths do not.

**Two devices edited the same page simultaneously**
Nextcloud creates a `(conflicted copy YYYY-MM-DD)` file rather than overwriting. The
cron job in `setup/nextcloud-sync.md` alerts you when this happens. Open both files,
merge the content manually, delete the conflicted copy.

---

## Tips

- **Schema first.** Before your first real ingest, spend 10 minutes in `schema.md` making
  sure the categories and ingest rules match your actual use case. Changing the schema
  later is fine but requires re-organizing existing wiki pages.

- **Sources are the audit trail.** Every file in `sources/` represents a point in time.
  Claude's wiki pages are derived from these. If you wonder "where did Claude learn X?"
  the source file is the answer.

- **Lint after every 5–10 ingests.** Orphaned pages and stale entries compound quickly.

- **Use Logseq for human edits.** You can edit wiki pages directly in Logseq. Claude
  will respect existing content during future ingests (it preserves accurate content).
  Just keep the frontmatter format intact.

- **Queries grow the wiki.** Every time Claude can't fully answer a question from existing
  wiki pages, that's a signal to add a source. The question backlog is your ingest backlog.
