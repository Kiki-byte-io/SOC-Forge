# SOCForge

**Design and Implementation of a SIEM-Based Security Operations and Threat Detection Laboratory Using Wazuh, Custom Detection Rules and Repeatable Validation.**

An isolated, reproducible blue-team lab that collects Windows and Linux endpoint telemetry in Wazuh, implements a small catalogue of custom detections, and validates each one with controlled positive and negative tests.

> **Status: Phase 0 — feasibility and inventory.** Nothing has been deployed yet. No detection results, pass rates or performance figures exist at this stage.

## Core idea

Not "install Wazuh and connect agents", but a repeatable detection-engineering loop:

1. Generate a controlled behaviour
2. Collect telemetry
3. Detect with a Wazuh rule
4. Validate expected vs. actual (positive and negative tests)
5. Investigate with evidence and timelines
6. Improve: tune and document

## Repository layout

| Path | Contents |
|---|---|
| `.claude/skills/` | Claude Code workflows (safe lab changes, learn-while-building) |
| `CLAUDE.md` | Short rules Claude Code loads every session |

More folders (detections, validation, investigations, reports) will be added as each phase begins.

## Safety

Lab-only, isolated networking. No secrets, credentials or raw unsanitized logs are committed. See `.gitignore`.

## Author

Khushi Kothari — B.Sc. Computer Science (Honours) major project.
