# ⚡ boost-performance

> **An Antigravity Agent Skill for evidence-driven web performance engineering.**
> Equips the Google Antigravity AI coding agent to perform disciplined, root-cause performance debugging and controlled optimization experiments across Core Web Vitals, JavaScript execution, rendering pipelines, animations, 3D WebGL, and rich media — while **strictly preserving visual design, 3D scenes, video backgrounds, and animation choreography.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## What It Is

`boost-performance` is a pure Markdown-based **Antigravity Agent Skill**. It operates natively within the Antigravity agent environment — without requiring a CLI, npm packages, binary installers, or background daemon processes.

The skill grounds the agent in the disciplined methodology of a senior performance engineer:

> **OBSERVE → MEASURE → FORM HYPOTHESIS → TEST → MODIFY ONE THING → BUILD → MEASURE AGAIN → COMPARE → KEEP OR ROLLBACK**

It **NEVER** follows the naive pattern:
> ~~LIGHTHOUSE RECOMMENDATION → AUTOMATIC CODE CHANGE~~

A Lighthouse audit reports **symptoms and heuristic opportunities**, not proven root causes. Every optimization intervention is treated as a controlled, single-variable, reversible experiment.

---

## The 9 Questions Every Optimization Must Answer

Before modifying any project code, the agent answers:

1. **What is slow?** (Specific metric or user-perceived delay)
2. **Where is it slow?** (Specific component, asset, network request, or DOM node)
3. **What evidence proves it?** (Trace timestamp, network waterfall, profiler call-tree, or DOM audit)
4. **What is the likely root cause?** (Underlying browser mechanism causing the bottleneck)
5. **What exact change will test that hypothesis?** (Targeted, minimal code or config adjustment)
6. **What metric should improve?** (Target metric and expected delta)
7. **What functionality could regress?** (Visual layout, animation timing, hydration, event listeners)
8. **How will we verify the result?** (Specific build command, preview test, or runtime checklist)
9. **What happens if the result is worse?** (Concrete rollback procedure to restore baseline)

---

## Installation

Copy the `.agents/skills/performance/` directory from this repository into your target project:

```
your-project/
└── .agents/
    └── skills/
        └── performance/
            ├── SKILL.md
            └── references/
                ├── animations.md
                ├── css.md
                ├── fonts.md
                ├── gsap-scrolltrigger.md
                ├── images.md
                ├── javascript.md
                ├── lighthouse.md
                ├── mobile-performance.md
                ├── nextjs.md
                ├── react.md
                ├── threejs-webgl.md
                ├── video.md
                └── vite.md
```

No build step, configuration file, or package manager installation is required.

---

## Usage

1. Open your project in **Antigravity**.
2. Trigger the skill by entering:
   ```
   /performance
   ```
   *(or ask naturally: "audit performance", "improve Core Web Vitals", "optimize loading speed")*
3. The agent will execute the read-only audit, label findings with strict evidence standards, present the prioritized experiment plan, and wait for your approval before modifying any code.

---

## Workflow Overview

The skill follows a strict 11-step master engineering workflow:

```
/performance
    ↓
1. Detect Environment (Development vs. Production)
    ↓
2. Detect Stack & Package Manager (npm / pnpm / yarn / bun)
    ↓
3. Establish Production Baseline (or document live tooling absence)
    ↓
4. Read-Only Inspection (Source, Configs, Assets, Network, Bundles)
    ↓
5. Root-Cause Diagnosis (Symptom → Evidence → Hypothesis)
    ↓
6. Classify Evidence ([VERIFIED] / [LIKELY] / [HYPOTHESIS])
    ↓
7. Prioritize Opportunities (Impact × Confidence × CWV Relevance)
    ↓
8. 🛑 USER APPROVAL GATE (Present Plan; Await Explicit Confirmation)
    ↓
9. Checkpoint State (Git status / Reversible branch checkpoint)
    ↓
10. Execute Controlled Experiment Loop (ONE logical change at a time):
    ┌────────────────────────────────────────────────────────┐
    │ a. Apply single minimal modification                   │
    │ b. Build production bundle (`pnpm build` / equivalent) │
    │ c. Verify build integrity & absence of errors          │
    │ d. Launch production preview server                    │
    │ e. Re-measure metrics (repeated runs to beat noise)    │
    │ f. Execute Runtime Regression Checklist                │
    │ g. Decision: IMPROVED (Keep) / REGRESSED (Rollback)    │
    └────────────────────────────────────────────────────────┘
    ↓
11. Final Verification & Production Delta Reporting
```

### 1. Environment Detection First
The skill identifies whether it is observing a Development or Production environment:
- **Development** (Vite dev server, HMR, unbundled ES modules, React development runtime): Used for logic inspection and debugging. The skill explicitly guards against treating development Total Blocking Time (TBT) or high module request counts as production bottlenecks, as dev-mode instrumentation heavily inflates both.
- **Production** (Minified bundles, production preview, tree-shaken chunks): The mandatory baseline for evaluating Core Web Vitals.

