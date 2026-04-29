# Setup and Usage Guide

Full walkthrough for getting the LLM Wiki running with Claude Code write-back.

---

## Prerequisites

| Tool | What it is | Install |
|---|---|---|
| **Claude Code CLI** | Anthropic's terminal AI agent — runs Claude in your shell | `npm install -g @anthropic/claude-code` |
| **Node.js + npx** | JavaScript runtime; `npx` runs npm packages without installing them globally | https://nodejs.org |
| **Git** | Version control — tracks all wiki changes and syncs to GitHub | Pre-installed on most Linux/macOS |
| **GitHub account** | Hosts the repo; Claude writes back here via MCP | https://github.com |
| **GitHub personal access token** | Lets the MCP server authenticate to GitHub on Claude's behalf | GitHub → Settings → Developer settings → Personal access tokens → Fine-grained — needs `repo` scope |

---

## Step 1 — Clone and Configure Paths

```bash
# Clone your fork of this repo to ~/logseq-wiki (or any path you prefer)
git clone https://github.com/xan942/logseq-claude-memory ~/logseq-wiki
cd ~/logseq-wiki
```

Edit `CLAUDE.md` — the three vault paths at the top tell Claude where to read and write:

```markdown
Wiki:    ~/logseq-wiki/wiki/
Schema:  ~/logseq-wiki/schema.md
Sources: ~/logseq-wiki/sources/
```

Replace `~/logseq-wiki` with your actual clone path if different. Everything else in `CLAUDE.md`
is Claude's operating instructions — leave it unless you want to change behavior.

---

## Step 2 — Configure MCP Write-back

**What MCP is:** Model Context Protocol — a standard for giving Claude Code access to external
tools. The GitHub MCP server (`@modelcontextprotocol/server-github`) adds GitHub read/write
tools to Claude's toolkit, so it can commit files directly to your repo.

**Why this matters:** Without it, Claude can read the wiki but nothing it writes persists.
MCP write-back is what makes this an LLM Wiki instead of a context dump.

```bash
# Copy the pre-configured MCP settings into Claude Code's user config directory
# ~/.claude/settings.json is where Claude Code looks for MCP server definitions
cp setup/mcp-settings.json ~/.claude/settings.json
```

Store your GitHub token securely — **not in `~/.bashrc`** (visible in shell history):

```bash
# ~/.config/claude/secrets is sourced at login but not stored in history
mkdir -p ~/.config/claude
echo 'export GITHUB_TOKEN="ghp_your_token_here"' > ~/.config/claude/secrets
chmod 600 ~/.config/claude/secrets              # only your user can read it

# Source it at login — add to ~/.profile (runs on login shells, not just interactive ones)
echo 'source ~/.config/claude/secrets' >> ~/.profile
source ~/.profile                               # apply immediately without re-logging in
```

Edit `~/.claude/settings.json` and replace `<your-github-personal-access-token>` with the
literal token string (starts with `ghp_`). Do not use the env var name — paste the token value.

Verify MCP is wired up:

```bash
cd ~/logseq-wiki
claude          # opens Claude Code in this directory
```

Inside Claude Code, type:
```
/mcp
```
You should see `github` listed as a connected server with a green status indicator.

---

## Step 3 — Customize the Schema

Open `schema.md`. This is the config document that defines:
- **Wiki structure** — what folders exist and what each one holds
- **Ingest rules** — which pages to update when a given type of source comes in
- **Lint rules** — what counts as an error, warning, or stale entry
- **Page format** — required frontmatter fields every page must have
- **Commit convention** — how Claude labels its commits

The default schema uses four categories (`hardware/`, `services/`, `configs/`, `issues/`) suited
for a homelab/self-hosted setup. Rename them to match your use case — software projects, research
notes, a personal knowledge base. **The schema is the only file you need to edit to change how
Claude organizes knowledge.**

---

## Step 4 — First Ingest

`sources/` holds the raw input documents you curate. These are immutable — once added, never
edit them. If a source changes, add a new file and ingest again. This preserves the audit trail
of what Claude learned and when.

```bash
# Drop any document into sources/ — a README, config file, notes, troubleshooting log
cp ~/my-notes/naspi-hardware.md sources/naspi-hardware.md

# Commit the source to the repo before ingesting
# (so the GitHub history shows when the source was added)
git add sources/naspi-hardware.md
git commit -m "sources: add naspi hardware notes"
git push
```

