# Style Gaps & Opportunities

Identified by auditing every CSS rule, inline style, and colour literal in `renderer/dashboard_full.py`, `renderer/assets.py`, `renderer/helpers.py`, and `renderer/tables.py`.

---

## 1. Inline styles everywhere

**Problem:** Almost every element in the renderer uses `style=""` attributes instead of class-based CSS.  This makes theming changes require touching Python source, not just a stylesheet.

**Opportunity:** Replace `style="..."` attributes with semantic utility classes defined in `components.css` / `layout.css`.  The token system in `tokens.css` enables one-line rebranding.

---

## 2. No responsive / mobile breakpoints

**Problem:** No `@media` rules exist.  The dashboard assumes a wide viewport.  `.chart-row` (2-col) and `.chart-row-three` (3-col) collapse badly on narrow screens.

**Opportunity:**
```css
@media (max-width: 1024px) {
  .chart-row-three { grid-template-columns: 1fr 1fr; }
}
@media (max-width: 640px) {
  .chart-row,
  .chart-row-three { grid-template-columns: 1fr; }
  .container { padding: var(--space-3); }
}
```

---

## 3. No focus / accessibility styles

**Problem:** Interactive elements (buttons, clickable BOs, ploeg links, storingen rows) have no `:focus` or `:focus-visible` styles.  Keyboard navigation is invisible.

**Opportunity:**
```css
:focus-visible {
  outline: 3px solid var(--color-brand-pink);
  outline-offset: 2px;
  border-radius: var(--radius-sm);
}
```

---

## 4. No print stylesheet

**Problem:** The dashboard has no `@media print` rules.  Printing produces backgrounds cut off, nav buttons visible, modals potentially shown.

**Opportunity:**
```css
@media print {
  .btn, .heartbeat-row, .modal-overlay { display: none !important; }
  body { background: white; padding: 0; }
  .container { box-shadow: none; max-width: 100%; }
  .chart-container { page-break-inside: avoid; }
}
```

---

## 5. No dark mode support

**Problem:** No `@media (prefers-color-scheme: dark)` block.  The dashboard is hardcoded to a white background.

**Opportunity:** Use CSS custom properties (already in `tokens.css`) to override colour variables:
```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-page:    #1a1a2e;
    --color-bg-card:    #16213e;
    --color-bg-section: #0f3460;
    --color-text-primary: #e0e0e0;
    ...
  }
}
```

---

## 6. Inconsistent section heading borders

**Problem:** Three different border styles are used for `h2` across the dashboard:
- `border-image: linear-gradient(to right, #662678, #E40D7E) 1` — most sections
- `border-bottom: 3px solid #FF5722` — "Lopende week" summary table
- `border-bottom: 3px solid #9C27B0` — "Afgelopen week" and storingen tables

There is no documented rationale for the colour difference.  The `accent--running` and `accent--previous` modifier classes in `base.css` capture this but a decision should be made about whether this distinction should be kept.

---

## 7. No loading / skeleton state

**Problem:** When the dashboard is regenerating, the page reloads fully with no intermediate state.  A blank white flash occurs.

**Opportunity:** Add a `.skeleton` class with an animated shimmer for chart canvases while Chart.js initialises:
```css
.skeleton {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: var(--radius-md);
}
@keyframes shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

---

## 8. No toast / notification component

**Problem:** There is no non-blocking feedback mechanism.  Errors (OTIF parse fail, storingen lookup) are silently swallowed or printed to the Python console only.  The user sees nothing.

**Opportunity:** Add a `.toast` component (bottom-right fixed) driven by a small JS helper.

---

## 9. Isotope colours only in JavaScript

**Problem:** Gantt isotope colours (`#FFDE21` Thallium, `#0000ff` Gallium, etc.) exist only in the JS `isotopeColors` object inside `gantt.py`.  They are not available as CSS variables and cannot be referenced from stylesheets (e.g. for a legend or a colour-coded table).

