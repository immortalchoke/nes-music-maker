# NESForge Design System — Audit & Componentization (Project A)

**Date:** 2026-03-26
**Scope:** Color tokens, typography, spacing, component classes, z-index — all in `nesforge.html`.

---

## Goals

1. Eliminate hardcoded hex values, font sizes, spacing, and z-index values throughout the codebase.
2. Replace ~80+ inline styles with named CSS classes.
3. Produce a clean, enforceable design system that the future `nesforge-design-system` skill can document.

**Visual change policy:** Most changes are visual no-ops. The two intentional visual shifts are called out explicitly in Section 1 with ⚠️.

---

## 1. Color Token System

### Variables to remove (duplicates)

**Safety rule:** Before removing any variable, grep for its name in the file. Only remove if zero usages found (or after all usages have been migrated to the canonical token in the same pass).

| Remove | Reason |
|---|---|
| `--bg2` | Identical to `--bg` (#46425e) |
| `--gray` | Identical to `--bg3` (#15788c) |
| `--gray2` | Identical to `--bg` (#46425e) |
| `--text2` | Identical to `--accent2` (#ffb0a3) |

### Variables to add or update

| Token | Value | Notes |
|---|---|---|
| `--bg` | `#46425e` | Unchanged |
| `--bg-deep` | `#1e1c30` | NEW — replaces 50+ hardcoded modal/panel backgrounds |
| `--bg3` | `#15788c` | Unchanged |
| `--bg-disabled` | `#444466` | NEW — replaces hardcoded disabled element backgrounds |
| `--hi` | `#00ccaa` | ⚠️ INTENTIONAL VISUAL CHANGE: was `#00b9be`. Brighter cyan becomes primary CTA. Every element using `--hi` will become slightly brighter/greener. Reviewed and accepted. |
| `--hi-dim` | `#00b9be` | NEW — the old `--hi` value; used for hover/focus states only |
| `--accent` | `#ff6973` | Unchanged |
| `--accent2` | `#ffb0a3` | Unchanged (remove `--text2` alias) |
| `--text` | `#ffeecc` | Unchanged |
| `--text-dim` | `#8888aa` | NEW — replaces hardcoded disabled/secondary text |
| `--border` | `2px solid var(--bg3)` | Unchanged — see border override note below |
| `--border-thin` | `1px solid var(--bg3)` | NEW — replaces hardcoded `1px solid #15788c` throughout |

### Border override note

`--border` is a shorthand token (`2px solid var(--bg3)`). When a component needs to override only the border color (e.g. `.btn:hover` changes `border-color`), always use individual properties — never `border: var(--border)` followed by `border-color` override on the same rule, as shorthand resets cascade. Pattern to follow:

```css
/* Correct — individual properties on base rule */
.btn {
  border-width: 2px;
  border-style: solid;
  border-color: var(--bg3);
}
.btn:hover { border-color: var(--hi-dim); } /* only overrides what changes */

/* Use --border shorthand only when no override will ever occur */
.modal { border: var(--border); }
```

### Hardcoded values to replace throughout

| Hardcoded value | Replace with | Notes |
|---|---|---|
| `#1e1c30` | `var(--bg-deep)` | |
| `#46425e` (in CSS/inline) | `var(--bg)` | |
| `#15788c` (in CSS/inline) | `var(--bg3)` | |
| `#00b9be` (in CSS/inline, non-hover contexts) | `var(--hi-dim)` | ⚠️ Context matters: if the element was "primary CTA", it should become `var(--hi)` |
| `#00ccaa` | `var(--hi)` | |
| `#ff6973` | `var(--accent)` | |
| `#ffb0a3` | `var(--accent2)` | |
| `#ffeecc` | `var(--text)` | |
| `#8888aa` | `var(--text-dim)` | |
| `#444466` | `var(--bg-disabled)` | |
| `#1a1a2e` | `var(--bg-deep)` | ⚠️ INTENTIONAL VISUAL CHANGE: `#1a1a2e` is slightly darker than `#1e1c30`. Used only in `.engaged` button state. Difference is imperceptible at this scale. Reviewed and accepted. |
| `#555577` | `var(--bg-disabled)` | Close enough; used only in inactive modal elements |
| `letter-spacing: 0.05em` | `letter-spacing: var(--tracking)` | |

### Exceptions (do not replace)

- `#e8e0c8` and `#333` — piano key colors. Unique to that component, hardcode is intentional.
- `rgba(0,0,0,*)` — transparency overlays. Keep hardcoded.
- Canvas `fillStyle` values in JS — cannot use CSS vars; see Section 2 for canvas text note.

---

## 2. Typography Scale

No changes to the scale — it's already correct. The work is consistent application everywhere it's been bypassed.

| Token | Value | Role |
|---|---|---|
| `--t-xs` | `8px` | Sub-labels, canvas overlays, kbd hints |
| `--t-sm` | `9px` | Button labels, dropdowns, field labels (most common) |
| `--t-base` | `10px` | Body text, general UI, note info |
| `--t-md` | `12px` | Section headings, emphasis |
| `--t-lg` | `14px` | Title text, modal headers |
| `--t-xl` | `18px` | Display text, startup screen |
| `--t-input` | `10px` | Form controls (prevents iOS auto-zoom) |

**Two additional tokens (moved here from color section):**

| Token | Value | Role |
|---|---|---|
| `--font` | `'Press Start 2P', monospace` | Replaces hardcoded font stack in 30+ inline styles |
| `--tracking` | `0.05em` | Replaces hardcoded `letter-spacing: 0.05em` in 20+ places |

**Migration:** Replace all hardcoded `font-size: 9px` (40+ instances), `font-size: 8px` (25+ instances), and `font-size: 10px` (15+ instances) in inline styles and JS-generated HTML with the appropriate `var(--t-*)`.

Replace all inline `font: 9px 'Press Start 2P', monospace` patterns with `font-family: var(--font)` + `font-size: var(--t-sm)` (or appropriate size token).

**Canvas text exceptions:** `fillText` calls use 5px, 6px, 8px hardcoded in JS — CSS vars cannot reach canvas context. These are documented as intentional and correct. Do not change them.

---

## 3. Spacing Scale

| Token | Value | Usage |
|---|---|---|
| `--space-1` | `4px` | Tight internal spacing — icon padding, tiny gaps |
| `--space-2` | `8px` | Standard control spacing — button padding, label gaps |
| `--space-3` | `12px` | Component internal padding — inputs, cards |
| `--space-4` | `16px` | Section padding — modal content, panel insets |
| `--space-5` | `24px` | Large section gaps — modal outer padding |
| `--space-6` | `36px` | Screen-level spacing — startup screen padding |

**Rounding rules (concrete):**

| Existing value | Target token | Decision rule |
|---|---|---|
| 1–3px | `--space-1` (4px) | Always round up — these are micro-gaps |
| 5–6px | `--space-2` (8px) | Always round up — they're standard control gaps |
| 10px | `--space-2` (8px) | If inside a compact control (button, input); `--space-3` (12px) if inside a panel/card |
| 14px | `--space-3` (12px) | If inside a modal section; `--space-4` (16px) if it's outer modal padding |
| All others | Nearest step | Use judgment; leave a comment if ambiguous |

Apply to all `padding`, `margin`, and `gap` values in CSS rules. Inline styles in JS-generated HTML get CSS classes instead (see Section 4).

---

## 4. Component Classes

All classes go in the `<style>` block in `nesforge.html`. They replace inline styles in both static HTML and JS-generated HTML strings.

### Buttons

One `.btn` base class with individual border properties (not shorthand) to allow clean hover overrides.

```css
.btn {
  background: var(--bg3);
  color: var(--text);
  border-width: 2px;
  border-style: solid;
  border-color: var(--hi-dim);
  font-family: var(--font);
  font-size: var(--t-sm);
  letter-spacing: var(--tracking);
  padding: var(--space-1) var(--space-2);
  cursor: pointer;
  border-radius: 0;
  image-rendering: pixelated;
}
.btn:hover   { background: var(--hi-dim); color: var(--bg); border-color: var(--hi-dim); }
.btn:active  { background: var(--accent); color: var(--text); border-color: var(--accent); }

.btn--primary  { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
.btn--primary:hover { background: var(--hi-dim); color: var(--bg); border-color: var(--hi-dim); }
.btn--danger   { background: var(--bg); color: var(--accent); border-color: var(--accent); }
.btn--engaged  { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
.btn--sm       { font-size: var(--t-xs); padding: 3px var(--space-1); } /* 3px vertical is intentional — --space-1 (4px) is too tall for inline track buttons */
.btn--icon     { width: 32px; height: 32px; padding: var(--space-1); }
.btn--full     { width: 100%; }
.btn--mob      {
  border-radius: 2px;
  border-color: transparent;
  border-width: 1px;
  padding: 0 var(--space-3);
  height: 44px;
  min-width: 44px;
}
```

**Existing class migration targets:**

| Existing class/pattern | Becomes |
|---|---|
| `.nes-btn` | `.btn` |
| `.nes-btn.engaged` | `.btn.btn--engaged` |
| `.mob-btn` | `.btn.btn--mob` |
| `.mob-btn:active` | `.btn.btn--mob:active` (handled by base `.btn:active`) |
| Insert/CTA buttons (bg: #00ccaa) | `.btn.btn--primary` |
| Delete/danger buttons (color: #ff6973, border: #ff6973) | `.btn.btn--danger` |
| Track M/S/REC buttons (compact) | `.btn.btn--sm` |
| Topbar icon buttons (32×32) | `.btn.btn--icon` |
| Full-width modal action buttons | `.btn.btn--full` |

Keep `.nes-btn` as a CSS alias (`/* alias */ .nes-btn { }` that extends `.btn`) during Phase 3–4 migration, then remove in Phase 5.

### Modals & Overlays

```css
.modal-overlay {
  position: fixed; inset: 0;
  background: rgba(0,0,0,0.7);
  display: flex; align-items: center; justify-content: center;
  z-index: var(--z-modal);
}
.modal {
  background: var(--bg-deep);
  border: var(--border);
  padding: var(--space-4);
  display: flex; flex-direction: column; gap: var(--space-3);
  font-family: var(--font);
}
.modal--sm  { width: 320px; }
.modal--lg  { width: min(680px, 94vw); max-height: 90vh; overflow-y: auto; }
.modal-title   { font-size: var(--t-lg); color: var(--text); letter-spacing: var(--tracking); margin-bottom: var(--space-2); }
.modal-section { display: flex; flex-direction: column; gap: var(--space-2); }
.modal-label   { font-size: var(--t-xs); color: var(--text-dim); letter-spacing: var(--tracking); font-family: var(--font); }
.modal-row     { display: flex; align-items: center; gap: var(--space-2); }
.modal-actions { display: flex; gap: var(--space-2); margin-top: var(--space-3); }
```

**Progression Builder / large overlays:** Use `.modal-overlay` with `z-index: var(--z-overlay)` (8000) via inline override or a `.modal-overlay--top` modifier. The `.modal--lg` class handles its container sizing.

### Form Controls

`.field-input` and `.field-select` share identical base declarations intentionally — they are a shared base. Future modifiers can add differences. Do not consolidate them into one selector, as they may diverge.

```css
.field         { display: flex; flex-direction: column; gap: var(--space-1); }
.field-label   { font-size: var(--t-xs); color: var(--text-dim); letter-spacing: var(--tracking); font-family: var(--font); }
.field-input   { background: var(--bg); color: var(--text); border-width: 2px; border-style: solid; border-color: var(--bg3); font-family: var(--font); font-size: var(--t-input); padding: var(--space-1) var(--space-2); }
.field-select  { background: var(--bg); color: var(--text); border-width: 2px; border-style: solid; border-color: var(--bg3); font-family: var(--font); font-size: var(--t-input); padding: var(--space-1) var(--space-2); }
```

### Layout Utilities

```css
.row     { display: flex; align-items: center; gap: var(--space-2); }
.col     { display: flex; flex-direction: column; gap: var(--space-2); }
.divider { width: 100%; height: 0; border: none; border-top: var(--border-thin); margin: var(--space-2) 0; }
```

### Status & Feedback

```css
.badge           { display: inline-block; padding: 1px var(--space-1); font-size: var(--t-xs); font-family: var(--font); letter-spacing: var(--tracking); border-width: 1px; border-style: solid; border-color: var(--bg3); color: var(--text-dim); }
.badge--active   { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
.status-error    { color: var(--accent); font-size: var(--t-xs); font-family: var(--font); }
.status-ok       { color: var(--hi); font-size: var(--t-xs); font-family: var(--font); }
```

---

## 5. Z-Index Scale

```css
--z-dropdown:   100;    /* context menus, tooltips */
--z-modal:      500;    /* standard modals, dialogs */
--z-modal-top:  1000;   /* modals that stack above other modals */
--z-overlay:    8000;   /* full-screen overlays (Progression Builder) */
--z-system:     99999;  /* audio unlock, scanlines — never covered */
```

**Current z-index → token mapping:**

| Current value | Token | Location |
|---|---|---|
| 1 | (keep hardcoded) | Black piano keys — CSS stacking, not layer management |
| 500 | `--z-modal` | Context menu, track settings popup |
| 501 | `--z-modal` | Error dialogs (same layer, remove the +1 hack) |
| 600 | `--z-modal` | Track creation dialog |
| 1000 | `--z-modal-top` | Export dialogs |
| 1001 | `--z-modal-top` | New track modal |
| 1500 | `--z-modal-top` | Velocity picker |
| 2000 | `--z-modal-top` | MagicGen modal |
| 8000 | `--z-overlay` | Progression Builder |
| 9999 | `--z-system` | Scanlines (body::after) |
| 10000 | `--z-system` | Export filename overlay |
| 99999 | `--z-system` | Audio unlock overlay |

---

## 6. Implementation Phases

Execute in this order. Each phase is independently verifiable before proceeding.

**Phase 1 — Tokens**
- Add all new CSS variables to `:root`
- Audit and confirm zero usages of duplicate variables (`--bg2`, `--gray`, `--gray2`, `--text2`), then remove them
- Update all existing CSS rules in `<style>` to use vars instead of hardcoded values
- No HTML or JS changes

**Phase 2 — CSS component classes**
- Add all component classes (`.btn`, `.modal`, `.field-*`, etc.) to `<style>` block
- No HTML or JS changes yet — classes exist but aren't applied

**Phase 3 — Static HTML migration**
- Update static HTML elements to use new class names
- Remove inline styles from static HTML
- `.nes-btn` alias keeps working throughout this phase

**Phase 4 — JS-generated HTML migration**
- Update all JS HTML strings to use new class names instead of inline styles
- Completion criterion: the following grep returns zero results (inline style attributes on buttons, modals, labels):
  ```
  grep -n "style=\".*font-size\|style=\".*background.*#\|style=\".*color.*#" nesforge.html
  ```
  Any remaining hits after Phase 4 are either intentional exceptions (canvas overlays, audio unlock) or missed migrations.
- JS locations to update: `showTrackSettings`, `showContextMenu`, `ChordTool.openInlinePicker`, `ChordTool._insertChord`, `MagicGen` modal builder, `ProgBuilder` startup/builder, `buildTopBar`, note info side panel, MIDI import dialog, export dialogs, BPM edit modal

**Phase 5 — Cleanup**
- Remove `.nes-btn` CSS alias
- Remove any orphaned CSS rules that are now superseded
- Run the Phase 4 grep one more time to confirm zero remaining inline style leaks

---

## 7. File Impact

All changes in `nesforge.html` only.

- `<style>` block: new variables, new/updated classes, removed duplicates
- Static HTML: class name updates, inline style removal
- JS code: HTML string updates in all modal/dialog generators (enumerated in Phase 4)

---

## Out of Scope (Project B)

The `nesforge-design-system` skill will encode this design system as Claude instructions for future development. One skill, single source of truth. Spec'd and built after Project A is complete and proven.