Open Claude Code and run the Ingest operation:

```bash
cd ~/logseq-wiki
claude
```

```
Ingest sources/naspi-hardware.md
```

Claude will:
1. Read `schema.md` — loads wiki structure and ingest rules
2. Read `sources/naspi-hardware.md` — the raw source
3. Identify which `wiki/` pages are affected based on the schema ingest rules
4. Create or update those pages, preserving existing accurate content
5. Update `wiki/index.md` with new pages and cross-references
6. Commit all changes to GitHub via the GitHub MCP server

---

## Step 5 — Querying

Once the wiki has content, ask questions in natural language:

```
What services are running on the NASPi?
What port does Nextcloud use?
Is there an open issue with SSL certificates?
```

Claude reads `wiki/index.md` first (the table of contents) to identify which pages are relevant,
then reads those pages to answer. If answering reveals information not yet captured in the wiki
(a new fact, a resolved ambiguity, a useful synthesis), Claude files a new page and commits it —
the wiki grows from queries as well as ingests.

---

## Step 6 — Lint

Lint is a health check for the wiki. Run it periodically to catch drift before it compounds:

```
Lint
```

**What Claude checks** (defined in `schema.md` lint rules):
- Orphaned pages — exist in `wiki/` but not linked from `index.md`
- Missing frontmatter — pages without required `Last updated`, `Summary`, or `Related` fields
- Stale service pages — `services/` pages not verified in more than 30 days
- Port conflicts — same port number appearing in two different service pages
- Contradictions — factual conflicts between pages

Claude outputs a structured report and commits it as `wiki/lint-YYYY-MM-DD.md`. It does **not**
auto-fix errors or conflicts — those require human judgment. The committed reports give you a
history of wiki health over time.

---

## Step 7 — Cross-Device Sync

See `setup/nextcloud-sync.md` for the full walkthrough.

**Short version (Nextcloud — recommended if you're already running it):**
1. Move `~/logseq-wiki` into your Nextcloud folder (e.g., `~/nextcloud/logseq-wiki`)
2. Update the three vault paths in `CLAUDE.md`
3. Install the Nextcloud desktop client on each device — the wiki syncs automatically
4. Claude's write-backs go to GitHub via MCP (version history); Nextcloud handles device propagation

**Short version (Git-only):**
Use the `systemd` user service in `setup/nextcloud-sync.md` — it watches `wiki/` with
`inotifywait` (part of `inotify-tools`) and auto-commits + pushes on every file save.

---

## Troubleshooting

**Claude says it can't find the wiki**
The paths in `CLAUDE.md` must be exact. Run `ls ~/logseq-wiki/wiki/` to confirm the directory
exists. Tilde expansion works; relative paths do not.

**`/mcp` shows GitHub as disconnected**
Verify `~/.claude/settings.json` is valid JSON (`jq . ~/.claude/settings.json` will tell you).
Then verify the token is in your environment: `echo $GITHUB_TOKEN` should print the token, not
an empty line. If empty, `source ~/.config/claude/secrets` and try again.

**Merge conflicts after sync**
Git-only sync: run `git pull --rebase origin main` before starting any session on a new device.
`--rebase` replays your local commits on top of remote changes instead of creating a merge commit,
keeping history linear. Nextcloud sync: look for `(conflicted copy)` files in `wiki/` and resolve
manually — compare, merge content, delete the conflicted copy.

**Claude is modifying `sources/` or `schema.md`**
Re-state the constraint in your prompt: *"Do not modify files outside wiki/ unless I explicitly
ask."* The `CLAUDE.md` instructions say this, but an explicit prompt always takes precedence.

---

## Tips

- **Sources are immutable.** If a source document changes (e.g., you updated a config), add
  a new file (`sources/nginx-config-v2.md`) and ingest it. Don't overwrite the original — the
  old version tells the story of what Claude knew at each point in time.

- **Schema is the lever.** If Claude organizes things differently than you'd like, edit `schema.md`
  rather than individual wiki pages. The schema is Claude's operating instructions for the entire
  wiki structure — changing it affects all future ingests.

- **Lint after every 5–10 ingests.** Catches orphaned pages and stale entries before they pile up.

- **Queries surface gaps.** Questions Claude can't answer from the wiki reveal what's missing.
  Those gaps become your ingest backlog — add source documents that fill them.
