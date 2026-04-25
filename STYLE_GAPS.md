# Style Gaps

Actual gaps found by reading every rule in the codebase. Each item is a concrete problem in the files as they stand today.

---

## 1. Hardcoded values that escaped tokenisation

Several raw values slipped through and are not referencing `tokens.css`:

| File | Selector | Property | Raw value | Should be |
|------|----------|----------|-----------|-----------|
| `gantt.css` | `.gantt-grid-line` | `background-color` | `#e0e0e0` | new token `--color-border-subtle` |
| `gantt.css` | `.gantt-block-label` | `color` | `#000` | `var(--color-text-primary)` |
| `gantt.css` | `.gantt-now-line`, `.gantt-now-label` | `background-color` | `#ff0000` | new token `--color-gantt-now` |
| `components.css` | `.tampering-banner` | `border-color` | `#8B0000` | new token `--color-danger-dark` |
| `components.css` | `.winner-card__detail` | `color` | `#FFD700` | new token `--color-gold` |
| `components.css` | `.kpi-strip__item--neutral` | `color` | `black` | `var(--color-text-primary)` (`#333`, not `#000`) |
| `components.css` | `.modal-header h2` | `font-size` | `22px` | new token or `var(--font-size-body)` |
| `components.css` | `.pill` | `padding` | `6px 21px` | token values |
| `components.css` | `.pill` | `font-size` | `20px` | new token `--font-size-pill` |
| `components.css` | `.pill` | `gap` | `9px` | token value |
| `layout.css` | `.heartbeat-row` | `gap` | `10px` | `var(--space-3)` or add `--space-2-5` |
| `components.css` | `.modal-header--bar` | `padding` | `var(--space-4) 25px` | `25px` is not in the scale |
| `components.css` | `.winner-card__detail` | `margin` | `5px 0` | `5px` is not in the spacing scale |
| `components.css` | `.kpi-strip__item + .kpi-strip__item` | `margin-left` | `20px` | `var(--space-5)` |

---

## 2. Spacing scale has a broken step

`tokens.css` defines:

```
--space-1:  4px
--space-2:  8px
--space-3: 12px
--space-4: 15px   ← breaks the pattern
--space-5: 20px
```

`--space-4` is `15px`, not `16px`. Every other step doubles or adds 4–8px consistently. `15px` is an odd value and causes callers to reach for raw `px` values when they need something between `12px` and `20px`. Either change it to `16px` or add explicit documentation explaining why it is `15px`.

---

## 3. Two tokens share the same value

`--color-bg-page: #f5f5f5` and `--color-bg-header: #f5f5f5` are identical. Same for `--color-text-primary: #333333` and `--color-border-strong: #333333`. Using separate tokens for the same hex is fine only if they are *semantically* distinct and may diverge. There is no comment explaining the intent, so a future editor may change one and not the other, silently breaking both.

---

## 4. Dead tokens — defined but never referenced in any CSS rule

`tokens.css` defines these; no rule in the codebase uses them:

- `--color-vsm-warning-bg: #fef9e7`
- `--color-vsm-danger-bg:  #fadbd8`
- `--font-size-h3: inherit`
- `--color-otif-*` (all six OTIF chart colours) — these are for Chart.js datasets, not CSS rules, but they sit in the CSS token file with no consumer

---

## 5. `.isotope-section h2` duplicates the base `h2` rule exactly

`layout.css` redeclares `color`, `border-bottom`, `border-image`, `padding-bottom`, and `margin-bottom` on `.isotope-section h2`. Every one of those declarations is already set identically by the `h2` rule in `base.css`. It adds nothing and will silently diverge if the base `h2` is ever updated.

---

## 6. `--z-gantt-labels` and `--z-gantt-block` are both `10`

```css
--z-gantt-labels: 10;
--z-gantt-block:  10;
```

The label column needs to sit above production blocks so labels are not obscured when blocks are wide. Two different elements sharing a z-index token means stacking order is decided by DOM order alone, which is fragile. Also `.gantt-grid-lines` uses raw `z-index: 1` with no token at all.

---

## 7. `--font-size-h3: inherit` is misleading

The token is declared in `tokens.css` but `h3` in `base.css` has no `font-size` declaration, so the token is never applied. The comment next to it says `/* browser default + bold */` but `base.css` also adds no `font-weight` to `h3`. The comment is wrong on both counts.

---

## 8. `animations.css` declares two keyframes that are never used

`fadeIn` and `slideUp` are defined but no rule in any file references them. They were added speculatively. Either wire them up or remove them to keep the file honest.

---

## 9. `.btn--outline` inherits a `border: none` from `.btn` then re-adds a border

`.btn` sets `border: none`. `.btn--outline` then sets `border: 2px solid var(--color-brand-pink)`. This works, but the `transition` on `.btn` only covers `transform` and `box-shadow`. The background and color change on `.btn--outline:hover` is instant — no easing — while the lift effect from `.btn--primary:hover` is eased. The two variants behave inconsistently on hover.

---

## 10. `.modal-overlay` base rule conflicts with `.modal-overlay--fullscreen`

`.modal-overlay` sets `display: none` and `overflow: auto`. The fullscreen variant needs `display: flex` and no overflow. The current approach uses two separate `.is-open` selectors:

```css
.modal-overlay--fullscreen.is-open           { display: flex; }
.modal-overlay:not(.modal-overlay--fullscreen).is-open { display: block; }
```

The `:not()` selector is non-obvious and will silently break if a third modal variant is added. The `overflow: auto` from the base rule is also never cancelled for the fullscreen variant, so the fullscreen overlay is scrollable when it should not be.

---

## 11. `.section-subtitle` uses a raw negative margin

```css
margin-top: -8px;
```

`-8px` is not expressed with a token and negative margins are invisible to the spacing scale. This makes it impossible to know from reading `tokens.css` alone what the vertical rhythm of a subtitle relative to its `h2` is.

---

## 12. Gantt row and block heights are magic numbers with no tokens

`.gantt-label-item`, `.gantt-row`, and `.gantt-block` all hard-code `70px` / `60px` heights. These values almost certainly appear in the JavaScript too (block positioning math). If the row height ever changes, it must be updated in both CSS and JS separately. They should be tokens:

```css
--gantt-row-height:   70px;
--gantt-block-height: 60px;  /* row-height minus 2×5px top/bottom inset */
```

---

## 13. `dashboard.css` causes `tokens.css` to be imported five times

Each partial (`base.css`, `layout.css`, `components.css`, `gantt.css`) individually `@import url('tokens.css')`. When `dashboard.css` is used as the entry point it imports all five partials, so `tokens.css` is fetched five times. Modern browsers deduplicate `@import`, but CSS bundlers and linters may not, and the intent is ambiguous — either the partials should be self-contained (and `dashboard.css` should not re-import `tokens.css`), or they should rely on `dashboard.css` to load tokens first.

---

## 14. No `line-height` tokens

Typography tokens define font sizes and letter spacing but no line heights. Several components have tight or tall text that depends on browser defaults, which differ across engines. `.gantt-label-item` uses flexbox centering to work around missing line-height; `.pill` uses `align-items: center`. A `--line-height-base`, `--line-height-tight`, and `--line-height-loose` would make the scale complete.

---

## 15. No `font-weight` tokens

`font-weight: bold`, `font-weight: 600`, and `font-weight: normal` all appear as raw values. `600` and `bold` are not the same value (600 is semi-bold in many variable fonts). A `--font-weight-normal`, `--font-weight-semibold`, and `--font-weight-bold` would make it consistent and easier to swap when using a variable font.
