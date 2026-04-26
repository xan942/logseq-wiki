# Logseq as Claude Code's Central Memory Store

Transform Logseq into a persistent, multi-device knowledge system that serves as Claude Code's authoritative source of truth across all sessions and devices.

## Overview

This repository contains a complete implementation plan and setup guide for using Logseq as a centralized memory store for Claude Code—enabling consistent, autonomous code generation and system automation across multiple devices and sessions.

### What This Achieves

Instead of scattered configuration files, memory prompts, and context spread across multiple `.claude` directories on different devices, this system provides:

- **Single Source of Truth:** All specifications, configurations, standards, and context stored in one Logseq vault
- **Multi-Device Consistency:** Same behavior and output from Claude Code regardless of which device you're using
- **Persistent Memory:** Claude Code reads from Logseq and understands full context of your projects
- **Automated Workflows:** Generate, test, document, and deploy code through integrated MCP servers (GitHub, Asana, Slack)
- **Knowledge Preservation:** Future you (and future Claude Code instances) remember decisions, issues, and solutions
- **Git-Backed Sync:** All changes version-controlled via Git for audit trail and recovery

### Key Innovation

Instead of telling Claude Code "remember this" with memory files scattered everywhere, you document specifications in Logseq and Claude Code reads them. Your system becomes self-documenting, with generated code and issues automatically linked back to their source specifications.

```
Before:
Config files → Memory files → Multiple devices → Inconsistent behavior

After:
Logseq Vault (single source) → Git (multi-device sync) → Claude Code (reads context) → Consistent automation
```

## Quick Start

### 1. Clone This Repository

```bash
git clone https://github.com/YOUR_USERNAME/logseq-claude-code.git
cd logseq-claude-code
```

### 2. Follow the Implementation Plan

Read `logseq-guide.md` for step-by-step setup instructions across 7 phases:

- **Phase 1:** Foundation (Logseq installation, Git setup, core architecture)
- **Phase 2:** Memory System (personal context, device inventory, service locations)
- **Phase 3:** Build Documentation (main build pages, specifications, templates)
- **Phase 4:** Synchronization (multi-device sync via Git, auto-sync scripts)
- **Phase 5:** Claude Code Integration (MCP servers, first test requests)
- **Phase 6:** Knowledge Base Building (issues log, daily logs, templates)
- **Phase 7:** Monitoring & Maintenance (weekly reviews, monthly maintenance)

### 3. Set Up Your Own Vault

```bash
# Follow Phase 1 instructions in logseq-guide.md to create your vault
# Key steps:
# 1. Install Logseq
# 2. Create vault directory
# 3. Initialize Git
# 4. Configure Logseq
# 5. Create initial pages
# 6. Push to GitHub
```

### 4. Start Using Claude Code

```bash
claude-code "
Read my Logseq vault at ~/logseq/homelab-vault
Read: [[Your-Spec-Page]]
Read: [[Claude-Config/Coding-Standards]]
Generate: [What you want]
Test: [How to verify]
Document: [Where to save]
"
```

## Why This System?

### Problems It Solves

| Problem | Solution |
|---------|----------|
| Scattered memory files across devices | Single Logseq vault, synced via Git |
| Inconsistent Claude Code behavior | Agent-Personality.md read by every instance |
| Forgetting design decisions | All specs documented in Logseq, linked to code |
| Duplicate specifications | One spec per build/layer, referenced everywhere |
| Lost context between sessions | Full context available in vault (device inventory, service locations, memory) |
| No audit trail of changes | Git history tracks every spec update, generated script, issue logged |
| Tedious documentation | Documentation generated automatically by Claude Code |
| Multiple deployment processes | Asana tasks auto-created, Slack notifications automatic |

### Real-World Benefits

✅ **Time Savings:** Claude Code generates and tests code automatically (hours per week)
✅ **Consistency:** Same standards and behavior across all generation sessions
✅ **Learning:** Future you understands why past decisions were made
✅ **Collaboration:** If sharing with team, vault is single source of truth
✅ **Recovery:** Git history lets you revert bad specs or generated code
✅ **Scalability:** Add new builds/projects by following folder structure
✅ **Intelligence:** Logseq queries generate reports; Claude Code can read and act on them

## Architecture

### Vault Structure

