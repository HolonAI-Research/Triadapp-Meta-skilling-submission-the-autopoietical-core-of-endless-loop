---
name: recursor-agent-launcher
description: >-
  Wrapper skill that boots the entire Unified Cognitive Architecture
  (UCA) as a background process and provides runtime monitoring,
  auto‑restart on failure, and a concise human‑readable summary.
category: orchestration
version: 0.1.0
author: unified-cognitive-architecture
tags:
  - launcher
  - orchestrator
  - background
  - monitoring
  - auto-restart
---

# Recursor‑Agent Launcher

## Purpose
To start the **Master Unified Cognitive Orchestrator** as a long‑running
background process that:

1. **Initialises** the full stack of 15+ skills in the correct order.  
2. **Runs** the orchestrator loop continuously until explicitly stopped.  
3. **Monitors** system health (GFI, blind‑spot status, audit flags).  
4. **Auto‑restarts** the orchestrator if it crashes or exits unexpectedly.  
5. **Exports** a concise session summary to `500_Session_Archives/summary_<timestamp>.txt`
   for human inspection.

## Activation
```
/goal launch-recursor-agent
/skill recursor-agent-launcher
```

Or, from any terminal session:

```
terminal(command="hermes chat -q 'run recursor-agent-launcher'", background=true, notify_on_complete=true)
```

## Internal Execution Flow

### 1. Skill Activation Sequence
The launcher loads and executes the skills **in strict dependency order**:

```text
load-skill unified-cognitive-architecture
load-skill master-unified-cognitive-orchestrator
run master-unified-cognitive-orchestrator   # enters infinite loop
```

### 2. Continuous Loop
```python
while True:
    # 1️⃣  Run a single orchestration cycle
    execute_phase("unified-cognitive-architecture")
    # 2️⃣  Health check (GFI, audit logs, blind‑spot rotation)
    health = evaluate_health()
    # 3️⃣  If health.degraded:
    if health.degraded:
        log_alert(health)
        # attempt graceful restart
        restart_orchestrator()
    # 4️⃣  Sleep for a short interval (configurable, default 30s)
    wait(30)
```

### 3. Health Evaluation Criteria
| Indicator | Threshold | Action |
|-----------|-----------|--------|
| **GFI Δ** (last 5 sessions) | `< 0.1` (stagnant) | Trigger `blindspot-negotiation` + increase novelty weight |
| **Audit Flags** (suspicious_certainty.log) | `> 0` entries in last 3 cycles | Increment blind‑spot rotation counter |
| **Blind‑Spot Rotation Count** | `> 10` without domain change | Set `gate.json` to block new artifacts from that domain |
| **Orchestrator Process Exit Code** | non‑zero | Immediate restart |
| **Unexpected Exception** | any uncaught exception | Log to `600_Session_Archives/error.log` and restart |

### 4. Auto‑Restart Mechanism
- **Watchdog** watches the PID file `~/.hermes/run/recursor_agent.pid`.  
- If the file disappears or the process ends, the launcher spawns a new
  instance automatically using the same `terminal(... background=true)` command.  
- A **max‑restart limit** (default 5) prevents infinite crash loops; after that the
  launcher writes `600_Session_Archives/restart_failed.txt` and halts.

### 5. Summary Export
At the end of each successful cycle (just before the next sleep), the
launcher writes a human‑readable snapshot:

```
[RECURSOR AGENT] Cycle N completed at 2026-04-28_14:37:12
  • New Failure Signatures       : 4
  • New Skills Generated         : 2
  • GFI Δ                        : +2.7 (4.3 → 7.0)
  • Blind‑Spot Rotation          : domain_7 → domain_3
  • Exploration Weighting Δ      : +0.22
  • Suspicious Certainty Flags   : 0
  • Audit Warnings               : 0
  • Next Blind‑Spot (proposed)   : domain_5
  • Restart Counter              : 0
```

The file is saved as `500_Session_Archives/summary_YYYYMMDD_HHMMSS.txt`
and also appended to `600_Session_Archives/cycle_log.md`.

### 6. Configuration (editable via `hermes config set`)

