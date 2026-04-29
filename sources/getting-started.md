# Getting Started Source

This is an example raw source document. Ingest it to see the LLM Wiki pattern in action.

## About This Wiki

This wiki is maintained by Claude Code using the LLM Wiki pattern described by Andrej Karpathy.
The wiki lives at `xan942/logseq-claude-memory` on GitHub.

## Example Hardware

**Device:** Raspberry Pi 4 (NASPi)
- Role: NAS + media server + home services host
- OS: Ubuntu 24.04 (aarch64)
- Storage: encrypted data partition at /mnt/data (184GB)

## Example Services

**Nextcloud** — self-hosted file sync and share
- Port: 443 (HTTPS via nginx reverse proxy)
- Data path: /mnt/data/nextcloud/
- URL: nextcloud.naspi.local

**Jellyfin** — open-source media server
- Port: 443 (HTTPS via nginx reverse proxy)
- URL: jellyfin.naspi.local

**Pi-hole v6** — DNS-based ad and tracker blocking
- IP: 192.168.1.223
- Port: 53 (DNS)

**Vaultwarden** — self-hosted Bitwarden-compatible password manager
- Port: 443 (HTTPS via nginx reverse proxy)
- URL: vaultwarden.naspi.local

## Notes

Replace this file with your own source documents. Sources in `sources/` are immutable —
once ingested, do not edit them. Add new source files for updated information.
