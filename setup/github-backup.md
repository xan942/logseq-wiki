# Optional: Private GitHub Backup

This sets up a nightly git push to a **private** GitHub repository as an off-site backup.
It is entirely optional — the wiki functions fully without it.

**What this is NOT:** This is not the write-back mechanism. Claude writes files directly
to your local filesystem. This is purely a backup of the wiki directory to GitHub.

---

## Create a Private Repo on GitHub

1. Go to https://github.com/new
2. Name it `logseq-wiki-backup` (or any name)
3. Set visibility to **Private**
4. Do not initialise with a README — it needs to be empty

---

## Set Up a Separate Git Remote for Backup

The main `origin` remote points at the public template repo (`xan942/logseq-wiki`).
Add a second remote for your private backup:

```bash
cd ~/Nextcloud/logseq-wiki

# Add the backup remote — use your GitHub username
git remote add backup https://github.com/YOUR_USERNAME/logseq-wiki-backup

# Verify both remotes
git remote -v
# origin   https://github.com/xan942/logseq-wiki (fetch)
# origin   https://github.com/xan942/logseq-wiki (push)
# backup   https://github.com/YOUR_USERNAME/logseq-wiki-backup (fetch)
# backup   https://github.com/YOUR_USERNAME/logseq-wiki-backup (push)
```

Store a GitHub personal access token (repo scope) for the backup push:

```bash
# Store the token — not in ~/.bashrc
mkdir -p ~/.config/logseq-wiki
echo 'ghp_your_token_here' > ~/.config/logseq-wiki/github-token
chmod 600 ~/.config/logseq-wiki/github-token
```

Configure git to use the token for the backup remote:

```bash
# Store credentials for the backup remote only
git config --local credential.helper \
    "store --file ~/.config/logseq-wiki/git-credentials"

# Write the credentials file
echo "https://YOUR_USERNAME:$(cat ~/.config/logseq-wiki/github-token)@github.com" \
    > ~/.config/logseq-wiki/git-credentials
chmod 600 ~/.config/logseq-wiki/git-credentials
```

---

## Automate Nightly Backup via Cron

```bash
# crontab -e
# Runs at 02:00 every night — commits any wiki changes and pushes to private backup repo
0 2 * * * cd ~/Nextcloud/logseq-wiki && \
    git add wiki/ && \
    git diff --cached --quiet || \
    git commit -m "backup: nightly $(date '+\%Y-\%m-\%d')" && \
    git push backup main >> ~/.local/share/logseq-wiki/backup.log 2>&1
```

The `git diff --cached --quiet || ...` check means the commit only runs if there are
actual changes — no empty commits on quiet nights.

Check the backup log:

```bash
cat ~/.local/share/logseq-wiki/backup.log
```

---

## What Gets Backed Up

Only `wiki/` pages are committed by the cron job. Source documents in `sources/` are
already versioned in the main repo. Schema and config files don't change often — commit
those manually when you update them.

---

## Privacy Reminder

Even in a private GitHub repo, your data is stored on GitHub's (Microsoft's) servers.
If your wiki contains sensitive personal information, consider whether this backup method
is appropriate for your threat model. Alternatives:
- Local backup only: `rsync -av ~/Nextcloud/logseq-wiki/ /mnt/data/backups/logseq-wiki/`
- Encrypted backup: use `restic` or `borgbackup` to a local or remote destination
