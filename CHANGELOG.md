# Changelog

All notable changes to the **boost-performance** project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [2.2.0] - 2026-09-21

### Changed — Senior Evidence-Driven Performance Engineering Upgrade
- **Core Engineering Methodology**: Redesigned the skill around `OBSERVE → MEASURE → FORM HYPOTHESIS → TEST → MODIFY ONE THING → BUILD → MEASURE AGAIN → COMPARE → KEEP OR ROLLBACK`, eliminating the anti-pattern of `Lighthouse Recommendation → Automatic Code Modification`.
- **The 9 Questions for Every Optimization**: Enforced that every intervention must answer: What is slow? Where? What evidence proves it? Likely root cause? Exact testing change? Expected metric improvement? Potential regressions? Verification method? Rollback plan?
- **3-Layer Root-Cause Engine**: Formalized the separation of every problem into Symptom, Evidence, and Root-Cause Hypothesis.
- **Stricter Confidence Classifications**: Hardened criteria for `[VERIFIED]`, `[LIKELY]`, and `[HYPOTHESIS]` to prevent hypotheses from being presented as established facts.
- **Environment Detection Realities**: Added explicit recognition of Vite dev patterns (`/@vite/client`, `/@react-refresh`, unbundled ES module waterfalls, HMR) to prevent false-positive diagnoses of production bundle size or TBT.
- **Noise vs. Delta Rule**: Embedded repeated measurement protocols to distinguish real improvements from synthetic measurement variance (±3–5%).
- **Single-Variable Controlled Experiments**: Mandated optimizing one logical item at a time with automatic rollback upon performance regression or functional defects.
- **Package Manager Auto-Detection**: Added native detection for `pnpm`, `yarn`, `bun`, and `npm`.
- **Causal 11-Factor LCP Debugging**: Updated `SKILL.md` and `references/lighthouse.md` to trace exact candidate nodes, discovery timing, and bandwidth competition.
- **Context-Aware Asset Engineering**: Updated `references/images.md` and `references/video.md` to evaluate assets based on fold placement, DPR, and LCP status rather than arbitrary file size thresholds.
- **JavaScript Payload vs. Execution Separation**: Updated `references/javascript.md` with 5-question Long Task diagnostics (Who, What, When, How long, Why) and code-splitting invariants.
- **Profiler-Backed Animation & 3D WebGL**: Updated `references/animations.md`, `references/gsap-scrolltrigger.md`, and `references/threejs-webgl.md` to require direct profiler evidence before blaming motion libraries, and to separate CPU vs. GPU bottlenecks.
- **Updated Documentation**: Completely aligned `README.md` and `CONTRIBUTING.md` with senior performance engineering standards.

## [2.1.0] - 2026-09-21

### Changed — Master Antigravity Agent Skill Architecture
- **Purged Legacy CLI Directory**: Completely deleted `legacy/` (73 files including legacy Node CLI engine, CLI binaries, tests, and outdated installer adapters). The repository is now 100% pure Markdown skill instructions.
- **Master 14-Step Workflow**: Overhauled `SKILL.md` to establish the authoritative 14-step performance workflow with strict separation between read-only AUDIT MODE and OPTIMIZATION MODE.
- **Environment Detection Protocol**: Added mandatory detection distinguishing Development environments (Vite dev, unbundled modules, HMR, React development runtime) from Production builds. Mandated that development TBT must never be treated as a production bottleneck without independent evidence, and established Production Lighthouse as the primary baseline.
- **Standardized Evidence Classification**: Enforced explicit labeling for all diagnostic findings with `[VERIFIED]`, `[LIKELY]`, or `[HYPOTHESIS]`. Hypotheses are strictly forbidden from being stated as facts.
- **User Approval Gate**: Established an explicit stop-and-wait gate requiring user confirmation of findings and planned file changes before any project file modifications occur.
- **Evidence-Based LCP Discovery**: Updated `SKILL.md` and `references/lighthouse.md` with explicit LCP element/resource discovery protocols, prohibiting blind preloading and forbidding lazy-loading of above-the-fold hero content.
- **Fold-Aware Video Strategy**: Enhanced `references/video.md` with above vs. below fold checks, intersection-based playback pausing, mobile autoplay compliance (`muted playsinline`), and a strict rule forbidding video removal to manipulate scores.
- **Animation & 3D Profiling Safeguards**: Refined `references/animations.md`, `references/gsap-scrolltrigger.md`, and `references/threejs-webgl.md` to prohibit automatically blaming GSAP or Three.js without profiler traces, while enforcing DPR clamping, WebGL resource disposal, and offscreen render pausing.
- **Runtime Regression Verification**: Added a mandatory post-optimization verification checklist protecting against React DOM reconciliation errors, broken Suspense/lazy boundaries, console errors, animation disruptions, 3D crashes, and media autoplay failures.
- **Mandatory Production Verification**: Required post-optimization production build, production preview startup, and production-to-production Lighthouse re-measurement.
- **Standardized Before/After Reporting**: Formatted reporting with standardized Core Web Vitals delta tables, asset payload tracking, and visual preservation confirmations.
- **Cleaned Public Documentation**: Rewrote `README.md` and `CONTRIBUTING.md` around native Antigravity skill usage without any CLI, npm, or npx installation references.

## [2.0.1] - 2026-09-21

