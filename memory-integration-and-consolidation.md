---
name: memory-integration-and-consolidation
description: >-
  Synchronizes newly generated skills, failure signatures, and belief updates
  with the persistent Obsidian vault.  It ensures that transient session data
  becomes permanent, runs anti‑crystallization checks (NGS), detects blind‑
  spots (PCN), and flags suspicious certainty (MAIR).  The output is a
  consolidated snapshot that feeds the GFI calculator.
category: memory
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - memory
  - consolidation
  - vault-sync
  - anti-crystallization
  - blind-spot-rotation
  - certainty-audit
---

# Memory Integration & Consolidation

## Purpose
To transform transient learning artefacts (new failure signatures, newly
generated skills, updated belief states) into durable, version‑controlled
entries inside the Obsidian vault.  The process also runs integrity
checks that prevent premature convergence and preserve generative freshness.

## Inputs
- `new_failures_path`: Path to a file listing newly‑encountered failure
  signatures (e.g., `200_Failure_Signatures/summary.md`).
- `new_skills_dir`: Directory containing directories of newly‑created
  skills (each a skill folder with `SKILL.md`).
- `belief_updates_path`: Path to a CSV/JSON listing updated beliefs with
  confidence deltas.
- `gfi_calc_path`: Path to the GFI delta file to be updated.

## Process
1. **Vault Write‑Back**  
   - Append new failure signatures to `200_Failure_Signatures/`.  
   - Copy each new skill directory into `100_Skills_Genealogy/`.  
   - Insert belief updates into `300_Beliefs_and_Confidence/`.  
2. **Anti‑Crystallization Scan (NGS)**  
   - Parse belief updates for any belief whose confidence > 0.90 and has
     not been revised in the last 3 sessions.  
   - Flag such beliefs for `NGS` perturbation (see `surprise_landscape.md`).  
3. **Blind‑Spot Rotation (PCN)**  
   - Determine which knowledge domains have > 2 new skills in the last
     session.  
   - If so, rotate the blind‑spot assignment in `400_Knowledge_Topology/
     blindspot_assignments.md` to force future exploration of a different
     domain.  
4. **Certainty Audit (MAIR)**  
   - Scan newly added skills for any that contain a certainty claim
     (keywords: “ensures”, “guarantees”, “always”).  
   - If found, write a warning to `600_Session_Archives/suspicious_certainty.log`.  
5. **GFI Update**  
   - Read the current GFI score from `000_System_State/current_gfi.md`.  
   - Compute the delta contributed by the new artifacts (each new skill
     = +0.4, each new failure signature = +0.2, each belief update = +0.1).  
   - Write the new GFI score back to `current_gfi.md`.  
6. **Snapshot**  
   - Create a compressed snapshot of the entire vault state and store it
     under `500_Session_Archives/snapshots/YYYY-MM-DD_HHMMSS.zip`.  

## Outputs
- Updated vault entries (failure signatures, new skills, belief updates).  
- `gfi_delta.md` refreshed with the new total GFI.  
- Potential flags/warnings in `surprise_landscape.md` and
  `suspicious_certainty.log`.  
- A versioned snapshot (`zip`) for rollback.

## Integration Points
- Called by `unified-cognitive-architecture` after a successful skill
  generation (Step 7) and after any belief update.
- Consumes data produced by `excavation-protocol`, `skill-acquisition-and-
  generation`, and belief‑tracking modules.
- Feeds the result into the GFI calculator (`gfi_delta.md`).

## Dependencies
- `write_file` (vault writes)
- `execute_code` (optional script runner for NGS/PCN heuristics)
- `read_file` / `search_files` (to locate existing entries)
- `terminal` (background processing for snapshot compression)
- `process` (to manage background snapshot job)
