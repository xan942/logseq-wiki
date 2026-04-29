# Cross-Device Sync via Nextcloud

Nextcloud is the recommended sync method because it handles file propagation, conflict detection,
and versioning without requiring manual git operations on every device.

**What Nextcloud is:** A self-hosted file sync and share platform — think a private Dropbox you
control. The desktop client watches a local folder and syncs changes to the server automatically.
If you're running it on NASPi, your server is already at `nextcloud.naspi.local`.

---

## Setup (Nextcloud already running)

### 1. Move the vault into Nextcloud

```bash
# Move the wiki repo into your Nextcloud sync folder
# The Nextcloud client watches this folder and syncs all changes automatically
mv ~/logseq-wiki ~/nextcloud/logseq-wiki    # adjust to your actual Nextcloud folder path

# Verify Git remote is still intact after the move
cd ~/nextcloud/logseq-wiki
git remote -v    # should still show origin → github.com/xan942/logseq-claude-memory
```

### 2. Update CLAUDE.md paths

Edit `CLAUDE.md` to reflect the new location:

```markdown
Wiki:    ~/nextcloud/logseq-wiki/wiki/
Schema:  ~/nextcloud/logseq-wiki/schema.md
Sources: ~/nextcloud/logseq-wiki/sources/
```

### 3. Install Nextcloud client on each device

Download the Nextcloud desktop client for your OS from https://nextcloud.com/install/#install-clients.
Point it at your NASPi Nextcloud server (`https://nextcloud.naspi.local`) and sync the
`logseq-wiki/` folder. The wiki will be available locally at `~/nextcloud/logseq-wiki/` on
every device — same path, same content.

### 4. Update CLAUDE.md on each device

Each device needs `CLAUDE.md` pointing at its local sync path. If all devices use the same
home directory structure (`~/nextcloud/logseq-wiki/`), no changes are needed — the paths are
already correct.

---

## Conflict Handling

Nextcloud detects write conflicts and creates `filename (conflicted copy YYYY-MM-DD).md` files
rather than silently overwriting — safer than Git's merge conflicts but still requires manual
resolution. Add a cron job to alert when conflicts appear:

```bash
# crontab -e — checks for Nextcloud conflict files every 15 minutes
# notify-send sends a desktop notification (requires a notification daemon like mako on Wayland)
*/15 * * * * find ~/nextcloud/logseq-wiki/wiki/ -name "*conflicted copy*" | grep -q . && \
    notify-send "Wiki conflict" "Resolve conflict files in ~/nextcloud/logseq-wiki/wiki/"
```

To resolve: compare the conflicted copy with the original, merge the content into one file,
delete the conflicted copy.

---

## How Git and Nextcloud Divide Responsibilities

With Nextcloud handling device sync, the two systems have distinct roles:

| System | Role |
|---|---|
| **Nextcloud** | Device propagation — file changes on one device appear on all others within seconds |
| **Git / GitHub** | Version history and audit trail — all of Claude's write-backs are committed here via MCP |

You do not need to run `git push` manually on each device. Claude's MCP write-backs go directly
to GitHub. Nextcloud then propagates those file changes to your other devices.

---

## Alternative: Git-only (no Nextcloud)

If you don't have Nextcloud, use a `systemd` user service with `inotifywait` to auto-commit
and push whenever a wiki file changes.

**`inotifywait`** is part of `inotify-tools` — a Linux utility that watches filesystem events
in real time. When a file is closed after writing (`close_write` event), the service triggers
a git commit and push.

```bash
# Install inotify-tools if not present
sudo apt install inotify-tools
```

Create the service file:

```bash
# ~/.config/systemd/user/wiki-sync.service
# systemd user services run as your user (no root needed) and start at login
```

```ini
[Unit]
Description=Wiki auto-sync on file change

[Service]
Type=simple
ExecStart=/bin/bash -c '
  inotifywait -m -r -e close_write ~/logseq-wiki/wiki/ --format "%%w%%f" |
  while read file; do
    cd ~/logseq-wiki
    git pull --rebase origin main 2>/dev/null    # pull first to reduce conflicts
    git add wiki/
    git commit -m "wiki: auto-sync $(date +%%Y-%%m-%%d\ %%H:%%M)" 2>/dev/null
    git push origin main 2>/dev/null
  done
'
Restart=on-failure

[Install]
WantedBy=default.target
```

```bash
# Enable and start the service — it will also restart automatically on login
systemctl --user enable --now wiki-sync.service

# Check it's running
systemctl --user status wiki-sync.service
```

**Limitation:** If two devices edit the same file within a few seconds of each other, Git will
produce a merge conflict on the next pull. Run `git pull --rebase origin main` before starting
a session on any device to minimize this. The `--rebase` flag replays your local commits on top
of remote changes, keeping history linear and conflicts rare.
