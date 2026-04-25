# Style Gaps

All issues below were identified by auditing the CSS files and resolved in commit `0b7c211`.

---

| # | Issue | File(s) | Resolution |
|---|-------|---------|------------|
| 1 | 14 hardcoded values not using tokens | `gantt.css`, `components.css`, `layout.css` | Added `--color-border-subtle`, `--color-gantt-now`, `--color-danger-dark`, `--color-gold`, `--font-size-pill`, `--font-size-modal-header`, `--pill-gap/padding/dot-size`; replaced every raw hex and raw `px` with token references |
| 2 | `--space-4: 15px` breaks the spacing scale | `tokens.css` | Changed to `16px` — scale is now 4 8 12 16 20 24 30 40 60 |
| 3 | Two token pairs sharing the same value with no explanation | `tokens.css` | Added a block comment explaining that `--color-bg-page`/`--color-bg-header` and `--color-text-primary`/`--color-border-strong` are intentionally separate semantic roles |
| 4 | Dead tokens never referenced in any CSS rule | `tokens.css` | Removed `--color-vsm-warning-bg`, `--color-vsm-danger-bg`; gave `--font-size-h3` a real value (`1.17em`) and wired it to `h3` in `base.css`; removed OTIF chart colour tokens (JS-only, no CSS consumer) |
| 5 | `.isotope-section h2` duplicated base `h2` rule exactly | `layout.css` | Removed the block entirely — the base rule in `base.css` already applies |
| 6 | `--z-gantt-labels` and `--z-gantt-block` both `10`; `.gantt-grid-lines` used raw `z-index: 1` | `tokens.css`, `gantt.css` | Set `--z-gantt-labels: 20`, added `--z-gantt-grid: 1`, replaced raw value in `gantt.css` |
| 7 | `--font-size-h3: inherit` misleading; `h3` had no font-size or font-weight | `tokens.css`, `base.css` | Token set to `1.17em` (browser default); `h3` in `base.css` now declares `font-size: var(--font-size-h3)` and `font-weight: var(--font-weight-bold)` |
| 8 | `fadeIn` and `slideUp` keyframes declared but never used | `animations.css` | Removed both |
| 9 | `.btn--outline` hover had no easing; base `.btn` used `border: none` conflicting with outline border | `components.css` | Base `.btn` now uses `border: 2px solid transparent`; added `background` and `color` to the shared transition |
| 10 | `.modal-overlay` base rule set `overflow: auto` leaked into fullscreen variant; `.is-open` logic was non-obvious | `components.css` | `overflow: auto` moved onto `.modal-overlay.is-open` only; fullscreen `.is-open` sets `overflow: hidden` explicitly; removed the `:not()` selector |
| 11 | `.section-subtitle` used raw negative margin `-8px` | `components.css` | Changed to `calc(-1 * var(--space-2))` |
| 12 | Gantt row/block heights (`70px`/`60px`) hardcoded in CSS with no tokens | `tokens.css`, `gantt.css` | Added `--gantt-row-height`, `--gantt-block-height`, `--gantt-block-inset`, `--gantt-header-height`, `--gantt-label-width`; all four gantt files now reference these |
| 13 | `tokens.css` imported five times — once per partial | all partials, `dashboard.css` | Removed `@import url('tokens.css')` from all partials; `dashboard.css` imports tokens once at the top, then each partial in order |
| 14 | No `line-height` tokens | `tokens.css`, `base.css`, `components.css` | Added `--line-height-tight`, `--line-height-base`, `--line-height-loose`; applied to `body`, `h1`–`h3`, and key components |
| 15 | No `font-weight` tokens; `bold` and `600` used interchangeably | `tokens.css`, all files | Added `--font-weight-normal`, `--font-weight-semibold`, `--font-weight-bold`; replaced every raw `font-weight` value across all files |
