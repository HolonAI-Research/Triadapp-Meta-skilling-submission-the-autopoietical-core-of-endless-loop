---
name: backward-goal-derivation
description: >-
  Generates prerequisite trees by applying inverse reasoning (Phi_INVERSIÓN) to a target goal.
  It backtracks from the desired outcome to minimal necessary prerequisites and detects convergence points.
category: epistemic
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - backward
  - derivation
  - goal-inversion
  - convergence
  - phi-inversión
---

# Backward Goal Derivation

## Purpose
To discover the minimal set of intermediate states that must exist for a target goal to be achievable.
Uses inverse composition (Phi_INVERSIÓN) to trace prerequisites backward from the goal toward foundational primitives.

## Inputs
- `target_spec`: Structured description of the desired final state.
- `max_depth`: Maximum recursion depth (default: 7).

## Process
1. **Inverse Expansion** – Recursively apply inverse operations to decompose the target.
2. **Prerequisite Collection** – Gather each intermediate state as a prerequisite node.
3. **Convergence Detection** – Look for overlapping prerequisites across branches (phi convergence).
4. **Pruning** – Remove redundant nodes that do not contribute to convergence.
5. **Output Generation** – Produce:
   - `prerequisite_tree.md` (list of nodes, dependencies)
   - `convergence_report.md` (where branches meet, depth, confidence)

## Outputs
- `prerequisite_tree.md` stored in `300_Prerequisites/`
- `convergence_report.md` stored in `300_Prerequisites/convergence/`

## Integration Points
- Called by `unified-cognitive-architecture` during Step 4 (Backward Derivation).
- Feeds nodes into `skill-gap-discovery` to identify which prerequisites correspond to missing skills.

## Dependencies
- `proof-by-failure` (to evaluate feasibility of each prerequisite)
- `excavation-protocol` (to record any failure encountered during inverse expansion)
