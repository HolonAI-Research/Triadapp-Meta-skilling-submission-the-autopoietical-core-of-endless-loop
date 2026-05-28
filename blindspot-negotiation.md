---
name: blindspot-negotiation
description: >-
  Dynamically rotates the system’s “blind‑spot” assignments based on
  activity patterns detected in the vault.  When a particular knowledge
  domain accumulates too many new skills or failure signatures, the
  negotiator re‑prioritises the domain to force exploration of an
  under‑examined area.  This prevents premature crystallisation of
  expertise and maintains productive asymmetry.
category: knowledge-management
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - blindspot
  - negotiation
  - domain-rotation
  - asymmetry
  - pcn
---

# Blindspot Negotiation

## Purpose
To maintain a healthy distribution of exploratory focus across the
knowledge domains captured in the vault.  When a domain becomes
over‑populated (e.g., many new skills or failure signatures), the
negotiator signals the system to treat a different domain as the next
primary blind‑spot, ensuring continued asymmetry and anti‑crystallisation.

## Inputs
- `domain_activity_log`: JSON file tracking count of new artifacts per
  domain (e.g., `400_Knowledge_Topology/domain_activity.json`).
- `blindspot_assignments_path`: Path to `400_Knowledge_Topology/blindspot_assignments.md`.
- `threshold`: Number of new artifacts that triggers a rotation
  (default: 5).

## Process
1. **Load Activity Log** – Read per‑domain artifact counts.
2. **Identify Over‑populated Domain** – Find any domain where count ≥ `threshold`.
3. **Select New Blindspot** – Choose the domain with the lowest current
   count that is not currently a blindspot.
4. **Update Assignments** – Write the new mapping to
   `blindspot_assignments.md` and increment a rotation counter.
5. **Log Rotation** – Append an entry to `600_Session_Archives/blindspot_rotations.log`
   with timestamp, old and new blindspot IDs, and counts.
6. **Trigger Anti‑Crystallization** – If a domain has been a blindspot for
   > 10 rotations without change, force a temporary suspension of new
   artifact creation in that domain (gate flag in `200_Failure_Signatures/gate.json`).

## Outputs
- Updated `blindspot_assignments.md`.
- Rotation entry in `blindspot_rotations.log`.
- Optional gate flag (`gate.json`) to pause new artifact ingestion from
  the over‑populated domain.

## Integration Points
- Invoked by `memory-integration-and-consolidation` after a new session’s
  artifacts are written.
- Consumes `domain_activity_log` produced by `vault-mining-for-patterns`
  (or directly by the vault‑watcher).
- Feeds the new blindspot into `surprise-landscape-navigation` (to bias
  exploration weighting) and into `blindspot-negotiation` (self‑reference).

## Dependencies
- `read_file`
- `write_file`
- `execute_code` (optional script to recompute counts)
- `terminal` (for logging rotations)
