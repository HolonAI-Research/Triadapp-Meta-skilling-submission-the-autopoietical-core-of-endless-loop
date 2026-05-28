---
name: unified-cognitive-architecture
description: >-
  Meta‑skill that boots the complete Recursor‑Agent system. 
  It orchestrates the 9‑route integration, vault handling and failure‑signature
  extraction.  This skill is the entry point to the full unified architecture.
category: core
version: 0.1.0
author: you / omni-architect
tags:
  - core
  - cognitive
  - unification
  - recursion
  - flash-of-genius
---

# Unified Cognitive Architecture – Recursor Agent Core

## Activation
/run/recursor_agent
/goal <<< bootstrap unified cognition >>>

## Main Workflow
1. **Task Ingestion** – Accept any user‑provided target.
2. **Hermes Execution** – Run the task in the base Hermes loop.
3. **Failure Detection** – If the task fails, generate a FailureSignature
   using proof‑by‑failure (excavation protocol).
4. **Backward Derivation** – Apply backward_attractor to the failure
   signature to obtain the *skill gap*.
5. **Gap‑Filling** – Invoke skill‑gap‑discovery → generate the minimal
   skill needed to close the gap.
6. **Skill Validation** – Execute the generated skill on the original task.
   - If successful, add it to `100_Skills_Genealogy/`.
   - If not, run skill‑mutation‑scaffolding and retry.
7. **Vault Update** – Persist:
   - FailureSignature → `200_Failure_Signatures/`
   - New/Modified Skill → `100_Skills_Genealogy/`
   - Updated GFI metrics → `000_System_State/current_gfi.md`
8. **System Auditing** – Evaluate the new state against the 9 routes:
   - MAIR: check for suspicious certainty.
   - PCN: rotate blind‑spot if a domain had > 2 new skills.
   - NGS: compute surprise‑gradient and update `exploration_targets.md`.
   - CPTD: log any new contradictions → `400_Knowledge_Topology/contradiction_graph.md`.
   - SBI: run symmetry‑breaker on the new skill’s abstract graph.
   - ECH: detect new seams → `400_Knowledge_Topology/seam_conflicts.md`.
   - EDT: spawn temporal‑dissonance agents for parallel validation.
   - IFH: queue any human‑feedback needed.
9. **Session Compression** – Run `session-compression-and-reporting` to
   produce a human‑readable summary (saved to `500_Session_Archives/`).
10. **Loop / Exit** – Return control to Hermes; the agent will attempt a
    new task or idle until the next user input.

## Dependencies (skills it loads, ordered)
- `proof-by-failure`
- `backward-goal-derivation`
- `excavation-protocol`
- `skill-gap-discovery`
- `skill-acquisition-and-generation`
- `memory-integration-and-consolidation`
- `vault-synchronization`
- `system-audit-mair`
- `system-audit-pcn`
- `system-audit-ngs`
- `system-audit-cptd`
- `system-audit-sbi`
- `system-audit-ech`
- `system-audit-edt`
- `session-compression-and-reporting`

## Error Handling
* If any sub‑skill returns an error, the orchestrator will:
  1. Record the error signature.
  2. Apply `skill-mutation-scaffolding` on the failing sub‑skill.
  3. Retry up to `max_attempts=3` with exponential back‑off.
  4. If still failing, flag the gap for manual review and store it in
     `200_Failure_Signatures/` for future analysis.

## Documentation in Vault
All generated artefacts are added to the vault schema described in the
`UNIFIED_COGNITIVE_ORCHESTRATION.md` (see `000_System_State/…`).