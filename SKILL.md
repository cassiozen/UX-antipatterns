---
name: ux-antipatterns
description: "Detects UX anti-patterns in frontend code — layout shifts, missing loading states, broken form inputs, focus traps, race conditions, and accessibility failures. Use when reviewing or building React, Vue, Svelte, or HTML/CSS components, pages, forms, or interactive flows in pull requests and feature branches."
---

# UX Anti-Pattern Detection

Scan frontend code for patterns that cause user frustration. 

## Core Axioms

Before checking individual rules, internalize these. They are the "why" behind every item below.

| # | Axiom | One-liner |
|---|-------|-----------|
| 1 | **Acknowledge every action** | Every user action must produce visible feedback within 100ms, even if the result takes seconds. |
| 2 | **Never destroy user input** | Not on error, not on navigation, not on timeout, not on refresh. |
| 3 | **State survives the unexpected** | Refresh, double-clicks or double submits, network loss — code must handle edge cases. |
| 4 | **Most recent intent wins** | Stale responses must never overwrite a newer user action. |
| 5 | **Explain every constraint** | If it's disabled, say why. If it failed, say how to fix it. If it succeeded, say what happened. |
| 6 | **Don't fight the platform** | Browser conventions, OS gestures, native controls, and accessibility APIs encode billions of hours of UX research. |

## When NOT to Use

- Backend-only code with no UI layer
- CLI tools or non-visual interfaces
- Design system tokens/docs without implementation code
- Pure API or data-layer reviews
- Performance profiling (unless it manifests as a UX symptom like layout shift)

## Workflow

1. Read [references/antipatterns.md](references/antipatterns.md) to load the full detection heuristics.
2. Scan the code under review against each applicable anti-pattern category.
3. Report findings grouped by anti-pattern, citing specific file:line locations.
4. For each finding, state: the anti-pattern name, the user harm, and a concrete fix.
5. If no anti-patterns are found, state that the code is clean rather than manufacturing findings.

## Anti-Pattern Categories

| # | Category | User Harm |
|---|----------|-----------|
| 1 | Layout Stability | Click target moves; wrong thing clicked. |
| 2 | Feedback & Responsiveness | Action feels ignored; user retries, waits, or loses trust that the system is working. |
| 3 | Error Handling & Recovery | User is stuck with no way forward; input destroyed; problem unsolvable without guessing. |
| 4 | Forms & Input Interference | Platform fights the user's typing; data mangled, basic editing broken. |
| 5 | Focus | User is typing and the UI yanks them elsewhere. |
| 6 | Notifications, Interruptions & Dialogs | User's flow broken; attention taxed by noise; forced to parse ambiguous choices under pressure. |
| 7 | Navigation, Routing & State Persistence | User can't go back; context evaporates on refresh or redirect. |
| 8 | Scroll & Viewport | Content unreachable or unstable; user fights the interface to see what they came for. |
| 9 | Timing, Debounce & Race Conditions | Actions fire twice, responses arrive stale, sessions expire mid-task; system behaves unpredictably under normal use. |
| 10 | Accessibility as UX | Entire interaction modes broken — keyboard users can't navigate, touch users locked out. |
| 11 | Visual Layering & Rendering | UI elements overlap, clip, or hide each other; controls become unreachable. |
| 12 | Mobile & Viewport-Specific | Keyboard covers input, layout jumps on scroll, tap targets unresponsive; basic mobile interaction degraded. |
| 13 | Cumulative Decay & Long-Term UX | App degrades over time; preferences lost, performance rots, stale experiments create inconsistencies. |

## Examples

Common anti-patterns and their fixes:

```jsx
// VIOLATION: submit button with no loading state (Category 2)
<button onClick={() => submitForm(data)}>Submit</button>

// FIX: disable and show feedback during async operation
<button onClick={handleSubmit} disabled={isSubmitting}>
  {isSubmitting ? "Submitting..." : "Submit"}
</button>
```

```jsx
// VIOLATION: immediate destructive action, no confirmation (Category 6)
<button onClick={() => deleteItem(id)}>Delete</button>

// FIX: confirm before irreversible action
<button onClick={() => setConfirmDelete(true)}>Delete</button>
{confirmDelete && (
  <dialog open>
    <p>Delete this item? This cannot be undone.</p>
    <button onClick={() => setConfirmDelete(false)}>Cancel</button>
    <button onClick={() => { deleteItem(id); setConfirmDelete(false); }}>
      Delete permanently
    </button>
  </dialog>
)}
```

```js
// VIOLATION: stale search response overwrites newer results (Category 9)
async function search(query) {
  const res = await fetch(`/search?q=${query}`);
  setResults(await res.json());
}

// FIX: abort previous request to prevent race condition
let controller;
async function search(query) {
  controller?.abort();
  controller = new AbortController();
  const res = await fetch(`/search?q=${query}`, { signal: controller.signal });
  setResults(await res.json());
}
```

## Common Mistakes

- **Flagging style preferences as anti-patterns.** A non-standard button shape is a design choice, not a UX violation. Only flag patterns that cause measurable user harm per the axioms.
- **Ignoring context.** A disabled button inside a wizard step IS explained by the wizard's own flow. Check for nearby explanatory elements before reporting.
- **Suggesting fixes that break accessibility.** A fix that adds a visual indicator but removes keyboard access trades one violation for another. Verify fixes against Axiom 6.
- **Over-reporting on handled edge cases.** If the code already has an AbortController, don't flag it for race conditions. Read the implementation before reporting.
- **Reporting framework internals as violations.** React's `key` prop remounts, Next.js loading states, or SvelteKit form actions may handle anti-patterns at the framework level. Understand the framework before flagging.