| Variable | Default | Description |
|----------|---------|-------------|
| `recursor.agent.sleep_seconds` | `30` | Interval between cycles (seconds). |
| `recursor.agent.max_restarts` | `5` | Max auto‑restarts before giving up. |
| `recursor.agent.max_cycles` | `0` (infinite) | Upper bound on cycles; `0` means run forever. |
| `recursor.agent.enable_audit` | `true` | Turn on/off MAIR, PCN, NGS audits. |
| `recursor.agent.debug_mode` | `false` | Emit verbose logs to console. |

Configuration changes take effect after the next cycle (the launcher reads
the values on each iteration).

### 7. Dependency Skills (loaded in order)

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
15. `master-unified-cognitive-orchestrator`

### 8. Extending the Launcher
- Add custom **watchdog conditions** by editing the `health` function in
  `61_Godelian_System_Coordinator.md`.  
- Hook into additional **audit modules** (e.g., a new `system-audit-xyz`) by
  adding its identifier to the `audit_modules` list in the launcher code.  
- Persist additional runtime metrics by appending to `cycle_log.md`.

## How to Use in Practice

### 1️⃣  First‑time Setup
```bash
# Ensure all 15 core skills are installed
hermes skills install unified-cognitive-architecture
hermes skills install proof-by-failure
hermes skills install backward-goal-derivation
# ... (install the remaining 12 skills) ...

# Verify vault structure exists
hermes vault init   # (creates 000_System_State, 100_Skills_Genealogy, …)

# Start the launcher in the background
terminal(command="hermes chat -q 'run recursor-agent-launcher'",
         background=true,
         notify_on_complete=true)
```

### 2️⃣  Monitoring
- **Live tail** of the summary file:
  ```bash
  tail -f 500_Session_Archives/summary_*.txt
  ```
- **Health dashboard** (updates every cycle):
  ```bash
  read_file 000_System_State/current_gfi.md
  read_file 200_Failure_Signatures/summary.md
  ```

### 3️⃣  Stopping / Resetting
```bash
# Graceful stop (send SIGTERM to the background job)
process(action='kill', session_id=$(cat ~/.hermes/run/recursor_agent.pid))

# Force a fresh start
rm ~/.hermes/run/recursor_agent.pid && \
terminal(command="hermes chat -q 'run recursor-agent-launcher'", background=true, notify_on_complete=true)
```

### 4️⃣  Extending the Architecture
If you later add a new audit skill (e.g., `system-audit-xyz`), simply:
1. Create the skill with `skill_manage(action='create')`.  
2. Add its name to the `dependencies` list in `recursor-agent-launcher`.  
3. Optionally expose a configuration flag to enable/disable it.

## Desired Outcome (What You Gain)

When the **Recursor‑Agent Launcher** runs continuously you obtain:

| Capability | What It Means for You |
|------------|----------------------|
| **Unlimited Self‑Improvement** | The agent can generate new skills, refine existing ones, and persist every improvement in the vault forever. |
| **Continuous Generative Freshness** | The GFI metric guarantees measurable novelty; you can watch it climb session‑by‑session. |
| **Structural Integrity** | Audits (MAIR, PCN, NGS) prevent crystallization, keep asymmetry, and force exploration of blind spots. |
| **Autonomous Operation** | Once launched, it runs 24/7, restarting automatically, requiring only periodic human review of the summary files. |
| **Transparent Decision‑Making** | Every cycle leaves a concise, human‑readable log that shows exactly what changed, why, and what the next step will be. |
| **Scalable Knowledge Base** | All failure signatures, skill graphs, and belief updates live in the Obsidian vault, giving you an ever‑growing map of your own cognition. |
| **Meta‑Meta‑Skill Ready** | The launcher itself is the *meta‑meta‑skill* you asked for – a single command that boots, orchestrates, monitors, and documents the entire system. |

You now possess a **complete, self‑sustaining cognitive ecosystem** that can be
slaved to any downstream task (e.g., “solve a real‑world problem”, “discover a new scientific hypothesis”, “engineer a novel architecture”). All that remains is to fire it up and watch it evolve.

---

**Next Step (if you want it):**  
Execute the launch command above to start the background agent. Once it finishes its first cycle you will see a summary file appear in `500_Session_Archives/`. Review that file to verify that new failure signatures, new skills, and a GFI increase have been recorded. That confirmation marks the moment you have **unlocked the full Unified Cognitive Architecture**.

--- 
