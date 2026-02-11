# UX Anti-Patterns Skill — Evals

Evals derived from the skill's own content and the writing-skills framework.
Each eval describes a scenario, expected behavior, and pass criteria.

---

## Category 1: Discovery & CSO (Claude Search Optimization)

### EVAL-1.1: Skill triggers on frontend code review request

**Scenario:** User asks: "Review this React component for UX issues" with a component containing a form with no loading states and a disabled button with no explanation.
**Expected:** Agent loads the ux-antipatterns skill based on description match.
**Pass criteria:** Skill is selected and loaded without user explicitly naming it.

### EVAL-1.2: Skill triggers on new UI feature build

**Scenario:** User asks: "Build me a search page with filters and infinite scroll."
**Expected:** Agent loads the skill during or after implementation for self-review.
**Pass criteria:** Agent references anti-pattern heuristics while building, or runs a review pass after.

### EVAL-1.3: Skill does NOT trigger for non-frontend tasks

**Scenario:** User asks: "Write a Python script to parse CSV files."
**Expected:** Skill is NOT loaded.
**Pass criteria:** Skill remains unloaded; no UX anti-pattern analysis attempted.

### EVAL-1.4: Keyword coverage catches symptom-based queries

**Scenario:** User says: "My users are complaining about double-submitting orders."
**Expected:** Agent recognizes this maps to anti-pattern 9.1 (duplicate submission).
**Pass criteria:** Agent identifies the correct anti-pattern and provides the right fix (disable on click + server-side idempotency).

---

## Category 2: Workflow Compliance

### EVAL-2.1: Agent reads reference file before scanning

**Scenario:** Agent is given a codebase to review for UX anti-patterns.
**Expected:** Agent reads `references/antipatterns.md` as step 1 of the workflow.
**Pass criteria:** Agent loads the reference file before reporting any findings.

### EVAL-2.2: Findings cite specific file:line locations

**Scenario:** Agent reviews a multi-file React app with 3+ anti-patterns present.
**Expected:** Every finding includes `file_path:line_number`.
**Pass criteria:** All reported findings include specific file and line references, not vague descriptions.

### EVAL-2.3: Findings include all three required elements

**Scenario:** Agent finds an anti-pattern in code.
**Expected:** Each finding states: (1) the anti-pattern name, (2) the user harm, (3) a concrete fix.
**Pass criteria:** All three elements present in every reported finding. Not just "this is wrong" — explains harm and fix.

### EVAL-2.4: Findings grouped by anti-pattern category

**Scenario:** Agent reviews code with issues spanning 4+ categories.
**Expected:** Output is organized by category, not file-by-file.
**Pass criteria:** Findings are grouped under anti-pattern headings, not scattered.

---

## Category 3: Detection Accuracy

### EVAL-3.1: Detects missing loading state on async action

**Scenario:** Button click handler fires `fetch()` with no UI state change (no spinner, no disabled state).
```jsx
<button onClick={() => fetch('/api/submit', { method: 'POST', body })}>
  Submit
</button>
```
**Expected:** Flags anti-pattern 2.1 (No immediate feedback on action).
**Pass criteria:** Identifies the violation, explains that users won't know if their click registered, suggests adding loading/disabled state.

### EVAL-3.2: Detects paste-blocking on input

**Scenario:** Input field with `onPaste={(e) => e.preventDefault()}`.
**Expected:** Flags anti-pattern 4.1 (Paste blocked).
**Pass criteria:** Identifies the violation and recommends removing the paste prevention.

### EVAL-3.3: Detects disabled button with no explanation

**Scenario:**
```html
<button disabled>Submit</button>
```
No tooltip, no adjacent text, no `aria-describedby`.
**Expected:** Flags anti-pattern 3.2 (Disabled controls with no explanation).
**Pass criteria:** Identifies the violation and suggests adding explanatory text with proper ARIA attributes.

### EVAL-3.4: Detects stale response race condition

**Scenario:** Search input that fires fetch on every keystroke with no AbortController or sequence tracking.
**Expected:** Flags anti-pattern 9.2 (Stale response overwriting newer intent).
**Pass criteria:** Identifies the race condition, explains the "appl" / "apple" scenario, suggests AbortController or request sequencing.

### EVAL-3.5: Detects focus indicator removal

**Scenario:** Global CSS: `*:focus { outline: none; }` with no `:focus-visible` replacement.
**Expected:** Flags anti-pattern 10.1 (Focus indicators removed).
**Pass criteria:** Identifies the violation and recommends `:focus-visible` approach.

### EVAL-3.6: Detects images without dimensions (layout shift)

**Scenario:** `<img src={asyncUrl} />` with no `width`, `height`, `aspect-ratio`, or container constraint.
**Expected:** Flags anti-pattern 1.1 (Elements that shift after render).
**Pass criteria:** Identifies layout shift risk and recommends explicit dimensions or aspect-ratio.

### EVAL-3.7: Detects multi-step form losing state on back

**Scenario:** Wizard component where step state is local (`useState`) and navigating back unmounts previous step.
**Expected:** Flags anti-pattern 4.6 (Multi-step forms that lose data on back-navigation).
**Pass criteria:** Identifies the data loss risk and suggests session storage, URL params, or global state.

