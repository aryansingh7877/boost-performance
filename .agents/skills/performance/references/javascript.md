# JavaScript Payload & Execution Performance Reference Manual

## 1. Separating Network Payload from CPU Execution

A large JavaScript bundle is not automatically a performance root cause. A 500KB bundle containing static lookup tables downloaded over fast broadband has low execution cost, while a 150KB bundle containing heavy regex parsers, unmemoized reconciliation loops, and synchronous crypto can lock the main thread for 400ms.

Always analyze the JavaScript lifecycle in distinct engineering layers:

| Layer | Question to Investigate | Diagnostic Tool / Evidence |
| :--- | :--- | :--- |
| **Network Payload** | How many compressed bytes are transferred over the wire? | Network panel / bundle visualizer |
| **Parse Cost** | How long does V8 / JavaScriptCore spend parsing syntax? | Performance trace "Parse Script" track |
| **Compile Cost** | How long does bytecode generation and JIT compilation take? | Performance trace "Compile Script" track |
| **Execution Cost** | Which specific functions occupy the CPU during initial startup? | Bottom-up / Call tree CPU profiler |
| **Long Tasks (>50ms)** | Which synchronous blocks block the main thread and hurt TBT/INP? | User Timing / Main thread timeline |
| **Initialization Work** | Which third-party libraries run heavy setup code immediately? | Startup CPU timeline breakdown |
| **Runtime Activity** | Does the application continue polling, recalculating, or re-rendering while idle? | CPU utilization during scroll / idle |

Only optimize the specific layer that the evidence proves is the bottleneck.

---

## 2. Main-Thread Diagnostics: The 5 Questions for Long Tasks

Whenever Total Blocking Time (TBT) or Interaction to Next Paint (INP) is degraded, investigate the main thread by answering:
1. **WHO?** Which script, dependency, or component initiated the execution?
2. **WHAT?** What work was being performed (React tree reconciliation, JSON parsing, heavy math, layout recalculation)?
3. **WHEN?** Did it happen during initial page parse, during hydration, or following user interaction?
4. **HOW LONG?** What was the exact duration in milliseconds (any task >50ms is a Long Task)?
5. **WHY?** Why was this work done synchronously on the main thread rather than batched, deferred, or delegated to a worker?

---

## 3. The Code-Splitting Protocol & The LCP Invariant

Dynamic imports (`import()`) and `React.lazy()` are not automatic optimizations. Code splitting is a double-edged sword: splitting code can reduce initial download size, but introduces network round-trips and chunk execution waterfalls.

### The LCP Code-Splitting Invariant:
> **NEVER lazy-load above-the-fold critical LCP rendering just because the module is large.**
>
> If a 300KB component renders the hero headline, hero image, or primary visual fold, deferring it behind an asynchronous chunk guarantees an LCP catastrophe:
> 1. HTML loads → 2. Main JS loads → 3. Main JS executes → 4. Dynamic chunk request dispatches → 5. Chunk loads → 6. Chunk executes → 7. LCP finally renders.
>
> That multi-step waterfall will destroy your LCP metric even if the initial bundle looks smaller on paper.

### Appropriate Candidates for Code-Splitting:
- Heavy below-the-fold sections (customer review carousels, detailed spec sheets).
- Modals, dialogs, drawers, and overlay sheets.
- Complex charting, analytics, and data visualization libraries.
- Rich-text editors and syntax highlighters.
- Non-hero 3D canvases and interactive experiences loaded after initial scroll.
- Route-level page boundaries in SPAs.

---

## 4. Main-Thread Cooperative Scheduling

Break up long synchronous loops and intensive initialization using cooperative scheduling:

```javascript
// Yield execution to allow browser paint and input processing
async function yieldToMain() {
  if ('scheduler' in window && 'yield' in scheduler) {
    return await scheduler.yield();
  }
  return new Promise((resolve) => setTimeout(resolve, 0));
}

async function processLargeDataSet(items) {
  for (let i = 0; i < items.length; i++) {
    processItem(items[i]);
    // Yield every 50 items to keep main thread responsive
    if (i % 50 === 0) {
      await yieldToMain();
    }
  }
}
```

---

## 5. Lifecycle Hygiene & Memory Leaks

- **Event Listeners**: Every `addEventListener` in a component lifecycle hook must have a matching `removeEventListener` in the cleanup callback.
- **Timers**: Clear all `setInterval` and `setTimeout` timers on unmount.
- **AbortController**: Use `AbortController` to cancel in-flight HTTP requests when components unmount, preventing state updates on dead component trees.

---

## 6. Third-Party Script Management

Third-party tags (analytics, tag managers, chat widgets) are a primary driver of unmanaged Long Tasks:
- Use facade patterns for heavy non-critical widgets (e.g. render a lightweight chat launcher button, downloading the full 1MB chat SDK only when the user clicks).
- Load analytics scripts with `defer` or `async`, or initialize them during idle time using `requestIdleCallback`.
