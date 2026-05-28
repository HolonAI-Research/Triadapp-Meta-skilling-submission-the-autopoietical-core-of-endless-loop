---
name: meta-performance
description: >-
  Tracks and records quantitative performance metrics of the Unified
  Cognitive Architecture, primarily the Generative Freshness Indicator
  (GFI), skill generation rate, and novelty weight adjustments.
category: performance
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - performance
  - metrics
  - gfi
  - novelty
  - skill-rate
---

# Meta‑Performance Tracker

## Purpose
To capture and persist performance metrics that describe how the
Recursor‑Agent improves itself over time.  Metrics are written to the
vault under `000_System_State/` and also logged in
`500_Session_Archives/meta_performance.log`.

## Key Metrics
1. **GFI Delta** – Change in Generative Freshness Indicator per session.
2. **Skill Generation Rate** – Number of new skills created per 100 cycles.
3. **Novelty‑Weight Adjustment** – Magnitude of weight change applied to the
   surprise‑landscape exploration weighting.
4. **Audit Flag Count** – Number of suspicious certainty flags raised in the
   last cycle.
5. **Blind‑Spot Rotations** – Count of blind‑spot rotations performed.

## Process
1. **Read Current Metrics** – Pull values from:
   - `000_System_State/current_gfi.md` (GFI score)
   - `500_Session_Archives/meta_performance.log` (append-only log)
   - `400_Knowledge_Topology/blindspot_rotations.log` (rotation count)
2. **Compute Deltas** – Compare against values stored in
   `000_System_State/previous_metrics.json`.
3. **Update JSON Store** – Write the new metrics (including deltas) to
   `000_System_State/metrics.json`.
4. **Log Entry** – Append a human‑readable line to
   `500_Session_Archives/meta_performance.log` with timestamp and raw
   values.
5. **Persist** – Ensure the updated JSON and log entries are written to the
   vault.

## Integration Points
- Triggered automatically by `master_unified_cognitive_orchestrator`
   after each cycle completes.
- Consumes data from:
  - `gfi_delta.md`
  - `500_Session_Archives/summary_*.txt` (parsed for new skills count)
  - `600_Session_Archives/blindspot_rotations.log`
  - `600_Session_Archives/suspicious_certainty.log`
- Outputs to:
  - `000_System_State/metrics.json`
  - `500_Session_Archives/meta_performance.log`

## Dependencies
- `read_file`
- `write_file`
- `execute_code` (optional for delta calculations)
- `terminal` (optional for async logging)

## Configuration
All thresholds can be overridden via Hermes config under the
`meta_performance` section:

| Config Key | Default | Description |
|-----------|---------|-------------|
| `meta_performance.gfi_weight` | `1.0` | Weight applied to GFI delta when computing overall performance score. |
| `meta_performance.skill_rate_window` | `100` | Number of cycles over which to average skill generation. |
| `meta_performance.novelty_weight_factor` | `0.5` | Multiplier for novelty‑weight change impact. |
| `meta_performance.log_retention_days` | `30` | How long to keep raw log entries before pruning. |

--- 