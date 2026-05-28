MD 003 Iterapp SPEC meta-scaffolding 
# ITERAPP Meta-Scaffolding System

## Complete Technical Specification — Framework 003

> **Status**: Specification v0.1 — Designed for implementation
> **Scope**: Model-agnostic cognitive mode orchestration via agentic scaffolding
> **Dependency**: Frameworks 001 (Asymmetric Iteration Axiom) + 002 (Meta-Intelligent Routing)

-----

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Architecture](#2-architecture)
3. [Topology Analyzer — Technical Spec](#3-topology-analyzer--technical-spec)
4. [Mode Configurations — Full Spec](#4-mode-configurations--full-spec)
5. [Scaffolding Primitives](#5-scaffolding-primitives)
6. [The Router — Decision Logic](#6-the-router--decision-logic)
7. [Mid-Task Switch Protocol](#7-mid-task-switch-protocol)
8. [API Contract](#8-api-contract)
9. [Demo Implementation](#9-demo-implementation)
10. [Known Limitations & Failure Modes](#10-known-limitations--failure-modes)

-----

## 1. System Overview

The ITERAPP Meta-Scaffolding System (MSS) is a model-agnostic orchestration layer that sits between the user and any LLM. It does not modify model weights, does not require fine-tuning, and does not depend on any specific provider.

**What it controls:**

- System prompt composition (mode activation)
- Context window structure (what goes in, in what order)
- Sampling parameters (temperature, top-p, presence penalty)
- Output constraints (format, length, structure)
- Step scaffolding (CoT, ToT, or none — selected per mode)
- Mid-task topology monitoring and mode switching

**What it does NOT do:**

- Select between different models (optional extension, not core)
- Modify model weights
- Require access beyond standard completion API

```
USER INPUT
    ↓
[TOPOLOGY ANALYZER]        ← lightweight, fast, runs first
    ↓
[META ROUTER]              ← selects mode composition
    ↓
[SCAFFOLDING BUILDER]      ← constructs the full inference configuration
    ↓
[LLM — any model]          ← executes with constructed configuration
    ↓
[OUTPUT MONITOR]           ← watches for topology shifts mid-task
    ↓
[SWITCH CONTROLLER]        ← decides if mode change is warranted
    ↓
OUTPUT
```

-----

## 2. Architecture

### 2.1 Component Map

```
iterapp-mss/
├── analyzer/
│   ├── topology_analyzer.py       # Axis detection
│   ├── feature_extractor.py       # Lightweight signal extraction
│   └── topology_vector.py         # TopologyVector dataclass
├── router/
│   ├── meta_router.py             # Mode selection logic
│   ├── mode_registry.py           # All mode configurations
│   └── switch_controller.py       # Mid-task switch logic
├── scaffolding/
│   ├── scaffolding_builder.py     # Builds inference config
│   ├── system_prompts/            # Mode system prompts
│   │   ├── panoramic.md
│   │   ├── extended_analytical.md
│   │   ├── dialectical.md
│   │   ├── associative.md
│   │   ├── archeological.md
│   │   ├── socratic.md
│   │   ├── witness.md
│   │   ├── pure_generative.md
│   │   └── verification.md
│   └── context_composers/         # Context window strategies
│       ├── holistic_composer.py
│       ├── sequential_composer.py
│       └── layered_composer.py
├── monitor/
│   ├── output_monitor.py          # Real-time topology shift detection
│   └── loop_detector.py           # Recursion without progress
├── adapters/
│   ├── anthropic_adapter.py
│   ├── openai_adapter.py
│   ├── ollama_adapter.py          # Local models
│   └── base_adapter.py            # Interface for new providers
└── core/
    ├── mss_engine.py              # Main orchestrator
    ├── config.py
    └── models.py                  # Shared dataclasses
```

### 2.2 Core Data Models

```python
from dataclasses import dataclass
from enum import Enum
from typing import Optional

class CognitiveMode(Enum):
    PANORAMIC         = "panoramic"
    EXTENDED_ANALYTIC = "extended_analytic"
    DIALECTICAL       = "dialectical"
    ASSOCIATIVE       = "associative"
    ARCHEOLOGICAL     = "archeological"
    SOCRATIC          = "socratic"
    WITNESS           = "witness"
    PURE_GENERATIVE   = "pure_generative"
    VERIFICATION      = "verification"

@dataclass
class TopologyVector:
    convergence: float          # 0=open exploration, 1=one correct answer
    dimensional_density: float  # 0=single dimension, 1=many independent dims
    emergence_potential: float  # 0=linear derivation, 1=non-linear emergence
    interdependence: float      # 0=loosely coupled, 1=tightly coupled
    temporal_horizon: float     # 0=step verification, 1=long-arc synthesis
    epistemic_anchor: float     # 0=generative freedom, 1=must be verifiable
    relational_vs_comp: float   # 0=compositional, 1=relational

    def dominant_axis(self) -> str:
        axes = {
            "convergence": self.convergence,
            "dimensional_density": self.dimensional_density,
            "emergence_potential": self.emergence_potential,
            "interdependence": self.interdependence,
        }
        return max(axes, key=axes.get)

@dataclass
class ModeConfiguration:
    mode: CognitiveMode
    system_prompt: str
    temperature: float
    top_p: float
    presence_penalty: float
    frequency_penalty: float
    max_tokens: Optional[int]
    context_strategy: str        # "holistic" | "sequential" | "layered"
    step_scaffolding: str        # "none" | "cot" | "tot" | "structured"
    output_format: Optional[str] # None = freeform

@dataclass
class InferenceConfig:
    mode: CognitiveMode
    mode_config: ModeConfiguration
    topology_vector: TopologyVector
    context: str
    switch_threshold: float      # From Asymmetric Axiom cost model
    antagonist_mode: Optional[CognitiveMode]

@dataclass
class TaskResult:
    output: str
    mode_used: CognitiveMode
    topology_at_start: TopologyVector
    topology_at_end: TopologyVector
    switches_performed: list[tuple[CognitiveMode, CognitiveMode]]
    antagonist_pass_applied: bool
```

-----

## 3. Topology Analyzer — Technical Spec

Two-stage detection: fast heuristic first, meta-prompt second if ambiguous.

### 3.1 Stage 1 — Heuristic Feature Extraction (< 50ms, no LLM call)

```python
class FeatureExtractor:
    """
    Extracts topology signals from raw text without LLM.
    Fast, cheap, model-agnostic.
    """

    # Convergence signals
    CONVERGENCE_MARKERS = [
        "what is", "calculate", "prove", "find the", "solve",
        "how many", "what are the steps", "debug", "fix",
        "qué es", "calcula", "demuestra", "cuántos"
    ]
    DIVERGENCE_MARKERS = [
        "explore", "imagine", "what if", "how might", "brainstorm",
        "what are possible", "generate ideas", "think about",
        "explora", "imagina", "qué pasaría", "cómo podría"
    ]

    # Dimensional density signals
    def estimate_dimensional_density(self, text: str) -> float:
        """
        Proxy: count unique named entities + distinct concepts.
        High unique entity count → high dimensional density.
        """
        # Simple: count capitalized multi-word phrases + connector words
        # ("relationship between X and Y" = 2 dims minimum)
        connectors = ["between", "and", "versus", "vs", "compared to",
                      "relationship", "interaction", "trade-off", "balance"]
        connector_count = sum(text.lower().count(c) for c in connectors)
        word_count = len(text.split())
        return min(1.0, connector_count / max(word_count / 20, 1))

    # Emergence potential signals
    EMERGENCE_MARKERS = [
        "novel", "new approach", "creative", "unexpected",
        "breakthrough", "innovative", "original", "unique",
        "never been done", "differently"
    ]

    # Temporal horizon signals
    LONG_HORIZON_MARKERS = [
        "overall", "big picture", "strategy", "vision", "framework",
        "architecture", "system", "long-term", "structure", "design"
    ]
    SHORT_HORIZON_MARKERS = [
        "step by step", "verify", "check", "confirm", "this specific",
        "exactly", "precisely", "correct this"
    ]

    def extract(self, text: str) -> dict[str, float]:
        text_lower = text.lower()

        convergence_score = sum(1 for m in self.CONVERGENCE_MARKERS
                                if m in text_lower)
        divergence_score = sum(1 for m in self.DIVERGENCE_MARKERS
                               if m in text_lower)
        total = convergence_score + divergence_score + 0.001

        return {
            "convergence": convergence_score / total,
            "dimensional_density": self.estimate_dimensional_density(text),
            "emergence_potential": min(1.0, sum(1 for m in
                self.EMERGENCE_MARKERS if m in text_lower) / 3),
            "temporal_horizon": self._horizon_score(text_lower),
            "epistemic_anchor": convergence_score / total,
        }

    def _horizon_score(self, text: str) -> float:
        long = sum(1 for m in self.LONG_HORIZON_MARKERS if m in text)
        short = sum(1 for m in self.SHORT_HORIZON_MARKERS if m in text)
        return long / (long + short + 0.001)
```

### 3.2 Stage 2 — Meta-Prompt Refinement (when heuristic is ambiguous)

Triggered when any axis score falls in the 0.35–0.65 range (ambiguous zone).

```python
TOPOLOGY_META_PROMPT = """
Analyze the following task. Respond ONLY with a JSON object, no preamble.

Score each axis from 0.0 to 1.0:
- convergence: (0=open exploration with no single answer, 1=one correct verifiable answer)
- dimensional_density: (0=single concept, 1=many independent dimensions must be held)
- emergence_potential: (0=answer derivable linearly, 1=answer can only arise non-linearly)
- interdependence: (0=components independent, 1=all components affect each other)
- temporal_horizon: (0=step-by-step verification, 1=long synthesis arc)
- epistemic_anchor: (0=generative freedom acceptable, 1=must be verifiable/factual)
- relational_vs_comp: (0=build new structure, 1=understand relationships between existing)

Task: {task}

Respond with JSON only:
{{"convergence": 0.0, "dimensional_density": 0.0, ...}}
"""

class TopologyAnalyzer:
    def __init__(self, llm_adapter, ambiguity_threshold=0.3):
        self.extractor = FeatureExtractor()
        self.llm = llm_adapter
        self.threshold = ambiguity_threshold

    def analyze(self, task: str) -> TopologyVector:
        features = self.extractor.extract(task)

        # Check if any axis is ambiguous
        ambiguous = any(
            self.threshold < v < (1 - self.threshold)
            for v in features.values()
        )

        if ambiguous:
            # Stage 2: meta-prompt refinement
            refined = self._meta_prompt_analysis(task)
            # Weighted average: heuristic 30%, meta-prompt 70%
            for k in features:
                features[k] = 0.3 * features[k] + 0.7 * refined.get(k, features[k])

        return TopologyVector(**features,
                              relational_vs_comp=features.get("relational_vs_comp", 0.5))

    def _meta_prompt_analysis(self, task: str) -> dict:
        prompt = TOPOLOGY_META_PROMPT.format(task=task)
        response = self.llm.complete(
            prompt=prompt,
            temperature=0.0,    # Deterministic for analysis
            max_tokens=150
        )
        import json
        try:
            return json.loads(response.strip())
        except json.JSONDecodeError:
            return {}           # Fall back to heuristic only
```

-----

## 4. Mode Configurations — Full Spec

### 4.1 Panoramic Mode

```python
PANORAMIC_CONFIG = ModeConfiguration(
    mode=CognitiveMode.PANORAMIC,
    system_prompt="""
You are operating in PANORAMIC MODE.

Your primary function is to hold the entire problem space in view simultaneously
and identify connections, patterns, and relationships across all dimensions at once.

ACTIVE CONSTRAINTS:
- Do NOT reason step-by-step unless absolutely necessary
- Do NOT narrow to a single thread before exploring the full landscape
- PRIORITIZE: cross-domain connections, structural similarities, emergent patterns
- ALLOW: non-linear jumps, associative leaps, simultaneous multi-axis consideration
- AVOID: premature convergence, single-thread analysis, sequential verification

You are flying at altitude. See the whole landscape before descending anywhere.
    """,
    temperature=0.85,
    top_p=0.95,
    presence_penalty=0.6,     # Penalize repetition → broader coverage
    frequency_penalty=0.3,
    max_tokens=None,
    context_strategy="holistic",   # Full context always present
    step_scaffolding="none",       # No CoT — suppresses linear reasoning
    output_format=None             # Free-form synthesis
)
```

### 4.2 Extended Analytical Mode

```python
EXTENDED_ANALYTIC_CONFIG = ModeConfiguration(
    mode=CognitiveMode.EXTENDED_ANALYTIC,
    system_prompt="""
You are operating in EXTENDED ANALYTICAL MODE.

Your primary function is rigorous step-by-step reasoning toward a verifiable answer.

ACTIVE CONSTRAINTS:
- Reason explicitly step by step
- Verify each step before proceeding
- Prefer precision over coverage
- Flag assumptions explicitly
- Converge toward a single best answer

Think slowly. Think carefully. Check your work.
    """,
    temperature=0.2,
    top_p=0.85,
    presence_penalty=0.0,
    frequency_penalty=0.0,
    max_tokens=None,
    context_strategy="sequential",  # Build context incrementally
    step_scaffolding="cot",         # Explicit chain-of-thought
    output_format="structured"
)
```

### 4.3 Dialectical Mode

```python
DIALECTICAL_CONFIG = ModeConfiguration(
    mode=CognitiveMode.DIALECTICAL,
    system_prompt="""
You are operating in DIALECTICAL MODE.

Before reaching any conclusion, you must fully develop BOTH:
1. The strongest possible case FOR the primary direction
2. The strongest possible case AGAINST it (or for an alternative)

Only after both cases are fully developed may you synthesize.

Structure your response as:
THESIS: [strongest case for primary direction]
ANTITHESIS: [strongest case against, or for alternative]
SYNTHESIS: [what becomes true when both are held simultaneously]

Do not collapse to one side prematurely. The tension is where truth lives.
    """,
    temperature=0.7,
    top_p=0.92,
    presence_penalty=0.2,
    frequency_penalty=0.1,
    max_tokens=None,
    context_strategy="holistic",
    step_scaffolding="structured",
    output_format="thesis-antithesis-synthesis"
)
```

### 4.4 Associative Mode

```python
ASSOCIATIVE_CONFIG = ModeConfiguration(
    mode=CognitiveMode.ASSOCIATIVE,
    system_prompt="""
You are operating in ASSOCIATIVE MODE.

Your primary function is to generate unexpected connections between this problem
and distant domains, structures, or patterns.

ACTIVE CONSTRAINTS:
- Actively seek analogies from unrelated fields
- Follow structural similarity, not surface similarity
- Prioritize: surprising connections, cross-domain mappings, novel framings
- Do NOT restrict yourself to the obvious domain of the problem
- Each connection should reveal something the direct approach would miss

You are a bridge builder between islands of knowledge that rarely see each other.
    """,
    temperature=1.0,
    top_p=0.98,
    presence_penalty=0.8,     # Maximum diversity
    frequency_penalty=0.5,
    max_tokens=None,
    context_strategy="layered",  # Core problem + broad context
    step_scaffolding="none",
    output_format=None
)
```

### 4.5 Witness Mode

```python
WITNESS_CONFIG = ModeConfiguration(
    mode=CognitiveMode.WITNESS,
    system_prompt="""
You are operating in WITNESS MODE.

You are not solving the problem. You are observing the reasoning process
that has been applied to it so far and reporting what you see with clinical detachment.

Analyze:
1. What cognitive mode has been active? Was it appropriate for the task topology?
2. Are there signs of loop (same structure recurring without new information)?
3. Has the problem space shifted since the reasoning began?
4. What dimension is being systematically neglected?
5. Is this iteration or reformulation territory?

Report only what you observe. Do not solve. Diagnose.
    """,
    temperature=0.1,          # Maximum determinism for observation
    top_p=0.8,
    presence_penalty=0.0,
    frequency_penalty=0.0,
    max_tokens=500,            # Focused diagnostic output
    context_strategy="holistic",  # Needs full history
    step_scaffolding="structured",
    output_format="diagnostic"
)
```

### 4.6 Socratic Mode

```python
SOCRATIC_CONFIG = ModeConfiguration(
    mode=CognitiveMode.SOCRATIC,
    system_prompt="""
You are operating in SOCRATIC MODE.

Do not answer the question. Generate the questions that, if answered,
would reveal the true structure of the problem.

Your output is ONLY questions — never answers.
Prioritize questions that:
- Reveal hidden assumptions
- Expose undefined terms
- Identify the actual goal beneath the stated goal
- Find the dimension the user hasn't considered
- Challenge the framing itself

The best Socratic question makes the problem look completely different.
    """,
    temperature=0.75,
    top_p=0.92,
    presence_penalty=0.4,
    frequency_penalty=0.2,
    max_tokens=400,
    context_strategy="holistic",
    step_scaffolding="none",
    output_format="questions-only"
)
```

-----

## 5. Scaffolding Primitives

### 5.1 Context Composers

```python
class HolisticComposer:
    """
    Always includes the full problem context.
    Used by: Panoramic, Dialectical, Witness, Holistic modes.
    """
    def compose(self, task: str, history: list, background: str) -> str:
        return f"""
FULL CONTEXT:
{background}

CONVERSATION HISTORY:
{self._format_history(history)}

CURRENT TASK:
{task}
        """

class SequentialComposer:
    """
    Builds context incrementally, step by step.
    Used by: Extended Analytic, Verification, Archeological modes.
    """
    def compose(self, task: str, history: list, current_step: int) -> str:
        # Only includes immediately relevant prior step
        recent = history[-2:] if len(history) > 2 else history
        return f"""
RECENT CONTEXT:
{self._format_history(recent)}

CURRENT STEP {current_step}:
{task}
        """

class LayeredComposer:
    """
    Core problem in focus, broader context available but not foregrounded.
    Used by: Associative, Pure Generative modes.
    """
    def compose(self, task: str, history: list, background: str) -> str:
        return f"""
CORE PROBLEM (primary focus):
{task}

BACKGROUND (secondary, use if relevant):
{background[:500] if background else "None provided"}
        """
```

### 5.2 Step Scaffolding Templates

```python
SCAFFOLDING_TEMPLATES = {
    "cot": """
Before answering, reason through this step by step.
Show your reasoning explicitly. Each step should follow from the previous.
""",
    "tot": """
Consider multiple reasoning paths simultaneously:
PATH A: [explore first approach]
PATH B: [explore alternative approach]
PATH C: [explore a third direction]
Then select and develop the most promising path.
""",
    "structured": """
Structure your response with clear sections.
Each section should build on the previous.
""",
    "none": ""  # No scaffolding — used for panoramic/generative modes
}
```

-----

## 6. The Router — Decision Logic

```python
class MetaRouter:

    # Mode selection matrix based on topology vector
    # Format: (convergence_range, density_range, emergence_range) → mode
    ROUTING_RULES = [
        # High convergence, low density, low emergence → pure analytical
        {"convergence": (0.7, 1.0), "dimensional_density": (0.0, 0.4),
         "emergence_potential": (0.0, 0.3), "mode": CognitiveMode.EXTENDED_ANALYTIC},

        # Low convergence, high density, high emergence → panoramic
        {"convergence": (0.0, 0.35), "dimensional_density": (0.6, 1.0),
         "emergence_potential": (0.5, 1.0), "mode": CognitiveMode.PANORAMIC},

        # High tradeoff signal (interdependence mid + both poles present) → dialectical
        {"interdependence": (0.5, 1.0), "convergence": (0.3, 0.7),
         "mode": CognitiveMode.DIALECTICAL},

        # High emergence, low convergence, low density → associative
        {"emergence_potential": (0.7, 1.0), "convergence": (0.0, 0.4),
         "dimensional_density": (0.0, 0.5), "mode": CognitiveMode.ASSOCIATIVE},

        # High temporal horizon, high density, low convergence → panoramic
        {"temporal_horizon": (0.7, 1.0), "dimensional_density": (0.5, 1.0),
         "mode": CognitiveMode.PANORAMIC},

        # High epistemic anchor, high convergence → verification
        {"epistemic_anchor": (0.8, 1.0), "convergence": (0.7, 1.0),
         "mode": CognitiveMode.VERIFICATION},
    ]

    # Antagonist pairs
    ANTAGONIST_MAP = {
        CognitiveMode.EXTENDED_ANALYTIC: CognitiveMode.PANORAMIC,
        CognitiveMode.PANORAMIC: CognitiveMode.VERIFICATION,
        CognitiveMode.ASSOCIATIVE: CognitiveMode.VERIFICATION,
        CognitiveMode.PURE_GENERATIVE: CognitiveMode.SOCRATIC,
    }

    def select_mode(self, tv: TopologyVector) -> CognitiveMode:
        tv_dict = {
            "convergence": tv.convergence,
            "dimensional_density": tv.dimensional_density,
            "emergence_potential": tv.emergence_potential,
            "interdependence": tv.interdependence,
            "temporal_horizon": tv.temporal_horizon,
            "epistemic_anchor": tv.epistemic_anchor,
        }

        scores = {}
        for rule in self.ROUTING_RULES:
            mode = rule["mode"]
            match_score = 0
            axes_checked = 0
            for axis, (low, high) in rule.items():
                if axis == "mode":
                    continue
                val = tv_dict.get(axis, 0.5)
                if low <= val <= high:
                    match_score += 1
                axes_checked += 1
            if axes_checked > 0:
                scores[mode] = scores.get(mode, 0) + match_score / axes_checked

        if not scores:
            return CognitiveMode.PANORAMIC  # Safe default

        return max(scores, key=scores.get)

    def needs_antagonist(self, tv: TopologyVector, mode: CognitiveMode,
                          task_length: int) -> bool:
        """
        Apply antagonist mode if:
        - Task is complex enough to warrant it (length proxy)
        - Primary mode has a known antagonist
        - Topology suggests the blind spot of the primary mode
        """
        if mode not in self.ANTAGONIST_MAP:
            return False
        if task_length < 100:    # Short tasks don't need antagonist pass
            return False
        # Analytical mode on high-emergence task → antagonist needed
        if mode == CognitiveMode.EXTENDED_ANALYTIC and tv.emergence_potential > 0.5:
            return True
        if mode == CognitiveMode.PURE_GENERATIVE and tv.epistemic_anchor > 0.5:
            return True
        return False
```

-----

## 7. Mid-Task Switch Protocol

```python
class OutputMonitor:
    """
    Monitors streaming or batched output for topology shift signals.
    """

    LOOP_SIGNALS = [
        "as I mentioned", "as stated above", "again,", "to reiterate",
        "como mencioné", "como dije", "nuevamente"
    ]

    DIMENSION_EXPANSION_SIGNALS = [
        "however, there's another", "but we also need to consider",
        "this also connects to", "another dimension",
        "pero también hay que considerar", "esto también conecta con"
    ]

    CONVERGENCE_INVERSION_SIGNALS = [
        "actually, it depends", "this is more complex than",
        "there are multiple valid", "it's not a single answer",
        "en realidad depende", "es más complejo de lo que"
    ]

    def detect_shifts(self, output_so_far: str,
                      original_tv: TopologyVector) -> dict:
        output_lower = output_so_far.lower()
        shifts = {}

        loop_count = sum(1 for s in self.LOOP_SIGNALS if s in output_lower)
        if loop_count >= 2:
            shifts["loop_detected"] = True

        dim_expansion = sum(1 for s in self.DIMENSION_EXPANSION_SIGNALS
                            if s in output_lower)
        if dim_expansion >= 2 and original_tv.dimensional_density < 0.5:
            shifts["dimensional_expansion"] = True

        conv_inversion = sum(1 for s in self.CONVERGENCE_INVERSION_SIGNALS
                             if s in output_lower)
        if conv_inversion >= 1 and original_tv.convergence > 0.6:
            shifts["convergence_inversion"] = True

        return shifts


class SwitchController:
    """
    Applies the Asymmetric Iteration Axiom to mode switching decisions.
    """

    # Estimated switching costs (normalized 0-1)
    SWITCH_COSTS = {
        (CognitiveMode.EXTENDED_ANALYTIC, CognitiveMode.PANORAMIC): 0.4,
        (CognitiveMode.PANORAMIC, CognitiveMode.VERIFICATION): 0.3,
        (CognitiveMode.ASSOCIATIVE, CognitiveMode.VERIFICATION): 0.35,
        (CognitiveMode.EXTENDED_ANALYTIC, CognitiveMode.DIALECTICAL): 0.5,
    }

    # Estimated gains from switch given specific trigger
    SWITCH_GAINS = {
        "loop_detected": 0.7,           # High gain — stuck is stuck
        "dimensional_expansion": 0.5,   # Medium gain
        "convergence_inversion": 0.6,   # High gain — wrong mode for task
    }

    def should_switch(self, current_mode: CognitiveMode,
                      proposed_mode: CognitiveMode,
                      triggers: dict,
                      uncertainty: float = 0.15) -> bool:
        if not triggers:
            return False

        cost = self.SWITCH_COSTS.get(
            (current_mode, proposed_mode), 0.5
        )
        gain = max(self.SWITCH_GAINS.get(t, 0) for t in triggers)

        # Asymmetric threshold: gain must exceed cost × (1 + uncertainty)
        return gain > cost * (1 + uncertainty)
```

-----

## 8. API Contract

### 8.1 Main Interface

```python
class MSSEngine:
    """
    Main entry point. Model-agnostic.
    """

    def __init__(self, adapter, config=None):
        self.adapter = adapter          # Any LLM adapter
        self.analyzer = TopologyAnalyzer(adapter)
        self.router = MetaRouter()
        self.monitor = OutputMonitor()
        self.switch_ctrl = SwitchController()
        self.mode_registry = ModeRegistry()

    def run(self, task: str, context: str = "",
            history: list = None, force_mode: str = None) -> TaskResult:

        history = history or []

        # 1. Analyze topology
        tv = self.analyzer.analyze(task)

        # 2. Select mode
        if force_mode:
            mode = CognitiveMode(force_mode)
        else:
            mode = self.router.select_mode(tv)

        # 3. Check antagonist need
        needs_antagonist = self.router.needs_antagonist(
            tv, mode, len(task)
        )

        # 4. Build inference config
        mode_config = self.mode_registry.get(mode)
        inference_config = self._build_config(
            task, context, history, mode_config, tv
        )

        # 5. Execute primary mode
        output = self.adapter.complete(inference_config)

        # 6. Monitor for topology shifts
        shifts = self.monitor.detect_shifts(output, tv)
        switches = []

        if shifts:
            new_mode = self._select_switch_mode(mode, shifts, tv)
            if new_mode and self.switch_ctrl.should_switch(
                mode, new_mode, shifts
            ):
                # Switch mode and continue
                new_config = self._build_config(
                    task, output, history, self.mode_registry.get(new_mode), tv
                )
                output = self.adapter.complete(new_config)
                switches.append((mode, new_mode))
                mode = new_mode

        # 7. Antagonist pass if needed
        antagonist_applied = False
        if needs_antagonist:
            antagonist_mode = self.router.ANTAGONIST_MAP[mode]
            ant_config = self._build_antagonist_config(
                task, output, antagonist_mode
            )
            antagonist_output = self.adapter.complete(ant_config)
            output = self._synthesize(output, antagonist_output)
            antagonist_applied = True

        tv_end = self.analyzer.analyze(output[:500])  # Re-analyze output

        return TaskResult(
            output=output,
            mode_used=mode,
            topology_at_start=tv,
            topology_at_end=tv_end,
            switches_performed=switches,
            antagonist_pass_applied=antagonist_applied
        )
```

### 8.2 Provider Adapters

```python
class BaseAdapter:
    def complete(self, config: InferenceConfig) -> str:
        raise NotImplementedError

class AnthropicAdapter(BaseAdapter):
    def __init__(self, api_key: str, model: str = "claude-sonnet-4-20250514"):
        self.client = anthropic.Anthropic(api_key=api_key)
        self.model = model

    def complete(self, config: InferenceConfig) -> str:
        response = self.client.messages.create(
            model=self.model,
            max_tokens=config.mode_config.max_tokens or 4096,
            temperature=config.mode_config.temperature,
            system=config.mode_config.system_prompt,
            messages=[{"role": "user", "content": config.context}]
        )
        return response.content[0].text

class OpenAIAdapter(BaseAdapter):
    def __init__(self, api_key: str, model: str = "gpt-4o"):
        self.client = openai.OpenAI(api_key=api_key)
        self.model = model

    def complete(self, config: InferenceConfig) -> str:
        response = self.client.chat.completions.create(
            model=self.model,
            temperature=config.mode_config.temperature,
            top_p=config.mode_config.top_p,
            presence_penalty=config.mode_config.presence_penalty,
            messages=[
                {"role": "system", "content": config.mode_config.system_prompt},
                {"role": "user", "content": config.context}
            ]
        )
        return response.choices[0].message.content

class OllamaAdapter(BaseAdapter):
    """Local models — fully offline, zero API cost."""
    def __init__(self, model: str = "llama3", host: str = "localhost:11434"):
        self.model = model
        self.host = host

    def complete(self, config: InferenceConfig) -> str:
        import requests
        response = requests.post(
            f"http://{self.host}/api/generate",
            json={
                "model": self.model,
                "prompt": f"{config.mode_config.system_prompt}\n\n{config.context}",
                "options": {
                    "temperature": config.mode_config.temperature,
                    "top_p": config.mode_config.top_p,
                },
                "stream": False
            }
        )
        return response.json()["response"]
```

-----

## 9. Demo Implementation

Minimum viable demo for the competition — visible, measurable, impactful.

```python
"""
DEMO: Same task, four modes, same model.
Shows that mode selection produces categorially different outputs.
Then shows the router selecting the correct mode automatically.
"""

def run_demo():
    adapter = AnthropicAdapter(api_key="...", model="claude-sonnet-4-20250514")
    engine = MSSEngine(adapter)

    # Task designed to be topology-ambiguous (interesting routing case)
    task = """
    We're building a product for people with ADHD and high ability.
    They get stuck in loops. They have enormous potential but can't ship.
    What should we build?
    """

    print("=" * 60)
    print("DEMO: Meta-Intelligent Routing vs. Fixed Mode")
    print("=" * 60)

    # Run same task with four different forced modes
    for mode in ["extended_analytic", "panoramic", "dialectical", "associative"]:
        result = engine.run(task, force_mode=mode)
        print(f"\n--- MODE: {mode.upper()} ---")
        print(result.output[:400] + "...")
        print(f"Topology detected: convergence={result.topology_at_start.convergence:.2f}, "
              f"density={result.topology_at_start.dimensional_density:.2f}")

    # Now run with auto-routing
    print("\n" + "=" * 60)
    print("AUTO-ROUTED:")
    auto_result = engine.run(task)
    print(f"Router selected: {auto_result.mode_used.value}")
    print(f"Antagonist applied: {auto_result.antagonist_pass_applied}")
    print(f"Switches performed: {auto_result.switches_performed}")
    print("\nOutput:")
    print(auto_result.output[:600])

if __name__ == "__main__":
    run_demo()
```

### Expected Demo Output Structure

```
--- MODE: EXTENDED_ANALYTIC ---
Step 1: Define the problem precisely...
Step 2: Identify the mechanism of loops...
[Narrow, precise, misses the gestalt]

--- MODE: PANORAMIC ---
The product lives at the intersection of five forces simultaneously...
[Wide, connective, captures the whole landscape]

--- MODE: DIALECTICAL ---
THESIS: Build a loop-breaking system that interrupts stagnation...
ANTITHESIS: The interruption itself creates new loops...
SYNTHESIS: The product must be a loop-distinguisher, not a loop-breaker...
[Reveals tension analytical mode would miss]

--- MODE: ASSOCIATIVE ---
This is structurally identical to how a circuit breaker works...
And to how immune systems distinguish self from non-self...
[Cross-domain illumination]

AUTO-ROUTED:
Router selected: panoramic
Antagonist applied: True (verification pass)
[Best of panoramic breadth + verification anchor]
```

-----

## 10. Known Limitations & Failure Modes

### Honest Assessment

|Limitation                              |Severity  |Mitigation                                |
|----------------------------------------|----------|------------------------------------------|
|Topology analyzer accuracy              |Medium    |Stage 2 meta-prompt covers ambiguous cases|
|Routing rules are hand-crafted          |Medium    |Can be learned from feedback over time    |
|Switch cost estimates are approximate   |Low-Medium|Calibrate per domain with usage data      |
|Panoramic mode not formally validated   |Medium    |Demo provides empirical validation        |
|Temperature control varies by provider  |Low       |Adapter layer normalizes differences      |
|Context window limits mid-task switching|Medium    |Summarization before switch               |

### What This Spec Does NOT Claim

- That any specific set of routing rules is optimal — they are a reasonable starting point
- That the topology vector fully captures all relevant task properties — it is a useful approximation
- That Panoramic Mode as defined will outperform Extended Thinking on all divergent tasks — this requires empirical validation
- That the Asymmetric Threshold values in SwitchController are calibrated — they need domain-specific tuning

### What Requires Empirical Validation

1. Does Panoramic Mode (system prompt + high temperature + no CoT) produce measurably different output than instruct mode on high-dimensional tasks?
2. Does the topology analyzer correctly classify tasks at >70% accuracy on a held-out benchmark?
3. Does the antagonist pass improve output quality on complex tasks, measured by human evaluation?

These are answerable questions. The architecture makes them testable.

-----

*ITERAPP Framework 003 — Meta-Scaffolding System*
*Model-agnostic. Topology-aware. Asymmetrically optimal.*
*From chaos to your peak — with the right geometry.*