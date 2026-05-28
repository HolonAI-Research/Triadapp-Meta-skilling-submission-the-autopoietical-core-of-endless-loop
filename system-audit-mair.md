---
name: system-audit-mair
description: >-
  Monitors the knowledge base for suspiciously high certainty claims.
  When a skill or belief asserts something with near‑certainty, it flags
  the item for review, logs a warning, and may trigger an anti‑crystallisation
  perturbation.
category: audit
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - audit
  - certainty
  - crystallization
  - mair
---

# System Audit – MAIR (Maximal Assertion Influence Review)

## Purpose
To detect and neutralise suspiciously certain assertions that could lead to
premature crystallization of knowledge.  The audit runs after each
session and scans newly added skills, beliefs, and failure signatures
for certainty markers.

## Inputs
- `new_skills_dir`: Directory of skills added in the current session.
- `new_beliefs_path`: Path to `300_Beliefs_and_Confidence/updates.csv`.
- `new_failures_dir`: Path to newly written failure signatures.

## Process
1. **Scan Skills** – For each new skill’s `SKILL.md`, run a regex
   `\\b(ensures|guarantees|always|must|will)\\b` on the description.
2. **Scan Beliefs** – Check confidence > 0.90 and recent revision age
   (older than 3 sessions).
3. **Flag Items** – Write flagged items to
   `600_Session_Archives/suspicious_certainty.log` with timestamp,
   item ID, and excerpt.
4. **Trigger Anti‑Crystallisation** – If > 2 items are flagged,
   invoke `blindspot-negotiation` to rotate the blind‑spot and force
   exploration of a different domain.

## Outputs
- `suspicious_certainty.log` appended with new flags.
- Possible rotation triggered via call to `blindspot-negotiation`.

## Dependencies
- `read_file`
- `write_file`
- `execute_code` (optional call to `blindspot-negotiation` via
  `terminal(background=true)`)
- `terminal` (background execution if auto‑rotation enabled)