### EVAL-3.8: Detects 100vh usage on mobile layout

**Scenario:** Hero section styled with `height: 100vh`.
**Expected:** Flags anti-pattern 12.2 (100vh jitter on mobile).
**Pass criteria:** Identifies the mobile jitter issue and recommends `100dvh` or `100svh`.

### EVAL-3.9: Detects hover-only controls

**Scenario:** Edit button that only appears on `:hover` with no `:focus` or `:focus-within` equivalent.
**Expected:** Flags anti-pattern 10.2 (Hover-only information or controls).
**Pass criteria:** Identifies that touch and keyboard users can't access the control.

### EVAL-3.10: Detects double-submit vulnerability

**Scenario:** Form submit handler that doesn't disable the button or guard against rapid re-invocation.
**Expected:** Flags anti-pattern 9.1 (Duplicate submission).
**Pass criteria:** Recommends both client-side disable AND server-side idempotency keys.

---

## Category 4: Axiom Adherence

### EVAL-4.1: Axiom 1 — Acknowledge every action

**Scenario:** Agent reviews code with a delete button that fires an API call but shows no feedback until the item disappears 2 seconds later.
**Expected:** Agent flags this as violating Axiom 1 (visible feedback within 100ms).
**Pass criteria:** References the axiom and explains the 100ms feedback rule.

### EVAL-4.2: Axiom 2 — Never destroy user input

**Scenario:** Form that clears all fields on a validation error.
**Expected:** Agent flags this as violating Axiom 2.
**Pass criteria:** Identifies that user input is destroyed and recommends preserving input while showing errors inline.

### EVAL-4.3: Axiom 4 — Most recent intent wins

**Scenario:** Autocomplete that doesn't cancel previous requests when new input arrives.
**Expected:** Agent flags this as violating Axiom 4.
**Pass criteria:** Explains that stale results could overwrite the user's newer intent.

### EVAL-4.4: Axiom 6 — Don't fight the platform

**Scenario:** Custom select dropdown that doesn't support keyboard navigation or native accessibility patterns.
**Expected:** Agent flags this as violating Axiom 6.
**Pass criteria:** Recommends following WAI-ARIA patterns or using native elements.

---

## Category 5: False Positive Resistance

### EVAL-5.1: Does NOT flag proper loading state implementation

**Scenario:** Button with `disabled={isLoading}` and a spinner shown during the async operation.
**Expected:** No violation flagged for feedback/responsiveness.
**Pass criteria:** Agent recognizes this as a correct implementation and does not flag it.

### EVAL-5.2: Does NOT flag modal that traps focus correctly

**Scenario:** Modal with Escape to close, backdrop click to close, visible close button, and proper focus trap that releases on close.
**Expected:** No violation flagged for keyboard traps or modal dismissal.
**Pass criteria:** Agent recognizes correct modal implementation.

### EVAL-5.3: Does NOT flag intentionally disabled button with explanation

**Scenario:**
```html
<button disabled aria-describedby="hint">Submit</button>
<p id="hint">Complete all required fields to submit.</p>
```
**Expected:** No violation flagged.
**Pass criteria:** Agent recognizes the explanation satisfies anti-pattern 3.2.

### EVAL-5.4: Does NOT flag focus-visible with custom ring

**Scenario:** CSS that removes outline on `:focus:not(:focus-visible)` but provides a visible ring on `:focus-visible`.
**Expected:** No violation flagged for focus indicator removal.
**Pass criteria:** Agent recognizes this as the recommended pattern.

---

## Category 6: Coverage Completeness

### EVAL-6.1: All 13 categories are scannable

**Scenario:** Agent is given a large codebase that contains at least one violation from every category.
**Expected:** Agent scans and reports findings across all 13 categories.
**Pass criteria:** No category is systematically skipped or ignored.

### EVAL-6.2: Agent handles clean code gracefully

**Scenario:** Agent reviews well-built code with no UX anti-patterns.
**Expected:** Agent reports that no anti-patterns were detected rather than inventing issues.
**Pass criteria:** Agent does not manufacture false positives to appear thorough.

---

## Category 7: Fix Quality

### EVAL-7.1: Fixes are concrete and implementable

**Scenario:** Agent finds 3 anti-patterns in a component.
**Expected:** Each fix is specific enough to implement (code snippet, attribute to add, pattern to follow) — not just "improve error handling."
**Pass criteria:** A developer could implement the fix directly from the agent's output without further research.

### EVAL-7.2: Fixes don't introduce new anti-patterns

**Scenario:** Agent suggests a fix for a race condition in search.
**Expected:** Fix doesn't introduce new issues (e.g., suggesting a loading state that steals focus).
**Pass criteria:** Suggested fix is reviewed against the axioms and doesn't violate them.

### EVAL-7.3: Fixes recommend server-side measures where appropriate

**Scenario:** Agent detects double-submit vulnerability.
**Expected:** Fix recommends BOTH client-side guard AND server-side idempotency.
**Pass criteria:** Agent doesn't suggest client-side-only solutions for problems that need server-side defense.
