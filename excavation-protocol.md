---
name: excavation-protocol
description: >-
  Implements the core Excavation workflow used by Proof‑by‑Failure and
  other structural analyses.  It orchestrates attempts, records
  failure signatures, and persists them to the vault for downstream
  consumption.
category: epistemic
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - excavation
  - protocol
  - failure-signature
  - structural-analysis
--- 

# Excavation Protocol

## Overview
The Excavation Protocol is the procedural backbone for any operation that
tries to construct a target from primitives and records exactly where
the construction breaks down.  It is reusable by Proof‑by‑Failure,
Backward Goal Derivation, and the Recursor‑Agent’s gap‑finding loop.

## Inputs
- `target_spec`: Structured description of the desired construct.
- `source_spec`: Description of available primitives or previously
  generated skills.
- `max_attempts`: Maximum number of composition attempts (default: 50).
- `attempt_id`: Identifier for provenance (defaults to auto‑generated UUID).

## Process
1. **Attempt Loop**  
   ```
   for i in range(max_attempts):
       result = attempt_composition(target_spec, source_spec)
       if result.success:
           return result
       else:
           signature = extract_failure_signature(result.failure)
           persist_signature(signature, attempt_id, i)
   ```
2. **Failure Signature Extraction**  
   - Captures `failure_type`, `failure_depth`, `information_missing`,
     `consistency`, and `confidence`.
   - Normalises the signature into a canonical JSON form.
3. **Persistence**  
   - Writes the signature to `200_Failure_Signatures/<attempt_id>.json`.
   - Updates `failures_encountered.md` (see `failures_encountered.md` reference).
4. **Consistency Check**  
   - If the same signature appears in ≥ 3 independent attempts,
     mark it as **Structural** and elevate its `confidence` to ≥ 0.80.
5. **Termination**  
   - Return `success=False` and the best‑recorded signature.

## Outputs
- `failure_signature.json` (in `200_Failure_Signatures/`)
- Updated `failures_encountered.md`
- Optional `debug_trace.log` (useful for post‑mortem analysis)

## Integration Points
- Invoked by `proof-by-failure` (Step 2) when a composition attempt fails.
- Re‑used by `unified-cognitive-architecture` during Gap‑Filling
  (Step 5) to validate that a generated skill resolves the recorded
  failure.
- Consumed by `skill-gap-discovery` to prioritise which missing capability
  to target next.

## Dependencies
- `execute_code` (to run composition scripts if external logic required)
- `memory` (to read/write failure signatures)
- `write_file` (to persist signatures)
- `read_file` (to retrieve prior signatures for consistency checks)