### 2. Audit Mode (Strictly Read-Only)
During initial inspection, the skill **never modifies project files**:
- Separates problems into **Symptom**, **Evidence**, and **Root-Cause Hypothesis**.
- Deconstructs LCP across 11 causal dimensions (never guessing or assuming a large image is the LCP).
- Evaluates assets contextually (file size alone is never treated as a root cause).
- Distinguishes JavaScript network payload from parse, compile, and execution costs.
- Investigates Long Tasks (>50ms) by answering: *Who? What? When? How long? Why?*
- Evaluates GSAP and Three.js usage without automatically blaming animation or 3D libraries without profiler evidence.

### 3. Strict Evidence Classification
Every finding is explicitly classified:
- `[VERIFIED]`: Direct empirical evidence confirms the finding (call-tree profiler trace, network waterfall timestamp, DOM inspection, production bundle analysis).
- `[LIKELY]`: Multiple data points strongly point toward the issue, but causation is not fully isolated.
- `[HYPOTHESIS]`: Plausible explanation or candidate optimization requiring an experiment. Hypotheses are never presented as facts.

### 4. User Approval Gate
Execution halts completely after presenting the audit report. The agent details:
- Current baseline metrics
- Evidence-classified root causes
- Proposed single-variable experiment order
- Expected impact, risks, and concrete rollback plans

**No code is touched until the user explicitly responds with approval.** Once approved, the agent executes the agreed experiment/measurement/rollback cycle autonomously within that approved scope.

### 5. Single-Variable Experiments & Rollback Protocol
- Modifies **one logical thing at a time**.
- Builds the production bundle using the project's native package manager (`npm`, `pnpm`, `yarn`, or `bun`).
- Launches the production preview server and re-measures metrics across repeated runs to distinguish genuine improvements from normal synthetic noise (±3–5% variance).
- **Mandatory Rollback**: If metrics regress or any visual, animation, or functional bug is introduced, the change is **rolled back immediately**.

### 6. Runtime Regression Check
Optimization is not considered successful if metrics improve but functionality breaks:
- [x] Application boot & hydration verified error-free
- [x] Visual design, layout, and typography verified 100% intact
- [x] GSAP / ScrollTrigger animations verified with exact timing
- [x] Three.js / WebGL scenes verified rendering at target frame rates
- [x] Video autoplay and looping behavior verified on mobile and desktop
- [x] Console log inspected: zero errors or unhandled warnings

---

## Domain Reference Guides

The skill references focused technical manuals in `.agents/skills/performance/references/`:
- [`lighthouse.md`](.agents/skills/performance/references/lighthouse.md): Symptoms vs. root causes, noise vs. delta rule, 11-factor LCP breakdown, Long Task diagnostics.
- [`images.md`](.agents/skills/performance/references/images.md): Context-aware image engineering, intrinsic vs. rendered dimensions, causal LCP rules, layout stability.
- [`video.md`](.agents/skills/performance/references/video.md): Conditional video decision engine, fold awareness, mobile autoplay, offscreen pausing.
- [`javascript.md`](.agents/skills/performance/references/javascript.md): Separating payload from execution, Long Tasks (Who/What/When/How long/Why), code-splitting invariants.
- [`react.md`](.agents/skills/performance/references/react.md): Evidence-based React debugging, re-render cascades, context splitting, memoization discipline.
- [`animations.md`](.agents/skills/performance/references/animations.md): Profiler-backed animation debugging, GPU compositor stages, layout thrashing elimination.
- [`gsap-scrolltrigger.md`](.agents/skills/performance/references/gsap-scrolltrigger.md): Lifecycle teardown (`gsap.context`), trigger batching, Lenis RAF synchronization.
- [`threejs-webgl.md`](.agents/skills/performance/references/threejs-webgl.md): CPU vs. GPU bottleneck differentiation, DPR clamping, VRAM disposal, render loop control.
- [`vite.md`](.agents/skills/performance/references/vite.md): Vite development vs. production realities, unbundled ESM, Rollup chunking.
- [`nextjs.md`](.agents/skills/performance/references/nextjs.md): Server/Client boundary discipline, dynamic imports with SSR, `next/image`, `next/font`.
- [`css.md`](.agents/skills/performance/references/css.md): Render-blocking waterfalls, GPU-composited vs. layout properties, `will-change` discipline.
- [`fonts.md`](.agents/skills/performance/references/fonts.md): WOFF2 standards, critical font preloading rules, metric-matched fallback overrides.
- [`mobile-performance.md`](.agents/skills/performance/references/mobile-performance.md): Thermal throttling, GPU power ceilings, passive touch listeners, mobile network profiles.

---

## License

MIT — see [LICENSE](LICENSE).
# boost-performance