**Opportunity:** Add to `tokens.css`:
```css
--color-isotope-thallium: #FFDE21;
--color-isotope-gallium:  #0000ff;
/* etc. */
```
(already done in `tokens.css`) and read them from JS using `getComputedStyle(document.documentElement).getPropertyValue('--color-isotope-gallium')`.

---

## 10. No utility / spacing classes

**Problem:** Spacing (`margin-left: 20px`, `margin-top: 10px`, etc.) is all inline.  There is no reusable spacing utility layer.

**Opportunity:** Add a thin utility layer:
```css
.mt-1 { margin-top: var(--space-1); }
.ml-5 { margin-left: var(--space-5); }
/* etc. */
```

---

## 11. Modal open/close state uses inline `display` toggling

**Problem:** JavaScript sets `el.style.display = 'flex'` / `'none'` directly.  This bypasses CSS and makes CSS-driven enter/exit animations impossible.

**Opportunity:** Use a `.is-open` class (already reflected in `components.css`) toggled by JS, so transitions can be applied in CSS:
```css
.modal-overlay {
  opacity: 0;
  pointer-events: none;
  transition: opacity var(--transition-base);
}
.modal-overlay.is-open {
  opacity: 1;
  pointer-events: auto;
}
```

---

## 12. No error / empty-state component

**Problem:** Empty tables fall back to `<td>No data available</td>` with no visual treatment.  There is no `.empty-state` class with an icon, heading, and sub-text.

**Opportunity:**
```css
.empty-state {
  text-align: center;
  padding: var(--space-9) var(--space-5);
  color: var(--color-text-muted);
}
.empty-state__icon { font-size: 48px; margin-bottom: var(--space-3); }
.empty-state__text { font-size: var(--font-size-body); }
```

---

## 13. Font `ISOCPEUR` has no fallback definition

**Problem:** `h1` and `h2` use `'ISOCPEUR'` as the first font choice, but there is no `@font-face` declaration.  If the font is not installed locally the browser silently falls back to `'Courier New'` — an acceptable but undocumented behaviour.

**Opportunity:** Either ship the font as a web font (`.woff2`) and declare it with `@font-face`, or document the fallback chain explicitly.

---

## 14. `font-size: 25px` / `15px` are magic numbers

**Problem:** Cell value sizes (`25px` for KPI values, `15px` for BO labels) are hardcoded inline throughout helpers and dashboard_full.  They are not tied to any type scale.

**Opportunity:** Use `--font-size-label-lg: 25px` and `--font-size-bo: 15px` from `tokens.css` (already defined) and apply them via classes.

---

## 15. No consistent border-radius on storingen / OTIF inline tables

**Problem:** Some tables have `border-radius: 4px` on their wrapper, others have none.  The Gantt wrapper uses `5px`, the card uses `8px`, the container uses `10px`.

**Opportunity:** Standardise on the scale in `tokens.css`: `--radius-sm: 3px`, `--radius-md: 5px`, `--radius-lg: 8px`, `--radius-xl: 10px`.

---

## Summary table

| # | Gap | Severity | Effort |
|---|-----|----------|--------|
| 1 | Inline styles everywhere | High | High |
| 2 | No responsive breakpoints | High | Medium |
| 3 | No focus/a11y styles | High | Low |
| 4 | No print stylesheet | Medium | Low |
| 5 | No dark mode | Low | Medium |
| 6 | Inconsistent section borders | Low | Low |
| 7 | No skeleton loader | Low | Medium |
| 8 | No toast component | Medium | Medium |
| 9 | Isotope colours JS-only | Medium | Low |
| 10 | No utility classes | Low | Low |
| 11 | Modal state via inline JS | Medium | Low |
| 12 | No empty-state component | Low | Low |
| 13 | ISOCPEUR not declared | Medium | Low |
| 14 | Magic font-size numbers | Low | Low |
| 15 | Inconsistent border-radius | Low | Low |
