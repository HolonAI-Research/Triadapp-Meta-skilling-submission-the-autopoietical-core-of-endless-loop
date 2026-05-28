---
name: system-audit-ngs
description: >-
  Monitors the Surprise Landscape for statistically significant changes,
  flags emerging novelty peaks, and adjusts the Exploration Weighting
  used by `surprise-landscape-navigation`.  Prevents novelty stagnation
  and forces continued asymmetric exploration.
category: audit
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - audit
  - novelty
  - surprise
  - landscape
  - exploration-weighting
---

# System Audit – NGS (Surprise Landscape)

## Purpose
To keep the generative process dynamically sensitive to new sources of
surprise.  When the landscape of unexpected patterns ceases to evolve,
the system risks converging on a narrow set of solutions.  This audit
detects such stagnation and forces a re‑weighting of the exploration
budget.

## Inputs
- `surprise_landscape_path`: `400_Knowledge_Topology/surprise_landscape.md`
- `ngs_history_path`: `500_Session_Archives/ngs_history.json` (stores prior
  surprise scores)
- `threshold`: Minimum change in average surprise score to trigger a
  re‑weighting (default 0.15)

## Process
1. **Load Current Landscape** – Parse the markdown/surprise score.
2. **Load Historical Scores** – Read the last N entries from
   `ngs_history.json`.
3. **Compute Delta** – Compare current average surprise to the historical
   mean; if `abs(delta) >= threshold`, record a **NoveltyShift** event.
4. **Update Exploration Weighting** – Write a new
   `exploration_weighting.md` entry:
   - `weight_adjustment`: `<delta> * 1.5` (positive = increase, negative = decrease)
   - `rationale`: short explanation of why the shift occurred.
5. **Persist History** – Append the current surprise score to
   `ngs_history.json` for future delta calculations.
6. **Log Event** – Add an entry to
   `600_Session_Archives/novelty_shifts.log` with timestamp, old score,
   new score, and `weight_adjustment`.

## Outputs
- Updated `exploration_weighting.md`.
- Appended entry in `novelty_shifts.log`.
- Updated `ngs_history.json`.

## Dependencies
- `read_file`
- `write_file`
- `execute_code` (optional for statistical delta calculations)
- `terminal` (optional background logging)