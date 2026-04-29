# Nextcloud Sync Setup

Nextcloud is the sync layer for this wiki. It propagates file changes between your devices
and your self-hosted Nextcloud server without any manual git operations.

**What Nextcloud is:** A self-hosted file sync platform — like Dropbox but running on your
own hardware. The desktop client watches a local folder and syncs changes to the server in
real time. Logseq and Claude Code interact only with the local folder; they never talk to
Nextcloud directly.

---

## If Nextcloud is Already Running (NASPi setup)

Your Nextcloud server is at `nextcloud.naspi.local`. Install the desktop client on each
device and point it at your server.

### Install the desktop client

```bash
# Ubuntu / Debian
sudo apt install nextcloud-desktop

# Or download the AppImage from https://nextcloud.com/install/#install-clients
```

### Connect to your server

1. Open the Nextcloud desktop client
2. Click **Log in** → enter `https://nextcloud.naspi.local`
3. Authenticate with your Nextcloud credentials
4. Choose which folders to sync — include `logseq-wiki/`

The `~/Nextcloud/logseq-wiki/` folder now mirrors your server. Any file written here by
Claude Code or Logseq syncs automatically.

---

## Conflict Handling

Nextcloud detects write conflicts and creates a `filename (conflicted copy YYYY-MM-DD).md`
file rather than silently overwriting. This is safer than a silent merge but requires
manual resolution.

Add a cron job to alert when conflict files appear in the wiki:

```bash
# crontab -e
# Checks every 15 minutes — notify-send sends a desktop notification via mako (Wayland)
*/15 * * * * find ~/Nextcloud/logseq-wiki/wiki/ -name "*conflicted copy*" 2>/dev/null | \
    grep -q . && notify-send "logseq-wiki conflict" \
    "Resolve conflict files in ~/Nextcloud/logseq-wiki/wiki/"
```

To resolve a conflict:
1. Open both files — the original and the `(conflicted copy)` version
2. Compare and manually merge the content you want to keep
3. Save the original with the merged content
4. Delete the conflicted copy

---

## Adding a New Device

1. Install Nextcloud desktop client on the new device
2. Connect to `https://nextcloud.naspi.local`
3. Sync the `logseq-wiki/` folder
4. Clone or the folder already has the files — open it in Logseq as a graph
5. Open Claude Code in `~/Nextcloud/logseq-wiki/` — `CLAUDE.md` is already there

---

## Privacy Notes

- All wiki data stays on devices you control (local disk + your Nextcloud server)
- The Nextcloud server is `nextcloud.naspi.local` — on your LAN, not a public server
- SSL is handled by your step-ca internal CA (certificates at `/etc/ssl/step/`)
- No wiki content leaves your network unless you explicitly set up the optional GitHub backup
