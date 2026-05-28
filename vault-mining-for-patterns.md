---
name: vault-mining-for-patterns
description: >-
  Scans the vault for recurring structural motifs across failure
  signatures, skill graphs, and belief updates.  It extracts
  statistically significant patterns and outputs them as
  `Pattern_Catalog.md` for downstream consumption.
category: analysis
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - analysis
  - vault
  - pattern-discovery
  - statistics
---

# Vault Mining for Patterns

## Purpose
To automatically discover reusable structural motifs in the persistent
memory vault that are not obvious from a single session.  Patterns are
extracted from three sources:

1. **Failure Signatures** – recurring failure types, depths, and missing
   information.
2. **Skill Dependency Graphs** – frequent prerequisite clusters.
3. **Belief Update Histories** – repeated confidence shifts on specific
   beliefs.

The resulting catalog feeds `surprise-landscape-navigation` and
`blindspot-negotiation`.

## Inputs
- `failure_signatures_dir`: Path to `200_Failure_Signatures/` (directory of
  JSON signatures).
- `skill_graph_path`: Path to `skill_graph.md` (current dependency graph).
- `belief_updates_path`: Path to `300_Beliefs_and_Confidence/updates.csv`.

## Process
1. **Collect Data** – Load all failure signatures, parse `skill_graph.md`,
   and parse belief update records.
2. **Tokenize & Normalize** – Convert each element into a normalized token
   set (e.g., `INFORMATION_GAP` → `gap`, `DEPTH_BARRIER` → `depth_barrier`).
3. **Frequency Counting** – Use a sliding‑window n‑gram counter to find
   recurring token sequences of length 2‑4.
4. **Statistical Filtering** – Apply a chi‑square test (p < 0.05) to keep
   only patterns that are significantly over‑represented compared to the
   background corpus.
5. **Cluster Similar Patterns** – Group patterns with Jaccard similarity
   ≥ 0.8; keep the representative with highest chi‑square.
6. **Pattern Catalog Generation** – Write `Pattern_Catalog.md` in the
   vault root:
   - Each pattern entry includes:
     * `signature`: normalized token sequence
     * `frequency`: raw count
     * `p_value`: statistical significance
     * `examples`: up to 3 raw excerpts from signatures / graphs /
       belief updates
7. **Export to JSON** – Also generate `pattern_catalog.json` for machine
   consumption.
8. **Integration Hook** – The catalog is automatically read by
   `surprise-landscape-navigation` (to weight exploration) and
   `blindspot-negotiation` (to rotate domains).

## Outputs
- `Pattern_Catalog.md` (human‑readable)
- `pattern_catalog.json` (machine‑readable)
- Updated `pattern_catalog_index.md` (index of discovered motifs)

## Integration Points
- Called by `memory-integration-and-consolidation` after a new
  session’s artifacts are written.
- Consumes outputs of `excavation-protocol` (failure signatures) and
  `skill-acquisition-and-generation` (new skill graphs).
- Provides input to `surprise-landscape-navigation` (exploration
  weighting) and `blindspot-negotiation` (blind‑spot rotation).

## Dependencies
- `read_file`
- `search_files` (to locate all JSON/csv files)
- `execute_code` (optional chi‑square calculator script)
- `write_file` (to produce `Pattern_Catalog.md` and `.json`)
- `terminal` (optional for heavy statistical loops)
