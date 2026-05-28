---
name: skill-gap-discovery
description: >-
  Identifies the exact procedural gap that prevents the Recursor-Agent
  from solving a given task.  Using the failure signature from
  `proof-by-failure`, it performs backward derivation (via
  `backward-goal-derivation`) to obtain a concrete skill specification
  that would close the gap.
category: epistemic
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - gap
  - discovery
  - backward-derivation
  - skill-engineering
  - failure-analysis
---

# Skill Gap Discovery

## Purpose
To translate a structural failure (a *failure signature*) into a concrete
procedural requirement: **the minimal skill that must exist for the
Recursor-Agent to succeed**.

## Inputs
- `failure_signature_path`: Path to the JSON failure signature produced by
  `excavation-protocol` (e.g., `200_Failure_Signatures/2026-04-28_001.json`).
- `target_task_id`: Identifier of the original task that triggered the
  failure (for traceability).

## Process
1. **Load Signature** – Read the JSON and validate its schema.
2. **Extract Failure Type** – Determine the `failure_type` (e.g., `INFORMATION_GAP`).
3. **Apply Backward Derivation** – Invoke `backward-goal-derivation`
   with the `information_missing` field as the *target_spec*.
   - Set `max_depth` based on `failure_depth` to limit recursion.
4. **Prune Redundant Prerequisites** – Remove nodes that are not
   essential for addressing the reported gap.
5. **Generate Skill Specification** – Output a structured representation:
   - `name`: Proposed skill name (`<gap_identifier>_v<major>.<minor>`).
   - `description`: One‑sentence summary of what the skill would do.
   - `prerequisites`: List of concrete sub‑steps or sub‑skills.
   - `validation_criteria`: Success metrics (e.g., “closes the reported
     information gap with confidence ≥ 0.70”).
6. **Persist Specification** – Write to `300_GapDiscovery/Specs/<generated_at>.json`
   and add an entry to `100_Skills_Genealogy/Skills_generated.md`.

## Outputs
- `skill_spec.json` containing the specification.
- Updated entry in `skills_generated.md` referencing the new spec.

## Integration Points
- Called by `unified-cognitive-architecture` during Step 5 (Gap‑Filling).
- Provides the concrete input to `skill-acquisition-and-generation`
  when the skill does not already exist.
- Feeds data back into `failures_encountered.md` if the generated skill
  still fails (closed loop).

## Dependencies
- `backward-goal-derivation`
- `proof-by-failure` (to obtain the failure signature)
- `write_file` (to persist the specification)
- `skill_manage` (to register the new skill spec)