### Changed — skill quality review
- Replaced fixed numeric performance targets ("60–120 FPS," "sub-second LCP," "~0 CLS") throughout `SKILL.md`, `README.md`, and `references/animations.md` with context-aware, evidence-based language. Core Web Vitals thresholds in `references/lighthouse.md` are now explicitly framed as reference points, not pass/fail requirements.
- Restructured `SKILL.md`'s workflow into eight explicit phases (Understand, Inspect, Measure, Diagnose, Plan, Implement, Validate, Report), separating diagnosis from planning and adding an explicit "risk to design/functionality" field to every finding.
- Expanded the Prime Directive to explicitly name every protected category (Lottie, Rive, shaders, smooth scrolling, transitions, responsive behavior) alongside GSAP/Three.js/video, and to state that optimization is always preferred over removal, with removal only ever a flagged, user-approved trade-off.
- Added missing coverage to `references/gsap-scrolltrigger.md` (repeated DOM queries/layout thrashing, timeline complexity, `ScrollTrigger.matchMedia()` for responsive/mobile behavior) and `references/threejs-webgl.md` (lights).
- Added explicit lab-vs-field and device/network variance guidance to `references/lighthouse.md`.
- Tightened `README.md` to describe the eight-phase workflow and reiterate there is no fixed numeric target.

### Verified (no code change needed)
- `.agents/skills/performance/SKILL.md` exists, `name: performance`, all 13 reference files present and linked from the reference table.
- No active documentation (outside `legacy/`) instructs the user to `npm install`, `npx`, or run a CLI command for this skill.
- `legacy/` remains fully separated and is not imported or required by the active skill.

## [2.0.0] - 2026-09-21

### Changed — architecture migration
- **Retired the npm/CLI product entirely.** `boost-performance` is no longer a published npm package and has no CLI, no `npx` command, and no runtime dependency. It is now a Markdown-based **Antigravity Agent Skill**.
- Moved the npm package, CLI binaries, Node.js audit engine (`core/`), and per-tool installer adapters (`adapters/`) to `legacy/cli/`. Preserved for historical reference only; not part of the current product.
- Moved the old unit test suite to `legacy/tests/` (it tested the now-archived CLI engine).
- Moved generated multi-tool adapter output (`.cursor/`, `.cursorrules`, `.codex/`, `.github/copilot-instructions.md`, `CLAUDE.md`, the installed `.gemini/skills/performance/` copy) to `legacy/multi-agent-adapters/` — these referenced the now-removed `npx boost-performance audit` command.
- Moved the original hand-authored `skill/` source to `legacy/original-skill-source/` for traceability.

### Added
- New skill entry point at `.agents/skills/performance/SKILL.md`, invocable in Antigravity as `/performance`, rewritten as a fully self-contained, no-CLI-dependency workflow.
- Expanded reference documentation from 4 to 13 focused files under `.agents/skills/performance/references/`: `lighthouse.md`, `react.md`, `nextjs.md`, `vite.md`, `javascript.md`, `css.md`, `images.md`, `fonts.md`, `animations.md`, `gsap-scrolltrigger.md`, `threejs-webgl.md`, `video.md`, and a new `mobile-performance.md`.
- Rewrote `README.md` and `CONTRIBUTING.md` around the skill-based architecture; installation via npm/npx is no longer documented as the way to use this project.

## [1.0.0] - 2026-09-18

### Added
- **Core Performance Engine:**
  - Recursive project file scanner (`core/scanner.js`) with intelligent category filtering.
  - Multi-framework and build tool detector (`core/project-detector.js`) supporting Next.js, React, Vue, SvelteKit, Astro, Remix, Vite, Turbopack, and Webpack.
  - Live Lighthouse audit integration with graceful fallback when Chrome/Lighthouse is unavailable (`core/lighthouse.js`).
  - Image and media asset audit engine (`core/assets.js`) with LCP priority checks and CLS dimension warnings.
  - Video asset optimizer (`core/video.js`) checking mobile autoplay flags (`muted`, `playsInline`), WebM fallbacks, and poster frames.
  - JavaScript & dependency auditor (`core/javascript.js`) detecting heavy packages and uncleaned event listeners.
  - React & Next.js auditor (`core/react.js`) detecting unstable keys, unmemoized context values, and root `'use client'` misuse.
  - CSS auditor (`core/css.js`) detecting layout-triggering properties and dangerous `will-change: all`.
  - Font optimizer (`core/fonts.js`) detecting legacy formats, missing `font-display`, and excessive weights.
  - Modern animation engine auditor (`core/animations.js`) with deep GSAP, ScrollTrigger, Framer Motion, and Lenis rules.
  - Three.js and React Three Fiber (R3F) auditor (`core/threejs.js`) checking `devicePixelRatio` clamping, WebGL resource disposal, and shadow maps.
  - Network and third-party script auditor (`core/network.js`).
  - Multi-format reporter (`core/reporter.js`) outputting terminal summaries, Markdown reports, and JSON data.
- **AI Coding Agent Skills & Adapters:**
  - Full AI skill specification in `skill/SKILL.md` with 5-phase engineering protocol.
  - Technical engineering reference manuals (`skill/references/`).
  - Google Gemini / Antigravity skill adapter (`adapters/antigravity/`).
  - Claude Code `/performance` command and `CLAUDE.md` adapter (`adapters/claude/`).
  - Cursor MDC rules and `.cursorrules` adapter (`adapters/cursor/`).
  - OpenAI Codex & GitHub Copilot adapter (`adapters/codex/`).
  - Cross-platform idempotent installer (`adapters/shared-installer.js`).
- **CLI Commands:**
  - `boost-performance` and `aryan-performance` binaries with `audit`, `install`, `optimize`, `doctor`, `report`, `init`, and `update` commands.
- **Automated Test Suite:**
  - Unit tests for detection, assets, animations, Three.js, reporter, and CLI commands.
