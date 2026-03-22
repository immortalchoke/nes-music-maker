# NESForge v2 — Piano Roll & UI Improvements Design

**Date:** 2026-03-17
**Scope:** Nine improvements to `nesforge.html`.

---

## 1. Piano Roll Interactions (rework)

Replace the existing interaction model entirely. The old `'paint'` dragState branch and the old delete-on-mouseup behavior are removed completely and replaced with the following:

| Gesture | Action |
|---|---|
| Click empty cell | Nothing (deselects all) |
| Double-click empty cell | Place single note (1/16th, snapped to scale) |
| Click + drag on empty | Draw marquee selection rectangle |
| Click note body (no drag) | Preview note (play audio ~0.15s) |
| Double-click note body | Delete note (or entire chord group by chordId) |
| Click + drag note body | Move note; audio preview on each new pitch row |
| Drag any selected note body | Move entire selection by same delta |
| Click + drag right edge (within 6px) | Resize note (unchanged) |

**Double-click detection:** Use a separate `dblclick` event listener on the canvas for both place-note (empty cell) and delete-note (existing note). The `mousedown` listener handles single-click drag initiation only. On the second click of a double-click, `e.detail === 2` will also fire on `mousedown` — use this to suppress drag initiation for that event (check `if (e.detail >= 2) return;` at top of mousedown handler, allowing the `dblclick` listener to handle it).

**Deselect:** On `mousedown` on an empty cell (no note hit), immediately clear all `note.selected` flags before initiating the marquee. This way every new marquee drag starts fresh.

**Moving unselected note while others are selected:** Dragging an unselected note clears the current selection and moves only that note. Dragging a selected note moves all selected notes together.

**Implementation — full dragState type list:**
- `{ type: 'move', note, startX, startY, origPitch, origStart, lastPitch, lastStart, hasMoved, movingSelection }`
  - `movingSelection`: true if `note.selected` was true at drag start
- `{ type: 'marquee', startX, startY, currentX, currentY }`
- `{ type: 'resize', note, startX, origDur }`
- `{ type: 'velocity', ... }` — velocity canvas, unchanged

---

## 2. UI Scaling

Double all dimensional constants and add more padding.

### CSS changes
| Property | Current | New |
|---|---|---|
| `html, body` font-size | 8px | 14px |
| `#topbar` height | 48px | 72px |
| `#topbar` padding | `0 8px` | `0 16px` |
| `#topbar` gap | 16px | 24px |
| `#pianoroll-panel` height | 300px | 480px |

### JS constant changes
| Constant | Object | Current | New |
|---|---|---|---|
| `RULER_HEIGHT` | AV | 20 | 36 |
| `TRACK_HEIGHT` | AV | 48 | 96 |
| `BEAT_WIDTH` | AV | 20 | 40 |
| `HEADER_WIDTH` | AV | 160 | 240 |
| `KEY_WIDTH` | PR | 32 | 56 |
| `ROW_HEIGHT` | PR | 8 | 16 |
| `BEAT_WIDTH` | PR | 40 | 80 |
| `VELOCITY_HEIGHT` | PR | 60 | 90 |

### Canvas draw positions in AV._drawTracks (TRACK_HEIGHT: 96)
| Element | Draw call | New values |
|---|---|---|
| M button | `fillRect(x, y+h, w, h)` | `fillRect(8, y+38, 20, 14)` |
| M label | `fillText('M', x, y)` | `fillText('M', 14, y+49)` |
| S button | `fillRect(x, y+h, w, h)` | `fillRect(34, y+38, 20, 14)` |
| S label | `fillText('S', x, y)` | `fillText('S', 40, y+49)` |
| Track name | `fillText(name, x, y)` | `fillText(name, 8, y+24)` |
| Volume bar | see Section 9 | x=64, y+10, w=80, h=12 |
| Pan knob | `drawKnob(cx, cy, r, ...)` | `drawKnob(ctx, 162, y+20, 12, ...)` |
| REC button (DPCM) | `fillRect(x, y+h, w, h)` | `fillRect(60, y+56, 36, 14)` |
| Waveform canvas left | `panelRect.left + 42` | `panelRect.left + 64` |