```
~/logseq/homelab-vault/
├── pages/
│   ├── Home.md                    # Central entry point
│   ├── Claude-Config/
│   │   ├── Agent-Personality.md  # How Claude Code behaves
│   │   ├── Coding-Standards.md   # Code generation standards
│   │   ├── How-To-Use-Claude-Code.md
│   │   └── Documentation-Templates.md
│   ├── Builds/
│   │   ├── ZTB (Zero Trust Bridge).md
│   │   ├── NASPi Services.md
│   │   └── [Your builds]
│   ├── Specs/
│   │   ├── Layer-1-DNS-Privacy.md
│   │   ├── Layer-2-Tailscale-Inner.md
│   │   ├── Layer-3-Mullvad-Outer.md
│   │   ├── Layer-4-Threat-Detection.md
│   │   └── Layer-5-Monitoring.md
│   ├── Memory/
│   │   ├── Personal-Context.md
│   │   ├── Device-Inventory.md
│   │   ├── Service-Locations.md
│   │   ├── Networking-Map.md
│   │   └── Long-Term-Goals.md
│   ├── Generated-Scripts/
│   │   ├── [script-name].sh.md
│   │   ├── [config-name].yml.md
│   │   └── [All Claude Code outputs]
│   ├── Issues-Log/
│   │   ├── [Issue-1].md
│   │   ├── [Issue-2].md
│   │   └── [Problems & solutions]
│   ├── Daily-Logs/
│   │   └── [Build progress journal]
│   ├── archives/
│   │   └── [Resolved issues & old pages]
│   └── setup/
│       └── sync-script.sh
├── journals/
│   ├── 2026_04_23.md
│   └── [Auto-created daily]
├── logseq/
│   └── config.edn              # Logseq configuration
├── .git/                        # Git repository
├── .gitignore
└── README.md
```

### Sync Architecture

```
Primary Device
    ↓ (git push)
GitHub Private Repo
    ↓ (git pull)
Secondary Devices (Laptop, Desktop, etc.)
    ↓
All devices running same Logseq vault (synced via Git)
    ↓
Claude Code reads from local vault on any device
    ↓
Consistent behavior and context regardless of which device invokes Claude Code
```

## Core Concepts

### 1. Logseq as Knowledge Base

Logseq provides:
- **Outline-based hierarchy:** Specs are one structured page with nested tasks
- **Bidirectional links:** Pages automatically link to related pages
- **Graph visualization:** See relationships between specs, configs, scripts, devices
- **Task tracking:** TODO/DOING/DONE with dependencies and deadlines
- **Queries:** Ask Logseq questions ("show me all HIGH priority tasks")
- **Daily journal:** Build log automatically created each day

### 2. Git for Multi-Device Sync

Git provides:
- **Version control:** Track every change to specs and configurations
- **Multi-device sync:** `git pull` before starting, `git push` after finishing
- **Conflict resolution:** Manual resolution of edit conflicts (rare with 1-2 users)
- **Backup:** GitHub serves as automatic backup
- **Audit trail:** `git log` shows who changed what and when

### 3. Claude Code as Automation Engine

Claude Code:
- **Reads specifications** from Logseq vault
- **Follows standards** defined in Claude-Config pages
- **Generates code** tailored to your setup
- **Tests implementations** before marking complete
- **Documents results** in Generated-Scripts pages
- **Integrates with external tools** (GitHub, Asana, Slack) via MCP servers

### 4. MCP Servers for Integration

MCP servers connect Claude Code to:
- **GitHub:** Store generated scripts and configurations
- **Asana:** Create deployment tasks automatically
- **Slack:** Notify when scripts are ready
- **Google Drive:** Archive backups (optional)

## Use Cases

### Use Case 1: Building a Secure Home Router

```
1. Document ZTB architecture in [[ZTB (Zero Trust Bridge)]]
2. Create specifications for each layer:
   - [[Specs/Layer-1-DNS-Privacy]]
   - [[Specs/Layer-2-Tailscale-Inner]]
   - [[Specs/Layer-3-Mullvad-Outer]]
   - etc.
3. Run Claude Code for Layer 1:
   - Reads spec from Logseq
   - Generates Pi-hole installation script
   - Tests on dev machine
   - Documents in [[Generated-Scripts/install-pihole.sh.md]]
   - Creates Asana task for deployment
4. Repeat for each layer
5. Monitor progress in daily logs and Grafana dashboards
```

