---
name: skill-acquisition-and-generation
description: >-
  Generates a brand‑new skill that fills the exact gap identified by
  `skill-gap-discovery`.  It creates the minimal procedural package,
  validates it against the original failure signature, and registers it
  in the vault.  Successful generation yields a new skill entry and
  updates the `skills_generated.md` index.
category: procedural
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - acquisition
  - generation
  - skill-creation
  - automation
---

# Skill Acquisition & Generation

## Purpose
To turn a discovered *skill gap* (output of `skill-gap-discovery`) into a
working skill module that can be dropped into the Recursor‑Agent’s
registry.  The process is fully procedural: generate → test → register.

## Inputs
- `skill_spec_path`: Path to the JSON specification produced by
  `skill-gap-discovery` (e.g., `300_GapDiscovery/Specs/2026-04-28_004.json`).
- `max_attempts`: Maximum validation runs (default: 5).
- `validation_timeout`: Seconds per validation (default: 30).

## Process
1. **Load Specification** – Parse `skill_spec.json` for:
   - `name`
   - `description`
   - `prerequisites`
   - `validation_criteria`
2. **Template Generation** – Using the specification, instantiate a
   directory structure under `~/.hermes/skills/<name>/`:
   - `SKILL.md` (boilerplate with front‑matter copied from template)
   - `scripts/validate.py` – runs the skill’s self‑test
   - `references/example.md` – placeholder for future documentation
3. **Validation Execution** – Run `scripts/validate.py` up to
   `max_attempts` times, each with a fresh timeout.
   - The script must exit with code 0 on success; any non‑zero exit
     indicates failure.
   - On success, capture `GFI_delta` contribution (adds a numeric
     contribution to the overall GFI score).
4. **Error Handling** – If all attempts fail:
   - Record the failure signature in `200_Failure_Signatures/`
     (reuse the pattern from `excavation-protocol`).
   - Return `skill_ready=false`.
5. **Registration** – On success:
   - Add the skill name to `100_Skills_Genealogy/Skills_generated.md`
     (incremental entry).
   - Update `skill_graph.md` (dependency graph) with edges from the
     specification’s `prerequisites`.
   - Append an entry to `300_GapDiscovery/Generated_Skills.md`.

## Outputs
- `SKILL.md` and supporting files in `<skill_dir>/`
- Validation log (`validation.log`)
- Registration entry in `Skills_generated.md`
- Update to `skill_graph.md`

## Integration Points
- Called by `unified-cognitive-architecture` during Step 6
  (Skill Validation).
- Feeds into `memory-integration-and-consolidation` once the skill is
  successfully registered.
- Provides the new skill to `skill-genealogy-tracking` for future
  mutation cycles.

## Dependencies
- `write_file`
- `execute_code` (to run `validate.py`)
- `skill_manage` (to register the new skill if it passes validation)
- `read_file` (to load the specification)
- `terminal` (optional background validation runner)
