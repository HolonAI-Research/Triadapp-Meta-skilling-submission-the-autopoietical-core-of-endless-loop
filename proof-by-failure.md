---
name: proof-by-failure
description: >-
  Core epistemic skill that executes the Proof‑by‑Failure protocol.
  It attempts to compose a target construct from a source primitive,
  records the exact structural failure point, and extracts a failure
  signature (type, depth, missing information, confidence).
category: epistemic
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - proof
  - failure
  - excavation
  - structural-analysis
---

# Proof‑by‑Failure Protocol

## Purpose
To discover irreducibility by systematically trying to build a target
(A) from a set of primitives (B).  The **failure signature** is the
certificate of structural inadequacy.

## Inputs
- `target_spec`: structured description of the desired construct.
- `source_spec`: description of available primitives.

## Process
1. **Composition Attempt** – Use forward excavation to apply source primitives
   toward the target.
2. **Failure Detection** – Identify the first depth at which construction
   cannot proceed.
3. **Signature Extraction** – Record:
   - `failure_type` (e.g., INFORMATION_GAP, DEPTH_BARRIER, …)
   - `failure_depth`
   - `information_missing`
   - `consistency` (across multiple attempts)
   - `confidence` (based on attempts count)
4. **Certificate Generation** – If consistency ≥ 0.70 and attempts ≥ 5,
   output an **Irreducibility Certificate**.

## Outputs
- `failure_signature.md` (written to `200_Failure_Signatures/`)
- `certificate_strength` field for downstream GST (Generative Systems
  Theory) evaluation.

## Integration Points
- Called by `unified-cognitive-architecture` during Step 3.
- Feeds into `skill-gap-discovery` (Step 4) when a gap is identified.
- Results are archived in `200_Failure_Signatures/` for audit.

## Dependencies
None (core primitive).