### Hit zones in AV.initEvents (updated to match new draw positions)
| Zone | New hit rect |
|---|---|
| M button | `mx >= 8 && mx <= 28 && my >= ty+38 && my <= ty+52` |
| S button | `mx >= 34 && mx <= 54 && my >= ty+38 && my <= ty+52` |
| Volume bar drag | `mx >= 64 && mx <= 144 && my >= ty+10 && my <= ty+22` |
| Pan knob drag | `mx >= 150 && mx <= 174 && my >= ty+8 && my <= ty+32` |
| REC button (DPCM) | `mx >= 60 && mx <= 96 && my >= ty+56 && my <= ty+70` |
| Track settings (header fallthrough) | everything else in `mx < HEADER_WIDTH` zone |

---

## 3. Arrangement Lane Note Previews

Inside each pattern block in `AV._drawTracks`, after drawing the block background and name label, draw miniature note representations using `ctx.save()` / clip to block rect / `ctx.restore()`:

**Pitched tracks (pulse50, pulse25, triangle, chord):**
- For each note: `noteY = blockY + blockH - 4 - ((note.pitch / 127) * (blockH - 8))`
- Draw `fillRect(noteX, noteY, 2, 2)` at 80% opacity

**DPCM and noise tracks:** draw a 2px tall tick mark vertically centered in the block at the note's x position (same tick-mark approach — neither is melodic pitch).

**Color:** same as `NOTE_COLORS[track.channelType]` at `globalAlpha = 0.8`.

---

## 4. Loop Region Visibility

In `AV._drawRuler`, wrap the entire loop region block (fill + stroke markers) in:
```js
if (ProjectState.loopEnabled) { ... }
```
The LOOP button already calls `AV.draw()` — no additional wiring needed.

---

## 5. Insert Chord Button

A `CHORD` button added to `#global-controls` in `buildTopBar()`.

