# NESForge Design System Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply a consistent, enforceable design system to `nesforge.html` — clean color tokens, enforced typography scale, spacing scale, component CSS classes replacing ~80+ inline styles.

**Architecture:** Five sequential phases all targeting `nesforge.html`: (1) add/fix CSS variables, (2) add component CSS classes, (3) migrate static HTML, (4) migrate JS-generated HTML, (5) cleanup. Each phase is independently verifiable before the next begins. No behavior changes — purely a visual/structural refactor.

**Tech Stack:** Vanilla HTML/CSS/JS, single file. No build tools. Visual verification in browser.

**Spec:** `docs/superpowers/specs/2026-03-26-design-system-audit.md`

---

## File Structure

Single file modified throughout:
- **Modify:** `nesforge.html` — all phases

The `<style>` block starts at line 9. The `:root` block is lines 11–29.

---

## Task 1: Update CSS Variables in `:root` (Phase 1a)

**Files:**
- Modify: `nesforge.html:11-29`

Remove duplicate variables, update `--hi`, add all new tokens. This is the foundation everything else depends on.

- [ ] **Step 1: Open browser to verify current state**

  Open `nesforge.html` in a browser. Take a mental note of what the app looks like — button colors, modal backgrounds, active states. This is your baseline.

- [ ] **Step 2: Replace the entire `:root` block (lines 11–29)**

  Find:
  ```css
  :root {
    --bg: #46425e; --bg2: #46425e; --bg3: #15788c;
    --accent: #ff6973; --accent2: #ffb0a3;
    --hi: #00b9be; --hi2: #ffeecc;
    --gray: #15788c; --gray2: #46425e;
    --text: #ffeecc; --text2: #ffb0a3;
    --border: 2px solid #15788c;
    /* ── NESForge Type Scale ...
    ...
    --t-input: 10px; /* Minimum for form controls — prevents iOS auto-zoom */
  }
  ```

  Replace with:
  ```css
  :root {
    /* ── Backgrounds ─────────────────────────────────────────── */
    --bg:          #46425e;   /* primary app background */
    --bg-deep:     #1e1c30;   /* modals, panels, deep surfaces */
    --bg3:         #15788c;   /* teal — borders, headers, controls */
    --bg-disabled: #444466;   /* inactive/disabled element backgrounds */

    /* ── Accent & Text ───────────────────────────────────────── */
    --hi:          #00ccaa;   /* primary CTA — insert, generate, engaged */
    --hi-dim:      #00b9be;   /* hover/focus state, secondary interactive */
    --accent:      #ff6973;   /* error, delete, warning */
    --accent2:     #ffb0a3;   /* secondary text, soft labels */
    --text:        #ffeecc;   /* primary text */
    --text-dim:    #8888aa;   /* disabled text, hints, sub-labels */

    /* ── Borders ─────────────────────────────────────────────── */
    --border:      2px solid var(--bg3);
    --border-thin: 1px solid var(--bg3);

    /* ── Spacing Scale ───────────────────────────────────────── */
    --space-1: 4px;    /* tight — icon padding, micro gaps */
    --space-2: 8px;    /* standard — button padding, control gaps */
    --space-3: 12px;   /* component — input/card internal padding */
    --space-4: 16px;   /* section — modal content, panel insets */
    --space-5: 24px;   /* large — modal outer padding, section gaps */
    --space-6: 36px;   /* screen — startup screen, hero spacing */

    /* ── Z-Index Scale ───────────────────────────────────────── */
    --z-dropdown:  100;
    --z-modal:     500;
    --z-modal-top: 1000;
    --z-overlay:   8000;
    --z-system:    99999;

    /* ── Typography Scale ────────────────────────────────────── */
    --font:    'Press Start 2P', monospace;
    --t-xs:    8px;    /* sub-labels, canvas overlays, kbd hints */
    --t-sm:    9px;    /* button labels, dropdowns, field labels */
    --t-base:  10px;   /* body text, general UI, note info */
    --t-md:    12px;   /* section headings, emphasis */
    --t-lg:    14px;   /* title text, modal headers */
    --t-xl:    18px;   /* display text, startup screen */
    --t-input: 10px;   /* form controls — prevents iOS auto-zoom */
    --tracking: 0.05em; /* universal letter-spacing */
  }
  ```

