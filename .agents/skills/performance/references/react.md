# Evidence-Based React Performance Reference Manual

## 1. Prime Rule: Never Blindly Add Memoization

Do **not** reflexively wrap components in `React.memo`, or functions in `useCallback`, or objects in `useMemo` without profiling evidence.
- Memoization introduces computational overhead: dependency comparison on every render, closure allocation, and memory retention.
- Wrapping a component in `React.memo` that renders lightweight DOM or receives unstable props on every render adds comparison cost without saving render work.
- **Rule**: Apply memoization only when React DevTools Profiler or source inspection confirms that an expensive component subtree is re-rendering with identical props.

---

## 2. Diagnosing Unnecessary Renders

Identify the real root causes of re-render cascades:

### Unstable References Defeating Memoization:
```jsx
// ❌ Inline object literal creates a fresh reference on every render
<ChildComponent options={{ theme: 'dark' }} />

// ✅ Stable reference preserves memoization
const options = useMemo(() => ({ theme: 'dark' }), []);
<ChildComponent options={options} />
```

### Context Invalidation Blast Radius:
Passing an inline object to a Context Provider causes every consumer of that context to re-render whenever the provider renders:
```jsx
// ❌ Re-renders all consumers on any parent state change
<UserContext.Provider value={{ user, theme }}>

// ✅ Only re-renders consumers when actual values change
const contextValue = useMemo(() => ({ user, theme }), [user, theme]);
<UserContext.Provider value={contextValue}>
```
Better yet: Split contexts with different update frequencies (e.g. separate `UserContext` from `ThemeContext` or `NotificationsContext`).

---

## 3. Expensive Work in Render Bodies

- Heavy calculations (filtering/sorting arrays of 1000+ items, complex data mapping, deep cloning) running directly in component render bodies block the main thread on every tick.
- Wrap genuinely expensive computations in `useMemo`:
  ```jsx
  const sortedItems = useMemo(() => {
    return items.filter(Boolean).sort((a, b) => b.score - a.score);
  }, [items]);
  ```
- Avoid `JSON.parse(JSON.stringify(obj))` for cloning. Use native `structuredClone()` or shallow spread where appropriate.

---

## 4. Effect Loops & Lifecycle Hygiene

- **Effect Dependency Loops**: An effect that updates state which in turn re-triggers the same effect causes infinite re-render loops or continuous main-thread burning. Always verify dependency arrays.
- **Uncleaned Listeners**:
  ```jsx
  useEffect(() => {
    const handleResize = () => {/* ... */};
    window.addEventListener('resize', handleResize);
    // Mandatory cleanup:
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  ```
- **Timers & Subscriptions**: Clear all `setInterval`, `setTimeout`, and WebSocket/event emitter subscriptions in effect cleanup returns.

---

## 5. Component Splitting & The LCP Invariant

- **Split God Components**: Isolate rapidly updating state (e.g. form inputs, search queries, sliders) into localized leaf components so updates don't re-render the entire page tree.
- **Code Splitting with Suspense**:
  ```jsx
  const AnalyticsModal = React.lazy(() => import('./AnalyticsModal'));
  <Suspense fallback={<ModalSkeleton />}>
    {isOpen && <AnalyticsModal />}
  </Suspense>
  ```
- **The Invariant**: **Never lazy-load critical above-the-fold or hero LCP content.** Deferring the LCP candidate behind an asynchronous JavaScript chunk causes a multi-step network waterfall that directly damages LCP.

---

## 6. List Keys & Reconciliation Stability

- **Never use array index or random numbers as keys** for dynamic or reorderable lists:
  ```jsx
  // ❌ Unmounts and remounts all children on reorder/filter
  {items.map((item, index) => <Card key={index} data={item} />)}

  // ✅ React reconciles existing DOM nodes accurately
  {items.map((item) => <Card key={item.id} data={item} />)}
  ```
  Unstable keys force React to destroy and recreate the full component DOM subtree, wiping focus, internal state, and running animations.
