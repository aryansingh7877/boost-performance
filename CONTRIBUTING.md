# Contributing to boost-performance

Thank you for contributing to `boost-performance`! This project is a Markdown-based **Antigravity Agent Skill** designed for automated, evidence-backed web performance engineering.

---

## Repository Structure

All skill instructions and reference materials reside in:
```
.agents/
└── skills/
    └── performance/
        ├── SKILL.md              # Master workflow, agent triggers, phase rules
        └── references/           # Specialized technical engineering manuals
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

---

## Core Contribution Guidelines

### 1. The Senior Engineering Principle
- **Never prescribe**: `Lighthouse Recommendation → Automatic Code Modification`.
- **Always enforce**: `OBSERVE → MEASURE → FORM HYPOTHESIS → TEST → MODIFY ONE THING → BUILD → MEASURE AGAIN → COMPARE → KEEP OR ROLLBACK`.
- Optimizations are controlled scientific experiments, not checklist tasks.

### 2. The Root-Cause Engine
Every performance problem must be decomposed into:
- **Symptom**: The observable metric delay (e.g. LCP 5.2s).
- **Evidence**: Concrete data points (trace timestamps, network waterfalls, DOM state).
- **Root-Cause Hypothesis**: The underlying browser mechanics causing the delay.

### 3. Strict Evidence Classification
All findings must use exactly one confidence tag:
- `[VERIFIED]`: Direct empirical proof (profiler call-tree, network waterfall, DOM inspection, bundle chunk analysis).
- `[LIKELY]`: Multiple strong indicators, but causation not fully isolated.
- `[HYPOTHESIS]`: Potential optimization requiring an experimental test. Hypotheses must never be stated as facts.

### 4. Generalization Over Project Specifics
- **Never hardcode project-specific artifacts**: Do not embed specific test project names, component filenames, media assets, or arbitrary test score targets.
- Ensure all rules and recommendations generalize cleanly across React, Next.js, Vite, Vue, SvelteKit, Astro, and plain HTML/CSS/JS.

### 5. The Prime Directive (Visual & Motion Preservation)
- Never propose changes that optimize metrics by removing animations, deleting 3D scenes, disabling hero videos, or flattening visual design.
- Optimizations must be additive, minimal, and preserve visual intent.

### 6. Controlled Experiments & Rollback Discipline
- Optimize **one logical thing at a time**.
- Evaluate results using repeated measurements to distinguish real improvements from synthetic measurement noise (±3–5%).
- If a change causes a performance regression or introduces any visual, animation, or functional bug: **roll it back immediately**.

---

## Development & Verification Workflow

1. Clone the repository:
   ```bash
   git clone https://github.com/aryan-sharma/boost-performance.git
   ```
2. Edit the relevant file(s) under `.agents/skills/performance/`:
   - Workflows, phases, and trigger words in `SKILL.md`.
   - Domain-specific heuristics and remediation patterns in `references/*.md`.
3. Test your changes against real web projects:
   - Copy `.agents/skills/performance/` into a test web application.
   - Open the project in Antigravity and invoke `/performance`.
   - Verify that the agent adheres to read-only inspection, properly classifies findings, requests user approval, builds the production bundle, and validates runtime behavior without regressions.
4. Document your changes in `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/).
5. Submit a clear pull request detailing the improvements and test results.

---

## Code of Conduct

Please treat all contributors and maintainers with respect, empathy, and professionalism.