- [ ] **Step 3: Verify the app still loads**

  Reload the browser. The app should look identical. If anything looks broken, check the `:root` block for a typo.

- [ ] **Step 4: Commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: update CSS variables — add missing tokens, remove duplicates"
  ```

---

## Task 2: Update Existing CSS Rules to Use New Variables (Phase 1b)

**Files:**
- Modify: `nesforge.html` (CSS `<style>` block, lines ~43–900)

Replace every hardcoded hex value in the `<style>` block with the appropriate CSS variable. No HTML or JS changes yet.

**Important:** Only change values inside the `<style>` block. Do NOT change JS code or HTML attributes yet.

- [ ] **Step 1: Replace hardcoded colors in CSS rules**

  Work through each replacement. Use find & replace with care — confirm each change is in the CSS block only.

  | Find (in CSS) | Replace with |
  |---|---|
  | `#1e1c30` | `var(--bg-deep)` |
  | `#46425e` | `var(--bg)` |
  | `#15788c` | `var(--bg3)` |
  | `#00b9be` | `var(--hi-dim)` — **exception:** if the element is a primary CTA (e.g. an INSERT or GENERATE button background), use `var(--hi)` instead |
  | `#00ccaa` | `var(--hi)` |
  | `#ff6973` | `var(--accent)` |
  | `#ffb0a3` | `var(--accent2)` |
  | `#ffeecc` | `var(--text)` |
  | `#8888aa` | `var(--text-dim)` |
  | `#444466` | `var(--bg-disabled)` |
  | `#555577` | `var(--bg-disabled)` |
  | `#1a1a2e` | `var(--bg-deep)` |
  | `letter-spacing: 0.05rem` | `letter-spacing: var(--tracking)` |
  | `letter-spacing: 0.05em` | `letter-spacing: var(--tracking)` |
  | `'Press Start 2P', monospace` | `var(--font)` |

  **Exceptions — do NOT replace:**
  - `#e8e0c8` and `#333` in piano key styles — intentional, keep hardcoded
  - `rgba(0,0,0,*)` transparency values — keep hardcoded
  - Canvas `fillStyle` in JS — not in scope for this task
  - `#00ffcc` in `.nes-btn.engaged` border — leave for now (gets replaced in Task 3)

- [ ] **Step 2: Replace hardcoded z-index values in CSS**

  | Find | Replace with |
  |---|---|
  | `z-index: 9999` (scanlines `body::after`) | `z-index: var(--z-system)` |
  | `z-index: 2000` (`#magic-modal`) | `z-index: var(--z-modal-top)` |
  | Any `z-index: 500` or `z-index: 501` | `z-index: var(--z-modal)` |
  | Any `z-index: 1000` or `z-index: 1001` | `z-index: var(--z-modal-top)` |
  | `z-index: 8000` | `z-index: var(--z-overlay)` |

- [ ] **Step 3: Update `#topbar` border-color to use new variable**

  Find in CSS (line ~102):
  ```css
  #topbar, #arrangement-panel, #pianoroll-panel {
    border-color: var(--gray);
  ```
  Replace `var(--gray)` with `var(--bg3)` (removing the now-deleted alias).

- [ ] **Step 4: Update `.mob-btn:active` and `.mob-bpm-display` color references**

  Line ~358: `.mob-btn:active { background: var(--hi); color: #1e1c30; }` — replace `#1e1c30` with `var(--bg-deep)`.
  Line ~364: `color: var(--hi);` — this already uses a var, but `--hi` now resolves to `#00ccaa` (intentional visual change — engaged state is slightly brighter).

- [ ] **Step 5: Verify in browser**

  Reload. The app should look almost identical. The only intentional visual shift: primary CTA elements that were `#00b9be` and are now `#00ccaa` will be slightly brighter/greener. Hover/focus states become `#00b9be`. If anything looks wrong, diff the CSS block to find the unintended change.

