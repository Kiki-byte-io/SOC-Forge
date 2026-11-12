# SOCForge — Claude Code rules

Wazuh-based SIEM detection-engineering lab (individual B.Sc. Hons major project).
Full brief: `docs/SOCForge_Claude_Code_Handoff.md` — read it before substantial work. If it conflicts with the proposal or these rules, stop and ask.

## Current phase
**Phase 2 — Wazuh deployment.** Phases 0–1 done (`docs/phase-0/`, `docs/architecture/`, `docs/decisions/`). Every host/VM/network change goes through the `safe-lab-change` skill with explicit owner approval, one step at a time.

## How to work with the owner
- The owner is learning. Follow: Explain → Plan → Owner approves → Implement → Test → Review → Teach-back → Commit.
- Use the `learn-while-building` skill for features and the `safe-lab-change` skill before anything that touches the host, VMs, disks, network, firewall, audit policy or endpoints.
- One small milestone at a time. Never span multiple phases in one go.
- Show exact commands before running anything non-trivial. Admin/elevated commands need explicit approval.

## Hard rules
- HoneyWatch / Cowrie assets (VMs, disks, repos, DBs, credentials, adapters) are OFF-LIMITS.
- Never start/stop/delete/move/reconfigure VMs or disks without explicit approval.
- C: drive is low on space — never place VM disks or large data there without checking.
- Never commit secrets, passwords, tokens, keys or raw unsanitized logs. Only `evidence/sanitized/` is committable.
- Don't invent event IDs, field names, Wazuh rule syntax or ATT&CK mappings — verify from official docs or observed events and cite sources (title, URL, access date).
- Never claim production-grade capability, full ATT&CK coverage or universal accuracy. State test-set size and limits.
- Lab-only targets. No public IPs, no destructive payloads.

## Validation statuses
`NOT_RUN`, `PASS`, `FAIL`, `PARTIAL`, `INCONCLUSIVE`
