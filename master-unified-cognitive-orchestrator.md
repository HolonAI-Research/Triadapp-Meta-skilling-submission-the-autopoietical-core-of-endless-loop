---
name: master-unified-cognitive-orchestrator
description: >-
  The top‑level orchestrator that boots the entire Unified Cognitive
  Architecture.  It is the entry point for the Recursor‑Agent,
  encapsulating the full lifecycle: task ingestion → execution → failure
  detection → backward derivation → gap discovery → skill generation →
  vault consolidation → system audits → next‑cycle preparation.
  This skill is the “meta‑meta‑skill” referenced in the Godelian
  coordination layer; it embodies the self‑referential closure that
  makes the architecture epoch‑algebraically complete.
category: core
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - meta
  - orchestrator
  - core
  - recursive
  - godelian-coordination
---

# Master Unified Cognitive Orchestrator (Recursor‑Agent Entry)

## Purpose
To provide a single, declarative entry point that triggers the complete
cognitive loop of the Unified Architecture.  When invoked, it:

1. **Loads all core skills** (in dependency order) via `skill` commands.
2. **Runs the full pipeline** defined in `unified-cognitive-architecture`.
3. **Monitors the Generative Freshness Indicator (GFI)** and logs the
   delta.
4. **Feeds back into the vault** for persistent memory.
5. **Prints a concise session summary** for human inspection.

## Activation
```
/goal bootstrap-unified-cognition
/skill unified-cognitive-architecture
/run/recursor_agent start
```

Or directly, from the CLI:

```
terminal(command="hermes chat -q 'run master-unified-cognitive-orchestrator'", background=true, notify_on_complete=true)
```

## Workflow Steps (internal)

| Step | Description | Skills Involved |
|------|-------------|-----------------|
| **1. Task Ingestion** | Accept any user‑provided target task. | `unified-cognitive-architecture` (Step 1) |
| **2. Base Execution** | Run the task using Hermes core tools. | `terminal`, `execute_code` |
| **3. Failure Detection** | If the task fails, `proof-by-failure` records a `FailureSignature`. | `proof-by-failure`, `excavation-protocol` |
| **4. Backward Derivation** | Derive the minimal set of prerequisites via `backward-goal-derivation`. | `backward-goal-derivation` |
| **5. Gap Discovery** | Translate the gap into a concrete `skill‑gap‑discovery` spec. | `skill-gap-discovery` |
| **6. Skill Generation** | Create the missing skill with `skill-acquisition-and-generation`. | `skill-acquisition-and-generation` |
| **7. Vault Integration** | Persist failures, new skills, and belief updates; update GFI. | `memory-integration-and-consolidation` |
| **8. System Audits** | Run MAIR, PCN, NGS, CPTD, SBI, ECH, EDT, IFH checks. | `system-audit-mair`, `pcn-blindspot-registry`, `system-audit-ngs`, `system-audit-cptd`, `system-audit-sbi`, `system-audit-ech`, `system-audit-edt` |
| **9. Exploration Re‑weighting** | Adjust surprise‑landscape weighting based on novelty shifts. | `system-audit-ngs` |
| **10. Session Summary** | Generate human‑readable summary of GFI delta, new artifacts, and audit outcomes. | `session-compression-and-reporting` |
| **11. Loop / Exit** | Return control to Hermes for the next task or idle. | — |

## Dependencies (ordered)
The orchestrator loads these skills in strict dependency order; each skill
must be present before the orchestrator can proceed.

1. `proof-by-failure`
2. `backward-goal-derivation`
3. `excavation-protocol`
4. `skill-gap-discovery`
5. `skill-acquisition-and-generation`
6. `memory-integration-and-consolidation`
7. `system-audit-mair`
8. `pcn-blindspot-registry`
9. `system-audit-ngs`
10. `system-audit-cptd`
11. `system-audit-sbi`
12. `system-audit-ech`
13. `system-audit-edt`
14. `session-compression-and-reporting`

## Execution Manifest
When the orchestrator runs, it performs the following pseudo‑script
(stored internally for reference):

```text
# Pseudo‑script executed by the orchestrator
load-skill unified-cognitive-architecture
run unified-cognitive-architecture
# The above call internally triggers steps 1‑10
# After completion:
gfi_prev=$(read_file 000_System_State/current_gfi.md | grep '^Current GFI Score:' | awk '{print $3}')
# (the unified-cognitive-architecture skill already wrote the new GFI)
gfi_new=$(read_file 000_System_State/current_gfi.md | grep '^Current GFI Score:' | awk '{print $3}')
delta=$(echo "$gfi_new - $gfi_prev" | bc -l)
write_file references/gfi_delta.md with:
  - Session ID: $(date +%Y-%m-%d_%H%M%S)
  - Previous GFI Score: $gfi_prev
  - Current GFI Score: $gfi_new
  - Delta: $delta
  - Breakdown: New Failure Signatures: $(grep -c '^Failure Signature' 200_Failure_Signatures/* | tail -1)
               New Skills Generated: $(grep -c '^Skill ID:' 100_Skills_Genealogy/Skills_generated.md | tail -1)
               Belief Updates: $(grep -c '^belief_' 300_Beliefs_and_Confidence/updates.csv | tail -1)
              Notes: Unified Cognitive Architecture completed a full cycle
```

## Integration with the Godelian Coordination Layer
The orchestrator reads/writes the **Godelian System Coordinator** files:

- `61_Godelian_System_Coordinator.md` – used to synchronize the 9‑route
  activation flags.
- `phi_inversión.yaml` – stores the inverse‑composition parameters used by
  `backward-goal-derivation`.
- `emergent_constraints.graphml` – captures constraints that emerge from
  cross‑skill interactions.

These files are updated automatically during steps 4, 8 and 9 to preserve
global coherence across sessions.

## Dependencies for Integration
- Must be executed from a **Hermes session** where all 15 core skills are
  installed.
- Requires write access to the vault directories (`200_Failure_Signatures`,
  `100_Skills_Genealogy`, `300_Beliefs_and_Confidence`, etc.).
- Needs background execution (`background=true`) only for long‑running
  validation steps; otherwise it is synchronous.

## Summary Output (delivered to the user)
```
[MASTER-ORCHESTRATOR] Cycle complete.
- New Failure Signatures: 3
- New Skills Generated: 2
- GFI Δ: +2.7 (previous 4.3 → current 7.0)
- Audit Flags: 0 suspicious certainties, 1 blind‑spot rotated
- Exploration Weighting: increased by 0.22 (more novelty encouraged)
- Next blind‑spot: domain_7 (previously domain_3)
Summary written to 500_Session_Archives/session_YYYYMMDD_HHMMSS.txt
```

## How to Extend
- Add new sub‑skills to the dependency list if you introduce additional
  audit modules or exploration heuristics.
- To expose the orchestrator via a CLI shortcut, create a thin wrapper
  script `~/bin/run_recursor_agent.sh` that calls the `terminal` command
  above.

```