- [ ] **Step 6: Commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: replace hardcoded CSS values with design tokens"
  ```

---

## Task 3: Add Component CSS Classes (Phase 2)

**Files:**
- Modify: `nesforge.html` — add new CSS classes to the `<style>` block, immediately after the existing `.nes-btn` block (~line 80)

No HTML or JS changes. Just define the classes — they aren't used yet.

- [ ] **Step 1: Add the `.btn` system after `.nes-btn.engaged { ... }` (around line 79)**

  Add this block:
  ```css
  /* ── Design System: Button ───────────────────────────────────
     .btn is the unified button base. .nes-btn is kept as an alias
     during migration and removed in Phase 5.
     ------------------------------------------------------------ */
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
    display: inline-flex;
    align-items: center;
    justify-content: center;
  }
  .btn:hover   { background: var(--hi-dim); color: var(--bg); border-color: var(--hi-dim); }
  .btn:active  { background: var(--accent); color: var(--text); border-color: var(--accent); }
  .btn--primary { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
  .btn--primary:hover { background: var(--hi-dim); color: var(--bg); border-color: var(--hi-dim); }
  .btn--danger  { background: var(--bg); color: var(--accent); border-color: var(--accent); }
  .btn--engaged { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
  .btn--sm      { font-size: var(--t-xs); padding: 3px var(--space-1); } /* 3px vertical intentional — space-1 too tall for inline track buttons */
  .btn--icon    { width: 32px; height: 32px; padding: var(--space-1); }
  .btn--full    { width: 100%; }
  .btn--mob     {
    border-radius: 2px;
    border-color: transparent;
    border-width: 1px;
    padding: 0 var(--space-3);
    height: 44px;
    min-width: 44px;
  }
  ```

- [ ] **Step 2: Add modal, form, layout, and status classes**

  Add this block immediately after the `.btn` system:
  ```css
  /* ── Design System: Modals ───────────────────────────────────── */
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

  /* ── Design System: Form Controls ───────────────────────────── */
  .field         { display: flex; flex-direction: column; gap: var(--space-1); }
  .field-label   { font-size: var(--t-xs); color: var(--text-dim); letter-spacing: var(--tracking); font-family: var(--font); }
  /* .field-input and .field-select share identical base — intentional shared base, may diverge later */
  .field-input   { background: var(--bg); color: var(--text); border-width: 2px; border-style: solid; border-color: var(--bg3); font-family: var(--font); font-size: var(--t-input); padding: var(--space-1) var(--space-2); }
  .field-select  { background: var(--bg); color: var(--text); border-width: 2px; border-style: solid; border-color: var(--bg3); font-family: var(--font); font-size: var(--t-input); padding: var(--space-1) var(--space-2); }
  .field-input:focus, .field-select:focus { border-color: var(--hi-dim); }

  /* ── Design System: Layout Utilities ────────────────────────── */
  .row     { display: flex; align-items: center; gap: var(--space-2); }
  .col     { display: flex; flex-direction: column; gap: var(--space-2); }
  .divider { width: 100%; height: 0; border: none; border-top: var(--border-thin); margin: var(--space-2) 0; }

  /* ── Design System: Status & Feedback ───────────────────────── */
  .badge         { display: inline-block; padding: 1px var(--space-1); font-size: var(--t-xs); font-family: var(--font); letter-spacing: var(--tracking); border-width: 1px; border-style: solid; border-color: var(--bg3); color: var(--text-dim); }
  .badge--active { background: var(--hi); color: var(--bg-deep); border-color: var(--hi); }
  .status-error  { color: var(--accent); font-size: var(--t-xs); font-family: var(--font); }
  .status-ok     { color: var(--hi); font-size: var(--t-xs); font-family: var(--font); }
  ```

- [ ] **Step 3: Verify the app still loads and nothing broke**

  Reload in browser. The new classes exist but aren't applied yet — nothing should change visually.

- [ ] **Step 4: Commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: add design system component CSS classes"
  ```

---

## Task 4: Migrate Static HTML to Use New Classes (Phase 3)

**Files:**
- Modify: `nesforge.html` — HTML section (everything after `</style>` and before the `<script>` block)

Apply the new classes to existing static HTML elements. Remove inline styles where class covers them.

- [ ] **Step 1: Update `.nes-btn` topbar icon buttons**

  The topbar icon buttons in `#global-controls` and `.tb-group` currently use `.nes-btn`. They also get the `.tb-group .nes-btn` override for icon sizing. Add `.btn--icon` class alongside `.nes-btn` for each one so the sizing now comes from the design system.

  Find every instance in static HTML like:
  ```html
  <button class="nes-btn" id="btn-play">
  ```
  Add `btn` class alongside `nes-btn`:
  ```html
  <button class="nes-btn btn" id="btn-play">
  ```
  And for icon-sized topbar buttons, also add `btn--icon`:
  ```html
  <button class="nes-btn btn btn--icon" id="btn-play">
  ```

  **Note:** Keep `nes-btn` class during Phase 3 — it still provides the base styles. `btn` classes layer on top and prepare for the eventual `nes-btn` removal in Phase 5.

- [ ] **Step 1b: Update `.mob-btn` elements in static mobile HTML**

  In the `#mob-transport` and `#mob-tracks` static HTML sections, find all `<button class="mob-btn">` elements and add the new classes:
  ```html
  <!-- Before -->
  <button class="mob-btn" id="mob-play-btn">
  <!-- After -->
  <button class="mob-btn btn btn--mob" id="mob-play-btn">
  ```
  Keep `mob-btn` during migration; it will be aliased and eventually removed in Phase 5.

- [ ] **Step 2: Update `#pianoroll-panel` toolbar buttons (the `#pr-topbar` area)**

  In the static `#pr-topbar` HTML, find all buttons with inline styles like:
  ```html
  <button class="nes-btn" style="flex:1;font-size:9px;padding:0">
  ```
  Replace with:
  ```html
  <button class="nes-btn btn btn--sm">
  ```

- [ ] **Step 3: Update `#pr-topbar` layout**

  In `#pr-topbar`, find any `<div style="display:flex;...gap:...">` wrappers and replace inline styles with `.row` or `.col` class where appropriate.

- [ ] **Step 4: Update the `#magic-modal` static HTML**

  In the static `#magic-modal-content` block, update:
  - The modal container: add class `modal modal--sm` (keep `id="magic-modal-content"` for JS hooks)
  - Any section title `<div>`s: add class `modal-title` and remove inline font/color styles
  - Field label `<div>`s: add class `modal-label` and remove inline font/color styles
  - Any inline `display:flex; gap:` wrappers: replace with `.row` or `.col`

- [ ] **Step 5: Update the audio unlock overlay**

  Find the `#ios-unlock-overlay` element (around line 908). It has a long inline style. Replace:
  ```html
  style="position:fixed;inset:0;z-index:99999;background:rgba(0,0,0,0.92);display:flex;flex-direction:column;align-items:center;justify-content:center;gap:16px;"
  ```
  With:
  ```html
  style="z-index:var(--z-system);background:rgba(0,0,0,0.92);" class="modal-overlay col"
  ```
  (Keep the inline z-index and background override — `modal-overlay` provides the positioning and flex layout.)

- [ ] **Step 6: Verify in browser**

  Reload. All controls should look identical. Check the piano roll toolbar, topbar buttons, and magic modal for any visual regressions.

- [ ] **Step 7: Commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: migrate static HTML to design system classes"
  ```

---

## Task 5: Migrate JS-Generated HTML to Use New Classes (Phase 4)

**Files:**
- Modify: `nesforge.html` — JS section (the `<script>` block)

This is the largest task. Every modal and dialog generated by JavaScript needs its inline styles replaced with class names. Work through each JS function systematically.

**Completion criterion:** When done, this grep should return zero results (or only the intentional exceptions listed at the end):
```bash
grep -n "style=\".*font-size\|style=\".*background.*#\|style=\".*color.*#" nesforge.html
```

- [ ] **Step 1: Update `showContextMenu` and `showTrackSettings`**

  Find `showContextMenu` in the JS. The generated HTML looks like:
  ```js
  el.innerHTML = `<div style="background:#1e1c30;border:2px solid #15788c;padding:8px;...">...`
  ```
  Replace inner container inline styles with class:
  ```js
  el.innerHTML = `<div class="modal modal--sm">...`
  ```
  For each `<button>` inside: add `class="btn"`, remove inline style.
  For each label `<span>` or `<div>`: add `class="modal-label"`, remove inline style.

  Repeat for `showTrackSettings` — same pattern.

- [ ] **Step 2: Update `ChordTool.openInlinePicker`**

  Find `ChordTool.openInlinePicker`. The popup container HTML uses inline styles for background, border, padding, z-index. Replace with `.modal.modal--sm`. Update inner buttons to `.btn`, selects to `.field-select`, labels to `.modal-label`.

- [ ] **Step 3: Update the new-track modal (in `buildTopBar` or wherever `showNewTrackModal` lives)**

  Find the HTML string that creates the new track dialog. Replace the container inline styles with `.modal.modal--sm`. Replace label divs with `.modal-label`. Replace inputs with `.field-input`. Replace the button row with `.modal-actions`. Replace action buttons with `.btn.btn--primary` and `.btn`.

- [ ] **Step 4: Update the MIDI import modal**

  Find the MIDI import dialog HTML string. Same pattern: `.modal.modal--sm` for container, `.modal-label` for labels, `.field-input`/`.field-select` for inputs, `.modal-actions` for button row, `.btn` for buttons.

- [ ] **Step 5: Update the BPM edit modal**

  Find the BPM edit dialog (around where `#mob-bpm-display` click handler builds the edit UI). Replace container inline styles with `.modal.modal--sm`, input with `.field-input`, buttons with `.btn`.

- [ ] **Step 6: Update export filename dialog**

  Find the export filename overlay HTML string. Replace overlay container with `.modal-overlay` + inline `z-index: var(--z-system)` (since this sits above everything else). Replace inner container with `.modal.modal--sm`. Replace label with `.modal-label`, input with `.field-input`, buttons with `.btn` and `.btn--primary`.

- [ ] **Step 7: Update `MagicGen` modal builder**

  Find where `MagicGen` builds its drum/chord preset buttons dynamically. Replace any inline `style="font-size:8px;padding:4px 2px;..."` patterns on buttons with `.btn.btn--sm`. Replace any inline label styles with `.modal-label`.

- [ ] **Step 8: Update `ProgBuilder` startup and builder modals**

  Find `ProgBuilder.openStartup()` and the main builder HTML. The startup modal uses full-screen overlay. Replace container with `.modal-overlay` + `style="z-index:var(--z-overlay)"`. Inner card containers: add `.modal`. Title divs: add `.modal-title`. Button rows: add `.modal-actions`. Buttons: add `.btn` or `.btn--primary`.

- [ ] **Step 9: Update note info side panel**

  Find where the note info side panel HTML is generated (the panel with pitch/duration labels and pitch-shift/harmonize buttons). Replace inline `style="font:9px 'Press Start 2P'..."` label spans with `.modal-label`. Replace pitch-shift and harmonize buttons with `.btn.btn--sm` or `.btn.btn--full`.

- [ ] **Step 10: Run the completion grep**

  ```bash
  grep -n "style=\".*font-size\|style=\".*background.*#\|style=\".*color.*#" nesforge.html
  ```

  Review each remaining hit. Intentional exceptions to keep:
  - Audio unlock overlay — `background: rgba(0,0,0,0.92)` (transparency, not a token)
  - Canvas overlay elements — if any use inline styles for dynamic positioning
  - Track color dots — inline `background: #RRGGBB` where color is dynamic per-track data
  - Any `style="display:none"` or `style="display:flex"` visibility toggles — these are behavioral, not design

  Any other remaining hits are unfinished migrations — fix them before proceeding.

- [ ] **Step 11: Visual check**

  Reload in browser. Open every dialog: new track, track settings, chord picker, MIDI import, export, BPM edit, Progression Builder startup, MagicGen. Verify each looks correct — padding, colors, button styles consistent throughout.

- [ ] **Step 12: Commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: migrate JS-generated HTML to design system classes"
  ```

---

## Task 6: Cleanup (Phase 5)

**Files:**
- Modify: `nesforge.html`

Remove the migration alias and any orphaned rules.

- [ ] **Step 1: Check `.nes-btn` usage count**

  ```bash
  grep -c "nes-btn" nesforge.html
  ```

  Review each remaining `nes-btn` reference. If all static HTML has been migrated to `btn` classes in Task 4, the only remaining usages of `nes-btn` should be the CSS rule definitions themselves (`.nes-btn`, `.nes-btn:hover`, etc.) and the `.tb-group .nes-btn` size override.

- [ ] **Step 2: Confirm all `.nes-btn` and `.mob-btn` HTML usages have been migrated**

  ```bash
  grep -n "class=\"nes-btn\"" nesforge.html
  grep -n "class=\"mob-btn\"" nesforge.html
  ```

  Every hit should also include `btn` in its class list (e.g. `class="nes-btn btn"`). If any don't, add `btn` (and relevant modifier) before proceeding.

- [ ] **Step 2b: Delete the old `.nes-btn` and `.mob-btn` CSS blocks entirely**

  Remove the original `.nes-btn` block (the one with hardcoded hex values, lines ~71–79) and the `.mob-btn` block. The `.btn` system defined in Task 3 fully replaces them. After deletion, search for any remaining `.nes-btn` or `.mob-btn` references in CSS:
  ```bash
  grep -n "\.nes-btn\|\.mob-btn" nesforge.html
  ```
  The only acceptable hits are HTML class attributes (e.g. `class="nes-btn btn"`). All CSS rule definitions for `.nes-btn` and `.mob-btn` should be gone.

  Then remove `nes-btn` and `mob-btn` from all HTML class attributes:
  ```bash
  # Verify count before
  grep -c "nes-btn\|mob-btn" nesforge.html
  ```
  Do a final sweep replacing `class="nes-btn btn"` → `class="btn"`, `class="mob-btn btn btn--mob"` → `class="btn btn--mob"`, etc.

- [ ] **Step 3: Remove orphaned CSS rules**

  Search for any CSS rules that reference the removed variables (`--bg2`, `--gray`, `--gray2`, `--hi2`, `--text2`):
  ```bash
  grep -n "var(--bg2)\|var(--gray)\|var(--gray2)\|var(--hi2)\|var(--text2)" nesforge.html
  ```
  For each hit, replace with the canonical token (`--bg`, `--bg3`, `--text-dim` as appropriate).

- [ ] **Step 4: Final grep verification**

  Run all three checks and confirm clean:
  ```bash
  # No inline design-system-style leaks
  grep -c "style=\".*font-size\|style=\".*background.*#\|style=\".*color.*#" nesforge.html

  # No removed variable references
  grep -c "var(--bg2)\|var(--gray)\|var(--gray2)\|var(--hi2)\|var(--text2)" nesforge.html
  ```

  Both should return `0` (or only the documented exceptions).

- [ ] **Step 5: Full visual regression check**

  Reload in browser. Check every screen:
  - Desktop: topbar, arrangement view, piano roll, all open dialogs
  - Piano roll: open a pattern, check note colors, toolbar buttons, ruler
  - Modals: new track, track settings, chord picker, MIDI import, export, Progression Builder
  - Mobile: switch to mobile viewport, check transport bar, track buttons, piano grid

- [ ] **Step 6: Final commit**

  ```bash
  git add nesforge.html
  git commit -m "refactor: design system complete — remove migration aliases, final cleanup"
  ```

---

## Verification Summary

After all tasks complete:

1. `grep -c "var(--bg2)\|var(--gray2)\|var(--hi2)\|var(--text2)" nesforge.html` → `0`
2. `grep -c "style=\".*font-size\|style=\".*background.*#\|style=\".*color.*#" nesforge.html` → `0` (plus documented exceptions)
3. App opens and all features work visually — no regressions
4. Every modal/dialog uses `.modal`, `.modal-label`, `.modal-actions`
5. Every button uses `.btn` (with `.nes-btn` as a kept alias until full HTML migration is confirmed)
