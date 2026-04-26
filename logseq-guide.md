# Complete Implementation Guide: Logseq as Claude Code's Central Memory Store

This guide provides step-by-step instructions for implementing a centralized Logseq vault that serves as Claude Code's persistent memory across multiple devices and sessions.

**Estimated Time:** 6-8 weeks for complete implementation
**Getting Working System:** 2-3 weeks (Phase 1-2)
**Difficulty:** Intermediate (requires comfort with CLI, Git, and Markdown)

---

# Table of Contents

- [Phase 1: Foundation (Weeks 1-2)](#phase-1-foundation-weeks-1-2)
- [Phase 2: Memory System (Weeks 2-3)](#phase-2-memory-system-weeks-2-3)
- [Phase 3: Build Documentation (Weeks 3-4)](#phase-3-build-documentation-weeks-3-4)
- [Phase 4: Synchronization (Week 4)](#phase-4-synchronization-week-4)
- [Phase 5: Claude Code Integration (Weeks 4-5)](#phase-5-claude-code-integration-weeks-4-5)
- [Phase 6: Knowledge Base Building (Weeks 5-8)](#phase-6-knowledge-base-building-weeks-5-8)
- [Phase 7: Monitoring & Maintenance (Ongoing)](#phase-7-monitoring--maintenance-ongoing)
- [Troubleshooting](#troubleshooting)
- [Common Workflows](#common-workflows)

---

## Phase 1: Foundation (Weeks 1-2)

### Overview

Phase 1 establishes your core Logseq vault with Git synchronization. At the end of this phase, you'll have a working system that can be synced across devices.

### Step 1.1: Install Logseq

**Why:** Logseq provides the outliner and knowledge base functionality for storing all your specifications.

**What it does:** Creates a local-first note-taking system with graph visualization and query capabilities.

**How it fits:** Foundation of everything—all Claude Code context comes from Logseq.

#### Installation

**macOS:**
```bash
brew install logseq
```

**Linux (Ubuntu/Fedora):**
```bash
flatpak install flathub com.logseq.Logseq
```

**Windows:**
Download from https://logseq.com/downloads

**All platforms (alternative):**
- Visit https://logseq.com/downloads
- Download appropriate version
- Follow installer instructions

#### Verify Installation

```bash
# If installed via CLI, verify
logseq --version

# Or just open Logseq from your applications menu
```

### Step 1.2: Create & Initialize Vault

**Why:** Your vault is where all specifications, configurations, and generated code will live. Git initialization enables multi-device sync.

**What it does:** Creates directory structure and sets up version control.

**Context:** This single vault will be the source of truth for all projects across all devices.

#### Create Vault Directory

```bash
# Create vault
mkdir -p ~/logseq/homelab-vault
cd ~/logseq/homelab-vault

# Create subdirectories for organization
mkdir -p pages journals archives
```

#### Initialize Git Repository

```bash
# Initialize Git
git init

# Configure your identity
git config user.email "your.email@example.com"
git config user.name "Your Name"

# Optionally, set as global config (used for all repos)
git config --global user.email "your.email@example.com"
git config --global user.name "Your Name"
```

#### Create .gitignore

```bash
cat > .gitignore << 'EOF'
# Logseq system files - don't sync these
logseq/bak/
logseq/version-files/
.DS_Store
*.swp
*~

# Node modules (if using plugins)
node_modules/

# Environment files
.env
.env.local

# IDE files
.vscode/
.idea/

# Avoid committing individual config changes
# (config.edn stays local to each device)
!logseq/config.edn
EOF
```

**Why .gitignore:** Prevents sync of system files, backups, and device-specific configs that would cause conflicts.

#### Create Initial Files

```bash
# Create README
cat > README.md << 'EOF'
# Homelab Vault (Logseq)

Central memory store for:
- All homelab builds (ZTB, NASPi, etc.)
- Claude Code configurations and standards
- Device inventory and network topology
- Generated scripts and configurations
- Build logs and issue tracking
- Personal context and long-term goals

## Quick Start
1. Open Logseq
2. Choose "Open a local directory"
3. Select ~/logseq/homelab-vault
4. Start with [[Home]]

## Syncing
```bash
# Before starting work
git pull origin main

# After making changes
git add -A && git commit -m "Description"
git push origin main
```

## Claude Code Integration
See [[Setup/How-To-Use-Claude-Code]] for usage patterns.
EOF

# Create initial commit
git add -A
git commit -m "Initial Logseq vault setup"
```

**What this does:** Establishes Git history and README documentation for your vault.

### Step 1.3: Create GitHub Repository

**Why:** GitHub hosts your vault for multi-device access and backup. Private repo keeps your sensitive information safe.

**What it does:** Creates remote repository for syncing vault across devices.

**Context:** This is your backup and multi-device sync hub.

#### Create Repository on GitHub

1. Go to https://github.com/new
2. Repository name: `homelab-vault` (or name of your choice)
3. **IMPORTANT:** Choose "Private" (keep sensitive data private)
4. **DO NOT** initialize with README/gitignore/license (you already have them)
5. Click "Create repository"

#### Connect Local Repository to GitHub

```bash
# Add remote (replace YOUR_USERNAME with actual username)
git remote add origin https://github.com/YOUR_USERNAME/homelab-vault.git

# Rename branch to main (if needed)
git branch -M main

# Push to GitHub
git push -u origin main
```

**Verify:**
- Go to your GitHub repo in browser
- Should see your vault contents (README.md, .gitignore, etc.)

### Step 1.4: Configure Logseq Settings

**Why:** Logseq configuration optimizes it for Claude Code integration with proper task tracking, querying, and metadata.

**What it does:** Customizes Logseq behavior for your workflow.

**Context:** These settings ensure Claude Code can read specs effectively and track metadata.

#### Open Logseq and Configure Vault

1. Open Logseq application
2. Click "Open a local directory"
3. Select `~/logseq/homelab-vault`
4. Allow access when prompted

#### Create `logseq/config.edn`

Logseq creates this file automatically, but edit it for Claude Code optimization:

```bash
cat > logseq/config.edn << 'EOF'
{:max-indent-level 8
 :preferred-format :markdown
 :preferred-todo-keyword "TODO"
 :other-preferred-keywords ["DOING" "DONE" "BLOCKED" "LATER" "PENDING"]
 
 ;; Enable features for Claude Code integration
 :feature/enable-block-timestamps? true
 :feature/enable-timetracking? false
 :graph/enabled? true
 :graph/show-all-pages? true
 :graph/pages-section? true
 :graph/show-orphaned-pages? true
 
 ;; Query support (critical for reporting)
 :query/enabled? true
 :query/page-ref-default-collapsed-depth 2
 
 ;; Journal settings
 :journal/page-title-format "yyyy-MM-dd"
 :journal/file-name-format "yyyy_MM_dd"
 :journal/page-title-short? false
 
 ;; Auto-sync to prevent conflicts
 :ui/auto-sync-enabled? true
 
 ;; Properties for Claude Code tracking
 :block-properties-order ["type" "status" "priority" "deadline" "created" "created-from" 
                          "depends-on" "deployed-to" "tested-on" "author" "version"]
 
 ;; Whiteboard/canvas support (optional)
 :feature/enable-whiteboards? true}
EOF
```

**Why this config:**
- `block-properties-order` enables metadata tracking for scripts
- `query/enabled? true` allows Logseq queries (Claude Code reads these)
- `feature/enable-block-timestamps?` tracks when specs/scripts created
- `auto-sync-enabled?` prevents divergence between git commits

#### Verify Configuration

1. In Logseq, go to Settings (gear icon)
2. Check "Show advanced settings"
3. Verify settings match config.edn
4. Close settings

### Step 1.5: Create Core Architecture Pages

**Why:** These foundational pages establish the structure for everything else.

**What they do:** Create basic pages that Claude Code will reference.

**Context:** These are the "brain" pages that Claude Code reads to understand how to act.

#### Create Home Page

In Logseq, create a new page called "Home":

```markdown
# Homelab Central Intelligence

**Last Updated:** [[2026-04-23]]

This is the central knowledge repository for all homelab projects, built as Claude Code's persistent memory store.

## Quick Navigation

### 📁 Core System
- [[Claude-Config/Agent-Personality]] - How Claude should act
- [[Claude-Config/Coding-Standards]] - Code generation standards
- [[Memory/Personal-Context]] - About you

### 📊 Builds (Active Projects)
- [[ZTB (Zero Trust Bridge)]] - Main build (status: IN PLANNING)
- [[NASPi Services]] - Storage and media (status: ACTIVE)

### 🧠 Memory & Context
- [[Memory/Personal-Context]] - Your expertise and preferences
- [[Memory/Device-Inventory]] - All network devices
- [[Memory/Service-Locations]] - URLs, IPs, access points

### 📋 Generated Content
- [[Generated-Scripts]] - All Claude Code outputs
- [[Issues-Log]] - Problems and solutions
- [[Daily-Logs]] - Build progress journal

## Today's Quick Tasks
[Will fill in as you work]

## System Status
[Status dashboard]
```

#### Create Claude-Config Pages

Create folder/pages structure in Logseq (you can use "/" in page names to create hierarchy):

1. **Claude-Config/Agent-Personality.md**

```markdown
# Claude Code Agent Configuration: Personality & Behavior

**Type:** CONFIG
**Version:** 1.0
**Applies-To:** All Claude Code invocations

## Core Identity

You are the **Build Automation Engineer** for the Homelab project.

**Role:**
- Generate production-ready code and configurations
- Test implementations thoroughly
- Maintain consistency across all builds
- Document all work
- Coordinate with external tools (GitHub, Asana, Slack)

## Decision-Making Framework

WHEN generating code::
- Read Logseq spec FIRST, never assume
- Test before marking complete
- Generate documentation automatically
- Create Asana task for deployment
- Notify via Slack

WHEN encountering ambiguity::
- Ask clarifying questions
- Reference specific Logseq page causing confusion
- Create [[Issues-Log]] entry if needed
- Proceed conservatively

## Output Standards

All code:
- Bash: #!/bin/bash, set -euo pipefail, logging
- Config: YAML format with comments
- Documentation: Every script gets Logseq page

All changes:
- Tested before marking done
- Linked to source specification
- Updated [[Issues-Log]] if problems
- Notified via Slack when ready
```

2. **Claude-Config/Coding-Standards.md**

```markdown
# Coding Standards

**Type:** CONFIG
**Version:** 1.0
**Applies-To:** All generated [[Generated-Scripts]]

## Bash Scripts

### Shebang & Error Handling
```bash
#!/bin/bash
set -euo pipefail
```

### Logging Function
```bash
log() {
    local level="$1"
    shift
    local message="$@"
    local timestamp=$(date +'%Y-%m-%d %H:%M:%S')
    echo "[${timestamp}] [${level}] ${message}" | tee -a "${LOG_FILE}"
}
```

### Configuration
```bash
SCRIPT_NAME="$(basename "$0")"
LOG_FILE="${LOG_FILE:-/var/log/${SCRIPT_NAME}.log}"
DRY_RUN="${DRY_RUN:-false}"
```

## Configuration Files

- Format: YAML (not TOML/INI)
- Comments: Explain each section
- Examples: Show how to use
- Version: Include at top

## Documentation

- Markdown format
- Clear structure
- Link to specs
- Include examples
```

3. **Claude-Config/How-To-Use-Claude-Code.md**

```markdown
# How to Use Claude Code with This Vault

**Type:** GUIDE

## Basic Pattern

```bash
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

Read specifications:
- [[Page/Name/For/Spec]]
- [[Claude-Config/Coding-Standards]]
- [[Memory/Context/Page]]

Generate:
- What to create
- What to test
- How to test

Document:
- Where to save results
- Update original spec page
- Notify via Slack
"
```

## Key Rules

1. Always read Logseq first
2. Test before marking done
3. Create documentation automatically
4. Link everything back to source spec
5. Create Asana task for manual steps

## Common Mistakes

❌ "Generate health check script" (missing spec)
✅ "Read [[Layer-2-Tailscale-Inner]], generate health check script"
```

#### Create Memory Pages

Create basic templates:

1. **Memory/Personal-Context.md**

```markdown
# Personal Context & Expertise

**Type:** MEMORY
**Last-Updated:** [[2026-04-23]]

## About You

**Experience:** Intermediate Linux admin, Advanced networking
**Expertise:** Bash, Docker, Networking, Security
**Learning Style:** Deep understanding preferred
**Goals:** Build ZTB (Pi5 secure router), Migrate to Headscale (future)

## Your Infrastructure

- [[NASPi]] - Ubuntu 24.04, Tailscale, Pi-hole
- [[Arch Linux ROG]] - Development system
- [[Windows PC]] - Client machine
- [[iPhone]] - Mobile access

## Communication Preferences

- Slack: Alerts
- Asana: Tasks
- GitHub: Code
- Logseq: Everything
```

2. **Memory/Device-Inventory.md**

```markdown
# Device Inventory

**Type:** MEMORY
**Last-Updated:** [[2026-04-23]]

## Active Devices

| Device | OS | LAN IP | Tailscale IP | Status |
|--------|----|----|----|----|
| NASPi | Ubuntu | 192.168.1.223 | 100.68.47.14 | ACTIVE |
| Arch ROG | Arch | DHCP | TBD | ACTIVE |
| Windows | Windows 11 | DHCP | TBD | ACTIVE |
| iPhone | iOS | WiFi | TBD | ACTIVE |

## Planned Devices

- ZTB (Pi5 Router): OpenWRT, 192.168.1.1, TBD

## Device Capabilities

| Device | Docker | Systemd | SSH | Can-Test |
|--------|--------|---------|-----|----------|
| NASPi | Yes | Yes | Yes | Yes |
| Arch | Yes | Yes | Yes | Yes |
| Windows | WSL | No | Yes | Limited |
| iPhone | No | No | No | No |
```

3. **Memory/Service-Locations.md**

```markdown
# Service Locations & Access

**Type:** MEMORY
**Last-Updated:** [[2026-04-23]]

## NASPi Services

### Pi-hole
- **URL:** http://100.68.47.14/admin (Tailscale)
- **Status:** ACTIVE
- **Port:** 53, 80

### Nextcloud
- **URL:** http://100.68.47.14 (Tailscale)
- **Status:** ACTIVE

### Jellyfin
- **URL:** http://100.68.47.14:8920 (Tailscale)
- **Status:** ACTIVE

## External Services

- Tailscale Admin: https://login.tailscale.com/admin
- GitHub: https://github.com/YOUR_USERNAME/homelab-vault
- Asana: [Your project link]
```

#### Commit Changes

```bash
cd ~/logseq/homelab-vault

# Stage and commit
git add -A
git commit -m "Phase 1: Created core architecture pages"

# Push to GitHub
git push origin main
```

**Verify:**
- Go to GitHub, see new pages in repo
- Check Logseq interface shows all pages
- Test graph view (should show relationships)

---

## Phase 2: Memory System (Weeks 2-3)

### Overview

Phase 2 populates your vault with context pages about yourself, your devices, your infrastructure, and your long-term goals. This is the "memory" that Claude Code reads to understand your environment.

### Step 2.1: Expand Personal Context

**Why:** Claude Code reads this to understand your expertise, preferences, and goals.

**What it does:** Documents who you are, what you know, how you like to work.

**Context:** Ensures Claude Code's output matches your style and expertise level.

Expand `Memory/Personal-Context.md`:

```markdown
# Personal Context & Expertise

**Type:** MEMORY
**Created:** 2026-04-23
**Last-Updated:** 2026-04-23
**Refresh-Interval:** Monthly

## About You

**Experience Level:** Intermediate Linux system admin, Advanced networking knowledge
**Learning Style:** Deep understanding preferred, not just instructions
**Expertise Areas:**
- Linux system administration
- Networking (TCP/IP, routing, firewalls, VPNs)
- Cybersecurity concepts
- Docker and containerization
- Self-hosted infrastructure
- Network monitoring

**Programming Languages:**
- Bash scripting: Expert
- Python: Intermediate
- YAML: Expert (configuration)
- JSON: Expert

**Preferred Tools:**
- OS: Linux (Ubuntu, Arch), OpenWRT
- Editors: VS Code, Vim
- Version Control: Git, GitHub
- Infrastructure: Docker, systemd, iptables
- Monitoring: Prometheus, Grafana
- Logging: syslog-ng

## Your Philosophy

**Privacy First:** Believe in self-hosted infrastructure over cloud services
**Security Layered:** Multiple overlapping security controls (defense-in-depth)
**Learning Focused:** Want to understand the "why" not just "how"
**Documentation:** Prefer comprehensive documentation
**Open Source:** Prefer open-source over proprietary tools
**Control:** Want complete visibility and control over infrastructure

## Learning Preferences

When Claude Code generates things, prefer:
- Detailed inline comments explaining each part
- Links to official documentation
- Rationale for design decisions
- Common pitfalls and how to avoid them

## Your Goals (6-month timeframe)

**Primary Goal:** Build and deploy [[ZTB (Zero Trust Bridge)]]
- Enterprise-grade secure router
- Double encryption (Tailscale + Mullvad)
- Network-wide threat detection
- Complete observability

**Secondary Goal:** Migrate to self-hosted Headscale
- Replace Tailscale coordination server with own instance
- Complete self-hosting for overlay network

**Tertiary Goal:** Complete network segmentation and advanced threat detection
- Guest network isolation
- IoT VLAN segmentation
- Advanced firewall rules

## Communication Preferences

**For Claude Code:**
- Reference Logseq pages directly
- Ask clarifying questions if ambiguity
- Provide both specification and standards
- Include relevant memory/context pages

**For notifications:**
- Slack: Immediate alerts
- Asana: Task tracking
- GitHub: Code storage
- Logseq: All specifications and documentation

## What NOT to do

DON'T:
- Assume specifications (always ask)
- Deploy without testing
- Skip documentation
- Assume device capability (check [[Device-Inventory]])

DO:
- Read Logseq first
- Test locally
- Document everything
- Link related pages
- Update status pages
```

### Step 2.2: Expand Device Inventory

**Why:** Claude Code reads this to understand which devices can run what.

**What it does:** Documents all your network devices, their capabilities, and their status.

**Context:** Claude Code uses this to generate device-appropriate scripts.

Expand `Memory/Device-Inventory.md`:

```markdown
# Device Inventory

**Type:** MEMORY
**Created:** 2026-04-23
**Last-Updated:** 2026-04-23
**Next-Review:** 2026-04-30

## Network Map Overview

```
                    ISP
                     |
          ISP Router (Bridge Mode)
                     |
        ____________|____________
       |            |            |
      NASPi     Arch Linux      iPhone
    (100.68)   (100.68.x.x)   (100.68.x.x)

[After ZTB deployment]
                    ISP
                     |
          ISP Router (Bridge Mode)
                     |
          ZTB (Pi5 Router) ← NEW
          192.168.1.1
            __|____|__
           |  |  |  |
          Layer1-5 + WiFi 6E
```

## Current Devices [ACTIVE]

### NASPi
- **Type:** NAS / Services
- **Hardware:** Raspberry Pi (model: [TBD])
- **OS:** Ubuntu 24.04 LTS Server
- **LAN-IP:** 192.168.1.223
- **Tailscale-IP:** 100.68.47.14
- **Services:** 
  - [[Pi-hole]] (DNS filtering) [ACTIVE]
  - [[Nextcloud]] (File storage) [ACTIVE]
  - [[Jellyfin]] (Media streaming) [ACTIVE]
- **Status:** ACTIVE
- **Uptime:** 99.9%
- **Disk Space:** [Check periodically]
- **CPU/RAM:** [Monitor performance]
- **SSH Access:** pi@192.168.1.223 or pi@100.68.47.14
- **Notes:** Primary services host

### Arch Linux ROG
- **Type:** Desktop / Development
- **Hardware:** ASUS ROG (GPU: [TBD], RAM: [TBD])
- **OS:** Arch Linux (rolling release)
- **Tailscale-IP:** 100.68.x.x (TBD)
- **Services:** Development environment
- **Status:** ACTIVE
- **Role:** Development, testing, compilation, script testing
- **SSH Access:** [username]@[IP]
- **Notes:** Used for testing before deployment

### Windows PC
- **Type:** Client
- **Hardware:** [Details TBD]
- **OS:** Windows 11
- **Tailscale-IP:** 100.68.x.x (TBD)
- **Services:** Tailscale VPN client, Windows services
- **Status:** ACTIVE
- **Role:** General use, Tailscale exit node testing
- **SSH Access:** WinRM or SSH (if configured)
- **Notes:** Daily driver for non-Linux work

### iPhone
- **Type:** Mobile
- **OS:** iOS 17+
- **Tailscale-IP:** 100.68.x.x (TBD)
- **Services:** Tailscale app only
- **Status:** ACTIVE
- **Role:** Mobile access to network services
- **Notes:** Can't deploy to iPhone, used for access only

## Planned Devices [INCOMING]

### ZTB (Zero Trust Bridge)
- **Type:** Router
- **Hardware:** Raspberry Pi 5 4GB
- **OS:** OpenWRT SNAPSHOT
- **LAN-IP:** 192.168.1.1 (will be assigned)
- **Tailscale-IP:** TBD (after setup)
- **Services:** 
  - Layer 1: Pi-hole (DNS Privacy)
  - Layer 2: Tailscale (Inner Encryption)
  - Layer 3: Mullvad (Outer Encryption)
  - Layer 4: Firewall (Threat Detection)
  - Layer 5: Monitoring (Grafana, Prometheus)
- **Status:** PLANNED (arrival date: 2026-04-24)
- **Role:** Central gateway, double encryption, threat detection, monitoring
- **SSH Access:** ubuntu@192.168.1.1 or ubuntu@100.64.x.x
- **Related:** [[ZTB (Zero Trust Bridge)]]

## Device Capabilities Matrix

| Device | Docker | Systemd | GPIO | Network | SSH | Can-Test | Can-Deploy |
|--------|--------|---------|------|---------|-----|----------|------------|
| ZTB | Yes | Yes | Yes | Ethernet + WiFi | Yes | Yes | Yes (primary) |
| NASPi | Yes | Yes | No | Ethernet + WiFi | Yes | Yes | Yes (secondary) |
| Arch Linux | Yes | Yes | No | Ethernet/WiFi | Yes | Yes | Yes |
| Windows PC | Yes (WSL) | No | No | Ethernet/WiFi | Limited | Limited | No |
| iPhone | No | No | No | WiFi only | No | No | No |

## Access Credentials Location

⚠️ **Note:** Actual passwords/tokens NOT stored in Logseq vault
⚠️ **Store in:** Password manager (1Password, Bitwarden, etc.)

| Service | Storage Location | Notes |
|---------|------------------|-------|
| NASPi Pi-hole admin | Password manager | Entry: "NASPi Pi-hole Admin" |
| Nextcloud | Password manager | Entry: "NASPi Nextcloud" |
| Jellyfin | Password manager | Entry: "NASPi Jellyfin" |
| Tailscale auth | Password manager | Entry: "Tailscale API Token" |
| GitHub PAT | Password manager | Entry: "GitHub Personal Access Token" |
| Asana API | Password manager | Entry: "Asana API Token" |
| Slack Bot Token | Password manager | Entry: "Slack Bot Token" |

## Deployment Checklist

When deploying scripts to a device:
- [ ] Check device is online and reachable
- [ ] Verify SSH access working
- [ ] Check available disk space (df -h)
- [ ] Verify network connectivity (ping 8.8.8.8)
- [ ] Review device capabilities above
- [ ] Backup important files before major changes
- [ ] Have rollback procedure ready

## Performance Baseline (for monitoring)

Record baseline performance metrics:

**NASPi:**
- CPU idle: ~10%
- RAM free: ~800MB
- Disk used: ~20%

**ZTB (when deployed):**
- CPU idle: ~5%
- RAM free: ~3.5GB
- Disk used: ~10%

## Planned Upgrades

- [ ] NASPi: Add external storage
- [ ] ZTB: Monitor CPU during peak traffic
- [ ] All: Update OS packages monthly
- [ ] All: Review security patches

## Status Summary

- **Total Devices:** 4 active, 1 planned
- **Total Uptime:** 99%+
- **Last Verified:** 2026-04-23
- **Next Verification:** 2026-04-30
```

### Step 2.3: Create Service Locations Page

**Why:** Claude Code references this when generating deployment scripts and monitoring configs.

**What it does:** Documents where all services are located and how to access them.

**Context:** Used when Claude Code needs to reference service URLs, ports, or access methods.

Expand `Memory/Service-Locations.md`:

```markdown
# Service Locations & Access Points

**Type:** MEMORY
**Created:** 2026-04-23
**Last-Updated:** 2026-04-23
**Sensitivity:** Internal only (don't share public)

## Connectivity Status

Last checked: 2026-04-23

- [✅] NASPi Pi-hole: Accessible via 100.68.47.14:53
- [✅] NASPi Nextcloud: Accessible via Tailscale
- [✅] NASPi Jellyfin: Accessible via Tailscale
- [✅] Tailscale network: All devices connected
- [❌] ZTB: Not yet deployed
- [⏳] Mullvad: Config downloaded, pending ZTB setup

## NASPi Services [ACTIVE]

### Pi-hole (DNS Filtering)
- **Service Type:** DNS sinkhole, ad blocking
- **URL (Tailscale):** http://100.68.47.14/admin
- **URL (LAN):** http://192.168.1.223/admin (internal only)
- **Port:** 53 (DNS), 80 (admin web)
- **Authentication:** Password [in password manager]
- **Access Method:** Tailscale required (secure)
- **Status:** ACTIVE
- **Last-Verified:** 2026-04-23
- **Blocklists:** Steven Black's hosts, OISD, Abuse.ch
- **Queries/Day:** ~50,000
- **Blocked Rate:** ~25-30%

### Nextcloud (File Storage)
- **Service Type:** File sync, sharing, collaboration
- **URL (Tailscale):** http://100.68.47.14 (port 80)
- **Port:** 80 (HTTP), 443 (HTTPS not yet setup)
- **Authentication:** Username [in password manager]
- **Access Method:** Tailscale required
- **Status:** ACTIVE
- **Last-Verified:** 2026-04-23
- **Disk Used:** ~500GB
- **Users:** 1 (personal)

### Jellyfin (Media Streaming)
- **Service Type:** Media server, streaming
- **URL (Tailscale):** http://100.68.47.14:8920
- **Port:** 8920
- **Authentication:** Password [in password manager]
- **Access Method:** Tailscale required
- **Status:** ACTIVE
- **Last-Verified:** 2026-04-23
- **Media Library Size:** ~2TB
- **Transcoding:** Enabled (CPU intensive)

## Future Services [ZTB - PENDING]

### Grafana (Monitoring Dashboards)
- **URL (LAN):** http://192.168.1.1:3000
- **URL (Tailscale):** http://100.64.x.x:3000
- **Port:** 3000
- **Status:** PENDING (ZTB not yet deployed)
- **Purpose:** Real-time system and tunnel monitoring

### Prometheus (Metrics Collection)
- **URL (LAN):** http://192.168.1.1:9090
- **URL (Tailscale):** http://100.64.x.x:9090
- **Port:** 9090
- **Status:** PENDING

### Unbound (Recursive DNS)
- **Port:** 5353 (local, not exposed)
- **Status:** PENDING (will be upstream for Pi-hole on ZTB)

## External Services

### Tailscale
- **Service Type:** VPN platform, overlay network
- **Admin Console:** https://login.tailscale.com/admin
- **Account:** [Your Tailscale account email]
- **Tailnet Name:** [Your tailnet domain]
- **Status:** ACTIVE
- **Role:** Manages overlay network, enables remote access
- **MCP Server:** Connected (for Claude Code integration)

### Mullvad VPN
- **Service Type:** Commercial VPN provider
- **Website:** https://mullvad.net
- **WireGuard Config:** Downloaded (stored locally)
- **API Endpoint:** https://api.mullvad.net/ip
- **Status:** PENDING (will be used in ZTB)
- **Authentication:** Account-free (anonymous)
- **Why:** Second encryption layer in ZTB

### GitHub
- **Service Type:** Code storage, backup, version control
- **Repository:** https://github.com/YOUR_USERNAME/homelab-vault
- **Branch:** main
- **Authentication:** SSH key [in password manager]
- **Status:** ACTIVE
- **Role:** Vault backup and multi-device sync
- **MCP Server:** Connected (for Claude Code integration)

### Asana
- **Service Type:** Task management, project tracking
- **Dashboard:** https://app.asana.com
- **Team:** Homelab Project
- **Status:** ACTIVE
- **Role:** Track deployment tasks, specifications
- **MCP Server:** Connected (Claude Code creates tasks)

### Slack
- **Service Type:** Notifications, messaging
- **Workspace:** [Your workspace]
- **Channel:** #homelab
- **Status:** ACTIVE
- **Role:** Claude Code completion notifications
- **MCP Server:** Connected (Claude Code posts updates)

## Access Credentials

⚠️ **Important:** Never store passwords in Logseq!

**All credentials stored in:** Password manager
- 1Password, Bitwarden, or KeePass

**Reference in Logseq:** Use descriptive names without actual values
- "Password [in password manager]"
- "API token [stored securely]"
- "SSH key [password manager entry 'Device SSH Key']"

## Network Security

### Current Setup (Pre-ZTB)
```
Client (Tailscale app)
    ↓ Layer 1: Tailscale encryption
NASPi Pi-hole (100.68.47.14:53)
    ↓ DNS queries unencrypted after Tailscale
Internet (ISP sees encrypted Tailscale traffic)
```

### Future Setup (Post-ZTB)
```
Client (Tailscale app)
    ↓ Layer 1: Tailscale encryption
ZTB Tailscale interface
    ↓ Layer 2: Mullvad encryption
Mullvad exit node
    ↓
Internet (ISP sees encrypted Mullvad traffic only)
```

## Connectivity Testing Commands

```bash
# Test Pi-hole DNS
nslookup google.com 100.68.47.14

# Test Nextcloud
curl -I http://100.68.47.14

# Test Jellyfin
curl -I http://100.68.47.14:8920

# Check Tailscale status
tailscale status

# Verify exit node IP (after ZTB)
curl https://api.mullvad.net/ip | jq .
```

## Backup & Recovery

### What to Backup
- NASPi media library (Jellyfin)
- Nextcloud data
- Configuration files from all services

### Backup Location
- Git: https://github.com/YOUR_USERNAME/homelab-vault
- Local: Pending [TBD]

### Recovery Procedure
1. Check [[Issues-Log]] for known solutions
2. Review service logs: `journalctl -u service-name`
3. Follow troubleshooting in relevant specification page
4. Document issue in [[Issues-Log]] if new
```

### Step 2.4: Commit Phase 2 Changes

```bash
cd ~/logseq/homelab-vault

git add -A
git commit -m "Phase 2: Expanded memory system with device inventory and service locations"
git push origin main
```

---

## Phase 3: Build Documentation (Weeks 3-4)

### Overview

Phase 3 creates pages for each build (ZTB, NASPi, etc.) and detailed specifications for each component. These are the "requirements" that Claude Code will read.

### Step 3.1: Create Main Build Pages

**Why:** Build pages are central hubs where Claude Code finds all specs for a project.

**What they do:** Document what's being built, organize specifications, track progress.

**Context:** When you ask Claude Code to build something, you reference these pages.

#### Create `Builds/ZTB (Zero Trust Bridge).md`

```markdown
# ZTB (Zero Trust Bridge)

**Type:** BUILD
**Status:** PHASE-0 (Hardware assembly phase)
**Priority:** HIGH
**Started:** 2026-04-23
**Est. Completion:** 2026-06-15
**Owner:** [[Memory/Personal-Context]]

## Overview

Enterprise-grade secure home router combining:
- Double encryption (Tailscale + Mullvad)
- Network-wide DNS filtering (Pi-hole)
- Threat detection (Suricata, ntopng)
- Complete monitoring (Grafana, Prometheus)
- Network segmentation (guest, IoT VLAN)

**Architecture:** See [[Networking-Map]]
**Related Builds:** [[NASPi Services]]

## Hardware

**Platform:** Raspberry Pi 5 4GB
**Network Cards:** Onboard Ethernet + Intel AX210 Wi-Fi 6E NIC (via Waveshare HAT)
**OS:** OpenWRT SNAPSHOT (not stable - needed for Pi 5 support)
**Estimated Assembly Time:** 2-3 hours

**Hardware Assembly Steps:**
- [x] Order components (completed 2026-04-15)
- [ ] Assemble Pi5 with heatsinks
- [ ] Install Waveshare PCIe HAT
- [ ] Install Intel AX210 NIC
- [ ] Mount in aluminum case
- [ ] Test power-on

**Related Spec:** [[Hardware-Assembly-Guide.md]] (will be created)

## Phase 0: Hardware Setup
	**Status:** TODO
	**Timeline:** This week (2026-04-24)
	**Deliverables:** Assembled Pi5, flashed OpenWRT, initial network config
	
	- [ ] Assemble hardware (Pi5 + HAT + AX210)
	  - [ ] Install heatsinks on CPU, RAM, PMI
	  - [ ] Insert PCIe HAT
	  - [ ] Insert AX210 NIC
	  - [ ] Mount in case with thermal paste
	
	- [ ] Flash OpenWRT SNAPSHOT
	  - [ ] Download from https://downloads.openwrt.org
	  - [ ] Flash to USB with Raspberry Pi Imager
	  - [ ] Boot and initial LuCI configuration
	
	- [ ] Initial Network Config
	  - [ ] Set eth0 as LAN (192.168.1.0/24)
	  - [ ] Configure AX210 as WiFi AP
	  - [ ] Enable SSH
	  - [ ] Verify web interface (192.168.1.1:80)

## Phase 1: Core Layers [IN PROGRESS]
	**Status:** IN PROGRESS
	**Timeline:** Weeks 2-3 (2026-04-25 - 2026-05-08)
	**Deliverables:** All 5 core layers installed and tested
	
	### Layer 1: DNS Privacy ([[Specs/Layer-1-DNS-Privacy]])
		**Status:** SPECIFICATION READY
		**Component:** Pi-hole + Unbound
		- [ ] Generate install script ([[Generated-Scripts/install-pihole.sh.md]])
		- [ ] Deploy to ZTB
		- [ ] Configure blocklists
		- [ ] Test DNS resolution
		- [ ] Monitor query performance
	
	### Layer 2: Inner Encryption ([[Specs/Layer-2-Tailscale-Inner]])
		**Status:** SPECIFICATION READY
		**Component:** Tailscale
		- [ ] Generate Tailscale config script
		- [ ] Install and configure on ZTB
		- [ ] Configure as exit node
		- [ ] Test client connectivity
		- [ ] Verify encryption working
	
	### Layer 3: Outer Encryption ([[Specs/Layer-3-Mullvad-Outer]])
		**Status:** SPECIFICATION READY
		**Component:** Mullvad WireGuard
		- [ ] Download Mullvad config
		- [ ] Generate configuration script
		- [ ] Configure iptables masquerade
		- [ ] Test double encryption
		- [ ] Verify exit IP is Mullvad
	
	### Layer 4: Threat Detection ([[Specs/Layer-4-Threat-Detection]])
		**Status:** SPECIFICATION READY
		**Component:** Firewall + IDS
		- [ ] Configure OpenWrt firewall
		- [ ] Add threat feed rules
		- [ ] Install and configure ntopng
		- [ ] Install and configure Suricata IDS
		- [ ] Test alert generation
	
	### Layer 5: Monitoring & Visibility ([[Specs/Layer-5-Monitoring]])
		**Status:** SPECIFICATION READY
		**Component:** Prometheus + Grafana + syslog-ng
		- [ ] Install Prometheus
		- [ ] Install Grafana
		- [ ] Create tunnel status dashboard
		- [ ] Install syslog-ng
		- [ ] Configure log aggregation

## Phase 2: Hardening [PLANNED]
	**Status:** PLANNED
	**Timeline:** Week 4 (2026-05-09 - 2026-05-15)
	**Deliverables:** IPv6 protection, network segmentation
	
	- [ ] Layer 6: IPv6 & DNS Protection
	  - [ ] Configure IPv6 firewall
	  - [ ] Test DNS leaks
	  - [ ] Block unused protocols
	
	- [ ] Layer 8: Network Segmentation
	  - [ ] Create guest network (separate SSID)
	  - [ ] Create IoT VLAN
	  - [ ] Configure firewall rules
	  - [ ] Test isolation

## Phase 3: Advanced Features [PLANNED]
	**Status:** PLANNED
	**Timeline:** Weeks 5+ (2026-05-16+)
	**Deliverables:** VPN monitoring, kill switch, threat feeds
	
	- [ ] Layer 9: VPN Monitoring & Kill Switch
	  - [ ] Generate health check scripts
	  - [ ] Create auto-restart logic
	  - [ ] Implement kill switch
	  - [ ] Test failure recovery
	
	- [ ] Layer 10: Threat Feeds
	  - [ ] Add malware domain lists
	  - [ ] Add phishing domain lists
	  - [ ] Auto-update feeds
	  - [ ] Test blocking
	
	- [ ] Layer 12: Automated Response
	  - [ ] Install and configure Fail2ban
	  - [ ] Create high-confidence alert rules
	  - [ ] Test auto-blocking

## Generated Scripts

All scripts for this build stored in [[Generated-Scripts/]]:

- [[Generated-Scripts/phase-0-hardware-setup.md]]
- [[Generated-Scripts/install-pihole.sh.md]]
- [[Generated-Scripts/install-tailscale.sh.md]]
- [[Generated-Scripts/configure-mullvad.sh.md]]
- [[Generated-Scripts/configure-firewall.sh.md]]
- [More as generated...]

## Known Issues

- [[Issues-Log/Mullvad-Config-Timeout]] - WireGuard endpoint timeout
- [[Issues-Log/AX210-Driver-Support]] - WiFi NIC driver compatibility
- [[Issues-Log/OpenWRT-Snapshot-Stability]] - SNAPSHOT branch instability

## Timeline & Dependencies

```
Phase 0 Hardware
    ↓
Phase 1 Layer 1 (DNS) → Phase 1 Layer 2 (Tailscale) → Phase 1 Layer 3 (Mullvad)
    ↓
Phase 1 Layer 4 (Firewall)
    ↓
Phase 1 Layer 5 (Monitoring)
    ↓
Phase 2 Hardening
    ↓
Phase 3 Advanced
```

## Success Criteria

When complete, ZTB will:
- ✅ Block ads/malware at DNS level
- ✅ Encrypt all traffic via Tailscale (Layer 2)
- ✅ Re-encrypt via Mullvad (Layer 3)
- ✅ Block malicious IPs via firewall
- ✅ Detect suspicious traffic via IDS
- ✅ Monitor all metrics via Grafana
- ✅ Log all events centrally
- ✅ Segment networks (guest, IoT)
- ✅ Auto-restart if tunnels fail
- ✅ ISP sees only Mullvad traffic
- ✅ No DNS leaks
- ✅ Complete ownership of security

## Related

- [[Memory/Networking-Map]] - Network architecture
- [[Memory/Device-Inventory]] - ZTB device details
- [[Issues-Log]] - Problems and solutions
- [[Daily-Logs]] - Build progress journal
```

#### Create `Builds/NASPi Services.md`

```markdown
# NASPi Services

**Type:** BUILD
**Status:** ACTIVE
**Priority:** MEDIUM
**Started:** 2026-03-15
**Owner:** [[Memory/Personal-Context]]

## Overview

Raspberry Pi running Ubuntu Server with:
- Pi-hole (DNS filtering)
- Nextcloud (file storage)
- Jellyfin (media streaming)
- All on Tailscale network for secure access

## Current Services

### Pi-hole (DNS Filtering)
- **Status:** ACTIVE
- **Installation:** Docker container
- **Configuration:** [[Memory/Service-Locations#Pi-hole]]
- **Blocklists:** Steven Black, OISD, Abuse.ch
- **Performance:** ~50k queries/day, 25-30% blocked
- **Last-Updated:** 2026-04-15

### Nextcloud (File Storage)
- **Status:** ACTIVE
- **Installation:** Docker container
- **Configuration:** [[Memory/Service-Locations#Nextcloud]]
- **Storage:** ~500GB
- **Users:** 1 (personal)
- **Last-Updated:** 2026-04-15

### Jellyfin (Media Streaming)
- **Status:** ACTIVE
- **Installation:** Docker container
- **Configuration:** [[Memory/Service-Locations#Jellyfin]]
- **Library Size:** ~2TB
- **Last-Updated:** 2026-04-15

## Future Enhancements

- [ ] Add backup service
- [ ] Add monitoring agent
- [ ] Increase storage capacity
- [ ] Add redundancy

## Related

- [[Memory/Service-Locations]] - How to access services
- [[Memory/Device-Inventory#NASPi]] - Hardware details
- [[Generated-Scripts]] - Deployment scripts
```

### Step 3.2: Create Specification Pages

**Why:** Specifications define WHAT to build. Claude Code reads these to generate code.

**What they do:** Document detailed requirements for each component.

**Context:** These are the "requirements documents" that guide code generation.

#### Create `Specs/Layer-1-DNS-Privacy.md`

```markdown
# Layer 1: DNS Privacy (Pi-hole + Unbound)

**Type:** SPEC
**Build:** [[ZTB (Zero Trust Bridge)]]
**Status:** DESIGN (ready for implementation)
**Created:** 2026-04-23
**Version:** 1.0
**Priority:** HIGH

## Purpose

Network-wide DNS filtering using Pi-hole combined with Unbound recursive resolver.

Creates a complete DNS privacy solution:
- Blocks ad/malware domains at DNS level
- Queries root nameservers directly (no ISP DNS)
- Prevents DNS leakage even if encryption fails
- Provides network-wide protection (all clients benefit)

## Specification

### Pi-hole Requirements

**Function:** DNS sinkhole, ad-blocking, malware filtering

**Installation:**
- Docker container on ZTB
- Listens on port 53 (DNS) and 80 (admin web)
- Upstream server: Unbound (localhost:5353)

**Blocklists to configure:**
- Steven Black's hosts compilation (ads + malware)
- OISD (tracking and ads)
- Abuse.ch (malware)

**Configuration:**
- Auto-update blocklists hourly
- Log all DNS queries
- Web admin interface with stats
- Configure gravity database for custom rules

**Expected Performance:**
- Support 50+ concurrent DNS queries
- Block 20-30% of requests (ads/tracking)
- Return results in <10ms

### Unbound Requirements

**Function:** Recursive DNS resolver

**Installation:**
- System package on ZTB
- Listens on port 5353 (local only)
- Used as upstream for Pi-hole (not exposed to clients)

**Configuration:**
- Enable DNSSEC validation
- Query root nameservers directly (no forwarding to ISP)
- Local cache for performance
- Configure trusted networks (localhost + Tailscale)

**Expected Performance:**
- Resolve queries in <20ms average
- Handle 100+ simultaneous queries

## Data Flow

```
Client (Tailscale)
    ↓ DNS query to Pi-hole
192.168.1.1:53 (Pi-hole)
    ↓ Query upstream for unknown domain
localhost:5353 (Unbound)
    ↓ Query root nameservers
Internet root DNS servers
    ↓ Response back through chain
Client gets DNS answer
```

## Testing Requirements

MUST test these scenarios:

1. **Ad domain blocking:**
   ```bash
   nslookup ads.google.com
   # Should return Pi-hole blocking IP (127.0.0.1 or 0.0.0.0)
   ```

2. **Normal domain resolution:**
   ```bash
   nslookup google.com
   # Should resolve to actual IP
   ```

3. **Unbound recursive resolution:**
   ```bash
   # Verify Unbound is being used upstream
   # Check Pi-hole admin logs for Unbound queries
   ```

4. **No DNS leaks:**
   - Visit https://www.dnsleaktest.com/
   - Should show only Pi-hole/Unbound as resolver
   - Should NOT show ISP DNS

5. **Performance:**
   - Response time <50ms for cached queries
   - No timeout errors
   - Handle 100+ simultaneous queries

## Success Criteria

✅ All DNS queries through Pi-hole
✅ Zero DNS queries leak to ISP
✅ Ad domain blocking at 95%+ effectiveness
✅ Monitoring shows query volume and blocked count
✅ Unbound resolves queries from root servers
✅ No timeout errors or performance issues

## Dependencies

- [[Specs/Layer-2-Tailscale-Inner]] (DNS accessible via Tailscale)
- [[Memory/Device-Inventory#ZTB]] (hardware to run on)

## Implementation

When Claude Code is asked to implement this layer:

1. Generate install script: `install-pihole.sh`
2. Generate Unbound config: `unbound.conf`
3. Test DNS resolution locally
4. Document in [[Generated-Scripts/install-layer-1-dns.md]]
5. Create Asana task for deployment
6. Notify Slack when ready

## References

- Pi-hole docs: https://docs.pi-hole.net/
- Unbound docs: https://nlnetlabs.nl/projects/unbound/about/
- Gravity database: https://docs.pi-hole.net/database/
```

Do the same for Layers 2-5 (outline provided earlier in the conversation).

### Step 3.3: Create Templates for Generated Scripts

**Why:** Templates ensure Claude Code outputs consistent, documented scripts.

**What they do:** Define the structure for documentation pages about generated code.

**Context:** Every script Claude Code generates gets a documentation page following this template.

#### Create `templates/Generated-Script-Template.md`

```markdown
# [Script Name]

**Type:** SCRIPT
**Generated:** [Date]
**Generator:** Claude Code
**Created-From:** [[Link to spec]]
**Status:** GENERATED (not yet tested)
**Tested-On:** [Device, or "pending test"]
**Deployed-To:** [Device, or "pending deployment"]
**Version:** 1.0

## Purpose

[One-sentence description of what this script does]

## Specification

[Link to originating spec]

Key requirements implemented:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

## Script Code

```bash
#!/bin/bash
[Full script content with comments]
```

## Testing

### Test Procedure
1. [Step 1]
2. [Step 2]
3. [Verification steps]

### Test Results
- Date tested: [Date]
- Device: [[Device name]]
- Result: ✅ [Pass or ❌ Fail with details]
- Issues found: [None, or description]

## Deployment

### Prerequisites
- [Package 1]
- [Access or permission required]

### Deployment Steps
1. Copy script to target
2. Make executable
3. Test with `--test` flag
4. Enable (systemd, cron, etc.)

### Rollback
If issues:
1. [Rollback step]
2. Check logs
3. Investigate

## Known Issues & Limitations

- [Issue 1 - Workaround]
- [Limitation 1 - Reason]

## Related

- depends-on:: [[Spec-this-implements]]
- related-to:: [[Other-scripts]]
- deployment-task:: [[Link to Asana task]]
```

### Step 3.4: Commit Phase 3

```bash
cd ~/logseq/homelab-vault

git add -A
git commit -m "Phase 3: Created build pages and specifications"
git push origin main
```

---

## Phase 4: Synchronization (Week 4)

### Overview

Phase 4 sets up automatic syncing of your vault across multiple devices via Git.

### Step 4.1: Create Auto-Sync Script

**Why:** Automates Git sync so you don't manually push/pull.

**What it does:** Periodically pulls latest changes and pushes new commits.

**Context:** Ensures all devices stay in sync without manual intervention.

Create `setup/sync-script.sh`:

```bash
#!/bin/bash
# Auto-sync Logseq vault across devices

set -euo pipefail

VAULT_DIR="${VAULT_DIR:-$HOME/logseq/homelab-vault}"
GIT_REMOTE="${GIT_REMOTE:-origin}"
GIT_BRANCH="${GIT_BRANCH:-main}"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Check if vault exists
if [ ! -d "$VAULT_DIR" ]; then
    log "ERROR: Vault directory not found at $VAULT_DIR"
    exit 1
fi

cd "$VAULT_DIR"

# Pull latest changes
log "Pulling latest from $GIT_REMOTE/$GIT_BRANCH..."
if ! git pull "$GIT_REMOTE" "$GIT_BRANCH"; then
    log "ERROR: Failed to pull from remote"
    exit 1
fi

# Check for uncommitted changes
if [ -n "$(git status --porcelain)" ]; then
    log "Found uncommitted changes, staging and committing..."
    git add -A
    git commit -m "Auto-sync: $(date +'%Y-%m-%d %H:%M:%S')"
fi

# Push changes
log "Pushing to $GIT_REMOTE/$GIT_BRANCH..."
if git push "$GIT_REMOTE" "$GIT_BRANCH"; then
    log "✅ Sync complete"
else
    log "⚠️  Sync complete but push failed (check network)"
fi
```

Make it executable:

```bash
chmod +x ~/logseq/homelab-vault/setup/sync-script.sh
```

### Step 4.2: Set Up Cron Job for Auto-Sync

**Linux/macOS:**

```bash
# Open crontab
crontab -e

# Add these lines:
# Sync every 5 minutes during work hours (8am-10pm)
*/5 8-22 * * * cd ~/logseq/homelab-vault && bash setup/sync-script.sh >> ~/.logseq-sync.log 2>&1

# Sync every 30 minutes outside work hours
*/30 23,0-7 * * * cd ~/logseq/homelab-vault && bash setup/sync-script.sh >> ~/.logseq-sync.log 2>&1
```

**Windows (Task Scheduler):**

1. Open Task Scheduler
2. Create Basic Task
3. Name: "Logseq Auto-Sync"
4. Trigger: Every 5 minutes
5. Action: Start program
   - Program: `bash`
   - Arguments: `-c "cd ~/logseq/homelab-vault && bash setup/sync-script.sh"`

### Step 4.3: Test Sync on Second Device

```bash
# On laptop, desktop, or other machine

# Clone vault
git clone https://github.com/YOUR_USERNAME/homelab-vault.git ~/logseq/homelab-vault

# Install Logseq if needed
# (Use your package manager or download from logseq.com)

# Open vault in Logseq
# Settings → "Open a local directory" → Select ~/logseq/homelab-vault

# Set up auto-sync cron job (see Step 4.2)
crontab -e
# Add same cron lines as primary device

# Test sync
cd ~/logseq/homelab-vault
git pull origin main
git log --oneline -5  # Should show recent commits from primary device
```

### Step 4.4: Commit Sync Scripts

```bash
cd ~/logseq/homelab-vault

git add -A
git commit -m "Phase 4: Added auto-sync scripts for multi-device"
git push origin main
```

---

## Phase 5: Claude Code Integration (Weeks 4-5)

### Overview

Phase 5 connects Claude Code to your vault and sets up MCP servers for automated workflows.

### Step 5.1: Enable MCP Servers

**Why:** MCP servers enable Claude Code to integrate with GitHub, Asana, Slack for automation.

**What they do:** Allow Claude Code to programmatically interact with external services.

**Context:** When Claude Code generates a script, it can automatically create Asana task and notify Slack.

#### Set Environment Variables

```bash
# Add to ~/.bashrc or ~/.zshrc

# GitHub
export GITHUB_TOKEN="your_github_personal_access_token"

# Asana
export ASANA_TOKEN="your_asana_api_token"

# Slack
export SLACK_BOT_TOKEN="your_slack_bot_token"

# Optional: Vault path (for easy reference)
export LOGSEQ_VAULT="$HOME/logseq/homelab-vault"
```

#### Create Personal Access Tokens

**GitHub:**
1. Go to https://github.com/settings/tokens
2. Click "Generate new token" → "Generate new token (classic)"
3. Select scopes: `repo`, `gist`, `user`
4. Copy token and add to ~/.bashrc

**Asana:**
1. Go to https://app.asana.com/account/my-apps
2. Click "New access token"
3. Copy token and add to ~/.bashrc

**Slack:**
1. Create Slack bot (if not exists)
2. Go to https://api.slack.com/apps
3. Create new app or select existing
4. Copy bot token and add to ~/.bashrc

### Step 5.2: Test Claude Code with Vault

**Why:** Verify Claude Code can read from Logseq vault correctly.

**What it does:** Generate a simple script to confirm integration works.

**Context:** First test of the complete system.

#### Example Test Request

```bash
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

Read pages:
- [[Home]]
- [[Memory/Personal-Context]]
- [[Memory/Device-Inventory]]

Generate:
- Summary of build status
- List of available devices
- Current priority tasks

Output:
- Just output the summary to console
- Don't create files yet
"
```

**Expected output:**
Claude Code reads your vault, understands your setup, provides a summary.

#### Example Real Request

```bash
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

Read specifications:
- [[Specs/Layer-1-DNS-Privacy]]
- [[Claude-Config/Coding-Standards#Bash Scripts]]
- [[Memory/Device-Inventory]]

Generate:
- Complete Pi-hole installation script
- Support --test flag
- Add comprehensive logging

Test:
- Run with --test flag
- Verify no errors

Document:
- Create [[Generated-Scripts/install-pihole.sh.md]]
- Include full script, test results, deployment steps
- Update [[ZTB (Zero Trust Bridge)]] with link

Notify:
- Create Asana task: 'Deploy Pi-hole to ZTB'
- Post to Slack: Script ready for testing
"
```

---

## Phase 6: Knowledge Base Building (Weeks 5-8)

### Overview

Phase 6 focuses on actually using the system—generating scripts, logging issues, tracking progress.

### Step 6.1: Start Daily Logging

Logseq automatically creates daily journal pages. Use them:

**Daily journal entry (`journals/2026-04-23.md`):**

```markdown
# 2026-04-23

## Today's Work

### ZTB Hardware
- [x] Ordered components
- [x] Received Pi5
- [ ] Begin assembly this week

### Claude Code Testing
- [x] Tested vault reading
- [x] Verified GitHub integration
- [ ] Generate first Layer 1 script

## Issues Found
- None yet

## Progress
- ZTB ready for Phase 0 assembly
- Vault structure complete
- MCP servers configured

## Tomorrow's Plan
- [ ] Assemble Pi5 hardware
- [ ] Flash OpenWRT SNAPSHOT
- [ ] Generate Phase 0 scripts
```

### Step 6.2: Log Issues as They Arise

When you encounter problems, document in [[Issues-Log]]:

```markdown
# [Issue Title]

**Type:** ISSUE
**Severity:** [LOW/MEDIUM/HIGH]
**Status:** [OPEN/IN-INVESTIGATION/RESOLVED]
**Created:** 2026-04-23
**Affects:** [[Related build or layer]]

## Description
[What's wrong]

## Symptoms
[What you observe]

## Root Cause (if known)
[Why this happens]

## Investigation Steps Tried
1. [What you tried]
2. [Results]

## Solution (when found)
[How to fix it]

## Related
- [[Related issue]]
- [[Related spec]]
```

### Step 6.3: Weekly Reviews

Every Sunday, create weekly summary:

```markdown
# Week of 2026-04-20 Review

## Summary
- Completed Phase 1-2 of implementation
- Vault fully functional across 2 devices
- Ready to start Phase 3

## What Went Well
- Git sync working smoothly
- Logseq interface intuitive
- Claude Code integration straightforward

## What Needs Improvement
- Need more example specs
- Could use more templates

## Next Week
- [ ] Begin Phase 3 build documentation
- [ ] Generate first scripts from Layer 1 spec
- [ ] Test deployment to dev machine
```

---

## Phase 7: Monitoring & Maintenance (Ongoing)

### Overview

Phase 7 establishes processes for keeping your system healthy and up-to-date.

### Step 7.1: Monthly Maintenance

Every month (1st of month):

1. **Backup check:**
   ```bash
   # Verify GitHub has latest changes
   cd ~/logseq/homelab-vault
   git log --oneline -1  # Shows last commit
   ```

2. **Update review:**
   - Check Logseq for software updates
   - Review Logseq release notes
   - Update if stable version available

3. **Vault review:**
   - Read through recent [[Daily-Logs]]
   - Update [[Memory/Personal-Context]] if goals changed
   - Update [[Memory/Device-Inventory]] with current status
   - Archive old issues from [[Issues-Log]]

### Step 7.2: Quarterly Reviews

Every 3 months (1st of quarter):

1. **Performance analysis:**
   - Are devices meeting performance needs?
   - Any bottlenecks?
   - Update [[Memory/Device-Inventory]]

2. **Goal progress:**
   - Review [[Memory/Long-Term-Goals]]
   - Update build status
   - Adjust timelines if needed

3. **Process improvement:**
   - Any pain points in workflow?
   - Could templates be better?
   - Update [[Claude-Config]] pages if needed

---

## Troubleshooting

### Git Sync Issues

**Problem:** "fatal: refusing to merge unrelated histories"
**Solution:**
```bash
cd ~/logseq/homelab-vault
git pull origin main --allow-unrelated-histories
```

**Problem:** "Updates were rejected because the remote contains work that you do not have locally"
**Solution:**
```bash
cd ~/logseq/homelab-vault
git pull origin main
git push origin main
```

### Logseq Issues

**Problem:** Pages not showing up after git pull
**Solution:**
1. Close Logseq
2. Run: `cd ~/logseq/homelab-vault && git pull origin main`
3. Reopen Logseq
4. Refresh: Cmd+R (macOS) or F5 (Windows)

**Problem:** Logseq asking for merge conflicts
**Solution:**
- Close Logseq
- Resolve conflicts in terminal
- Reopen Logseq

### Claude Code Issues

**Problem:** Claude Code can't find vault directory
**Solution:**
```bash
claude-code "
My Logseq vault is at: ~/logseq/homelab-vault
Verify this path is correct
List all pages in the vault
"
```

**Problem:** Claude Code not finding [[specific page]]
**Solution:**
- Verify page exists in Logseq
- Check page name exactly (case-sensitive)
- Try: `claude-code "List all pages starting with 'Claude'"`

---

## Common Workflows

### Workflow 1: Generate Installation Script from Spec

```bash
# 1. Open Logseq, update Layer 1 spec with new requirement

# 2. Run Claude Code
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

Read:
- [[Specs/Layer-1-DNS-Privacy]]
- [[Claude-Config/Coding-Standards#Bash Scripts]]
- [[Memory/Device-Inventory#ZTB]]

Generate:
- Pi-hole installation script
- All requirements from spec
- Support --test flag
- Comprehensive logging

Test:
- Run with --test
- Simulate on Arch Linux ROG
- Verify no errors

Document:
- Create [[Generated-Scripts/install-pihole.sh.md]]
- Update [[ZTB (Zero Trust Bridge)]] with link
- Create Asana task: 'Deploy Pi-hole to ZTB'
"

# 3. Review generated script in Logseq
# 4. Approve for deployment
# 5. Claude Code handles rest
```

### Workflow 2: Troubleshoot Issue

```bash
# 1. Document issue in [[Issues-Log]]
#   - What's wrong
#   - Symptoms
#   - Investigation steps tried

# 2. Run Claude Code
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

Read:
- [[Issues-Log/[Issue-Name]]]
- [[Related-Spec-Page]]
- [[Memory/Device-Inventory]]
- [[Issues-Log]] for similar issues

Analysis:
- Root cause analysis
- Potential solutions
- Recommended fix

Generate:
- Diagnostic script
- Troubleshooting steps
- Fix procedure

Document:
- Update [[Issues-Log/[Issue-Name]]] with solution
- Create [[Generated-Scripts/diagnostic-[issue].sh.md]]
"

# 3. Review diagnostic script
# 4. Run on affected device
# 5. Log results in Issues-Log
```

### Workflow 3: Add New Device

```bash
# 1. Update [[Memory/Device-Inventory]] with new device
#   - Device specs
#   - IP address
#   - Capabilities

# 2. Run Claude Code to generate setup scripts
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault

New device added to [[Memory/Device-Inventory]]

Generate:
- Device-specific configuration
- Installation scripts
- Setup verification script

Test:
- Verify scripts on similar device

Document:
- Create scripts in [[Generated-Scripts]]
"

# 3. Deploy to new device
# 4. Update [[Memory/Device-Inventory]] with actual data
```

---

## Success Indicators

When your system is working well, you'll see:

✅ **Automatic syncing:** Logseq updates across devices without manual git commands
✅ **Claude Code reading vault:** Can ask it about your setup without repeating context
✅ **Generated scripts ready:** Claude Code outputs production-ready code with full docs
✅ **Issues tracked:** [[Issues-Log]] documents problems and solutions
✅ **Build progress visible:** [[Daily-Logs]] show consistent progress
✅ **MCP integration working:** Asana tasks created, Slack notifications arriving
✅ **Multi-device consistency:** Same behavior from Claude Code on any device
✅ **Time savings:** Hours per week saved on code generation and documentation

---

## Next Steps

1. **Completed Phase 1?** Start Phase 2 (Memory system)
2. **Completed Phase 2?** Start Phase 3 (Build documentation)
3. **Completed Phase 3?** Start Phase 4 (Synchronization)
4. **All foundations done?** Move to Phase 5 (Claude Code integration)
5. **Ready to build?** Use Phase 6 (Knowledge building) for real work

---

## Support & Questions

- **Setup issues?** Check troubleshooting section
- **Not sure how to use?** See "Common Workflows"
- **Want to share improvements?** Create pull request in GitHub
- **Found bugs/gaps?** Open issue in repository

---

**Happy building! Your Logseq vault is now Claude Code's extended mind.** 🧠