### Use Case 2: Scaling Infrastructure

When adding new devices:
```
1. Update [[Memory/Device-Inventory]] with new device
2. Run Claude Code:
   - Reads device inventory
   - Generates device-specific configs
   - Creates deployment scripts
3. Deploy to new device
4. Update inventory with actual data
```

### Use Case 3: Fixing Issues Across Multiple Devices

```
1. Document issue in [[Issues-Log/Issue-Name.md]]
2. Reference relevant specs and devices
3. Run Claude Code:
   - Reads issue description
   - Reads affected device specs
   - Generates diagnostic script
   - Tests fix procedure
4. Deploy fix to all affected devices
5. Update issue status to RESOLVED
```

## Getting Started: Real Example

### Example 1: Generate Health Check Script

You want Claude Code to generate a script that monitors your encryption tunnels (Tailscale + Mullvad).

**In Logseq, you:**
1. Update [[ZTB (Zero Trust Bridge)#Phase 1#Layer 2]] with spec:
   ```
   - Spec: Health check script
     - Monitor Tailscale tunnel every 60 seconds
     - Monitor Mullvad tunnel every 60 seconds
     - Auto-restart if either down
     - Log to syslog
     - Alert to Slack if failures
   ```

2. Run Claude Code:
   ```bash
   claude-code "
   Read my Logseq vault at ~/logseq/homelab-vault
   Read: [[ZTB (Zero Trust Bridge)#Phase 1#Layer 2]]
   Read: [[Claude-Config/Coding-Standards#Bash Scripts]]
   Read: [[Memory/Device-Inventory]]
   
   Generate:
   - Complete health check bash script
   - Support --test flag for dry run
   - Log every action
   
   Test:
   - Verify script runs without errors
   - Test on Arch Linux ROG (dev machine)
   - Simulate tunnel failure
   - Verify auto-restart works
   
   Document:
   - Create [[Generated-Scripts/tunnel-health-check.sh.md]]
   - Include full script, testing results, deployment steps
   - Update [[ZTB (Zero Trust Bridge)]] with link
   
   Notify:
   - Create Asana task 'Deploy health check to ZTB'
   - Post to Slack with results
   "
   ```

3. Claude Code:
   - Reads your spec from Logseq (complete context)
   - Reads Coding Standards (knows your preferred bash style)
   - Reads Device Inventory (knows target devices)
   - Generates complete script with error handling and logging
   - Tests on your dev machine (Arch Linux ROG)
   - Creates documentation page in Logseq with:
     - Full script code
     - Test results
     - Deployment instructions
     - Known limitations
   - Updates source spec page with link
   - Creates Asana task for deployment
   - Posts to Slack

4. You:
   - Review generated script (takes 2 minutes)
   - Approve deployment
   - Claude Code helps deploy to ZTB
   - Update deployment status in Logseq

**Total time:** 10 minutes (most of which is waiting for Claude Code to run)
**Result:** Production-ready, tested script with full documentation

## Prerequisites

### To Use This System

- **Logseq:** Download from https://logseq.com/downloads
- **Git:** Already installed on macOS/Linux, available on Windows
- **GitHub account:** Free tier works (private repo recommended)
- **Claude Code:** `npm install -g @anthropic/claude-code` (or use Claude.ai)
- **Basic understanding:**
  - How to use Markdown
  - Basic Git operations (clone, pull, push, commit)
  - How to use your terminal

### Optional But Recommended

- **MCP Servers:** Connect GitHub, Asana, Slack for automation (see logseq-guide.md)
- **Password manager:** Store secrets (Pi-hole password, API tokens, etc.)
- **Cron/Task scheduler:** Set up auto-sync (Linux/macOS) or Task Scheduler (Windows)

## Implementation Timeline

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| Phase 1: Foundation | 1-2 weeks | Logseq vault, Git setup, initial pages |
| Phase 2: Memory System | 1 week | Personal context, device inventory, services |
| Phase 3: Build Documentation | 2 weeks | Build pages, specifications, templates |
| Phase 4: Synchronization | 3-5 days | Multi-device sync setup |
| Phase 5: Claude Code Integration | 3-5 days | MCP servers, first test requests |
| Phase 6: Knowledge Building | 2-4 weeks | Daily logs, issues, generated scripts |
| Phase 7: Maintenance | Ongoing | Weekly reviews, monthly updates |

**Total:** 6-8 weeks to full implementation

**But:** You have a working system after Phase 1-2 (2-3 weeks). Phases 3-7 add sophistication.

## File Structure of This Repository

```
.
├── README.md                          # This file
├── logseq-guide.md                    # Complete step-by-step guide
├── templates/
│   ├── Home.md
│   ├── Agent-Personality.md
│   ├── Coding-Standards.md
│   ├── How-To-Use-Claude-Code.md
│   ├── Personal-Context.md
│   ├── Device-Inventory.md
│   ├── Service-Locations.md
│   ├── Build-Template.md
│   ├── Spec-Template.md
│   ├── Generated-Script-Template.md
│   └── Issue-Template.md
├── scripts/
│   ├── sync-script.sh                 # Auto-sync for multi-device
│   └── setup-vault.sh                 # One-command vault setup (optional)
├── examples/
│   ├── Example-Build-ZTB.md           # Real example: Pi5 router
│   ├── Example-Generated-Script.md    # Real example: health check script
│   └── Example-Workflow.md            # Real example: complete workflow
└── CONTRIBUTING.md                    # How to contribute improvements
```

## Next Steps

1. **Read the guide:** Open `logseq-guide.md` for detailed step-by-step instructions
2. **Review templates:** Check `templates/` folder to see page structure
3. **Study examples:** Look at `examples/` to see real use cases
4. **Start Phase 1:** Begin with Logseq installation and vault setup
5. **Build your own:** Adapt the structure to your projects/builds

## Key Difference from Other Approaches

### Traditional Claude Code Usage
```
User: "Generate a script"
Claude Code: Generates script
User: Stores script somewhere
Next session: User provides spec again
Result: Inconsistent, no memory
```

### With This System
```
User: Writes spec in Logseq, runs Claude Code with link
Claude Code: Reads vault, understands full context
Claude Code: Generates consistent, documented code
Claude Code: Integrates with GitHub, Asana, Slack
Result: Persistent memory, consistent behavior, automated workflow
```

## Community & Support

### Getting Help

- **Questions about setup?** See logseq-guide.md Phase 1-4
- **Questions about using Claude Code?** See logseq-guide.md Phase 5
- **Want to share your vault structure?** Submit a pull request or discussion
- **Found a better way?** Open an issue with improvements

### Contributing

See `CONTRIBUTING.md` for guidelines on:
- Improving the guide
- Adding new templates
- Sharing real-world examples
- Fixing typos/errors

## License

This guide and templates are provided under the MIT License. Use freely, modify for your needs, and share improvements.

## Acknowledgments

This system was designed to solve a specific problem: maintaining consistent Claude Code behavior and complete context across multiple devices and sessions for complex homelab projects. It's built on:

- **Logseq:** Open-source outliner and knowledge base
- **Git:** Version control for multi-device sync
- **Claude Code:** Autonomous code generation and automation
- **MCP Servers:** Integration with external tools

The combination creates a powerful system for infrastructure-as-code and homelab automation.

---

## Quick Reference

### Most Important Links

- **Main Guide:** See `logseq-guide.md` for complete setup instructions
- **Templates:** See `templates/` folder for page structures
- **Examples:** See `examples/` folder for real-world usage
- **Vault Setup:** Phase 1 of logseq-guide.md

### Most Common Commands

```bash
# Clone this repo
git clone https://github.com/YOUR_USERNAME/logseq-claude-code.git

# In your Logseq vault, sync with remote
cd ~/logseq/homelab-vault
git pull origin main          # Get latest changes
git add -A && git commit -m "message" && git push  # Send your changes

# Run Claude Code
claude-code "Read my Logseq vault at ~/logseq/homelab-vault
Read: [[Your-Spec-Page]]
Generate: [What you want]"
```

### Key Concepts

- **Logseq Vault:** Your central knowledge store (markdown files + Logseq)
- **Git Sync:** Keeps vault consistent across devices via GitHub
- **Claude Config:** Pages that define how Claude Code should behave
- **Specifications:** Pages that define what you want built
- **Generated Scripts:** Pages where Claude Code outputs are documented
- **Memory:** Pages that provide context (device inventory, service locations, personal preferences)

---

**Ready to get started?** Open `logseq-guide.md` and begin with Phase 1!

Questions? See the guide's troubleshooting sections or check the examples folder.