**Popup:** uses the same `showContextMenu` pattern but is a form div. Contains:
- Root: dropdown (C / C# / D / D# / E / F / F# / G / G# / A / A# / B)
- Type: dropdown (maj / min / dim / aug / maj7 / min7 / dom7)
- Notes: toggle button (3 or 4)
- INSERT button

**Chord intervals table (semitones from root):**
| Type | 3-note | 4-note |
|---|---|---|
| maj | 0, 4, 7 | 0, 4, 7, 12 |
| min | 0, 3, 7 | 0, 3, 7, 12 |
| dim | 0, 3, 6 | 0, 3, 6, 9 |
| aug | 0, 4, 8 | 0, 4, 8, 12 |
| maj7 | 0, 4, 7 | 0, 4, 7, 11 |
| min7 | 0, 3, 7 | 0, 3, 7, 10 |
| dom7 | 0, 4, 7 | 0, 4, 7, 10 |

**Insert logic:**
1. If `PR.activePattern === null`, `alert('Open a pattern in the piano roll first.')` and return.
2. `rootMIDI = Scales.NOTE_ORDER.indexOf(root) + 60` (C4 base).
3. `chordNotes = intervals.slice(0, numNotes).map(i => rootMIDI + i)`
4. Look up entry: `const entry = ap.track.arrangement.find(e => e.patternId === ap.pattern.id);`
5. `relBar = ProjectState.playheadBar - (entry ? entry.startBar : 0)`; snap to nearest 1/16th.
6. Call `ChordTool._insertChord(chordNotes, snappedRelBar, label)` to reuse existing chord insertion logic (handles `makeId`, `chordId`, `chordLabel`, `PR.draw()`).

---

## 6. Multi-Note Selection

### Data model
Each note gains optional `selected: boolean` (falsy by default).

### Selected note rendering
In `PR.draw()`, selected notes render at `globalAlpha = 1.0` using the brighter color from this table (not `color + '88'` — that produces transparency, not brightness):

| Track | Normal color | Selected color |
|---|---|---|
| pulse50 | `#cc3300` | `#ff6644` |
| pulse25 | `#ff6600` | `#ff9955` |
| triangle | `#00ccaa` | `#44ffdd` |
| noise | `#33aa66` | `#55dd88` |
| dpcm | `#9933cc` | `#cc66ff` |
| chord | `#ff9900` | `#ffcc44` |

Add a `SELECTED_NOTE_COLORS` map to the PR object alongside `NOTE_COLORS`.

### Marquee
`dragState = { type: 'marquee', startX, startY, currentX, currentY }`

On `mousemove`: update `currentX/Y`, call `PR.draw()` (which draws the marquee as an overlay).
On `mouseup`: find all notes whose pixel bounding box overlaps the marquee rect, set `note.selected = true`. Clear dragState.

Marquee overlay drawn at end of `PR.draw()` when `PR._dragState?.type === 'marquee'`: `rgba(0,204,170,0.15)` fill, `#00ccaa` 1px stroke. Expose dragState for draw via `PR._dragState` property set from `initEvents`.

### Marquee hit test (note overlaps rect)
```
noteX = KEY_WIDTH + note.start * barW - scrollX
noteY = rowIdx * ROW_HEIGHT - scrollY
noteW = note.duration * barW
overlaps if: noteX < rectRight && noteX+noteW > rectLeft && noteY < rectBottom && noteY+ROW_HEIGHT > rectTop
```

---

## 7. Track Settings Popup Fix

**Bug:** `showTrackSettings` registers a `click` dismiss listener via `setTimeout(..., 50)`. On desktop, mousedown → mouseup → click fires in rapid succession — the dismiss listener catches the same interaction that opened the popup, closing it immediately.

**Fix:** Change the dismiss event from `'click'` to `'mousedown'`, and increase the delay to 100ms so the opening mousedown has fully completed before the dismiss listener is armed.

```js
setTimeout(() => document.addEventListener('mousedown', () => el.remove(), { once: true }), 100);
```

Apply the same fix to `showContextMenu` and `ChordTool.openInlinePicker` which use the same pattern.

---

## 8. Volume Bar Slider + Labels

Replace the volume knob in `AV._drawTracks` with a horizontal bar slider. Add text labels for both volume and pan.

**Volume bar (drawn in canvas):**
- Background rect: `fillRect(64, y+10, 80, 12)` in `#222240`
- Fill rect: `fillRect(64, y+10, track.volume * 0.8, 12)` in `#00ccaa`
- Label: `fillText('VOL', 64, y+8)` in `#8888aa`, font `5px "Press Start 2P"`

**Pan knob (keep knob, add label):**
- Knob position: `drawKnob(ctx, 162, y+20, 12, (track.pan+1)/2, '#ff6600')`
- Label: `fillText('PAN', 150, y+8)` in `#8888aa`, font `5px "Press Start 2P"`

**Volume drag (AV.initEvents):**
- Hit zone: `mx >= 64 && mx <= 144 && my >= ty+10 && my <= ty+22`
- On drag: `track.volume = Math.max(0, Math.min(100, Math.round((mx - 64) / 0.8)))`
- (Direct value from x position, not delta-based)

---

## File Impact

All changes in `nesforge.html` only.

**Sections modified:**
- `<style>` — font size, topbar, pianoroll-panel
- `PR` object — constants, `NOTE_COLORS`, new `SELECTED_NOTE_COLORS`, `draw()` for selected rendering + marquee overlay
- `PR.initEvents` — full replacement with new model; expose `PR._dragState`
- `AV` object — constants, `_drawRuler()` loop visibility, `_drawTracks()` note previews + scaled positions + volume bar + pan label
- `AV.initEvents` — updated hit zones
- `buildTopBar()` — CHORD button + popup
- `showTrackSettings()`, `showContextMenu()`, `ChordTool.openInlinePicker()` — dismiss fix
