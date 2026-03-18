# NESForge v2 — Piano Roll & UI Improvements Design

**Date:** 2026-03-17
**Scope:** Six improvements to `nesforge.html` — piano roll interactions, UI scaling, arrangement note previews, loop region visibility, insert chord, and multi-note selection.

---

## 1. Piano Roll Interactions (rework)

Replace the current click-to-place and drag-to-paint behavior with a Logic-style interaction model.

| Gesture | Action |
|---|---|
| Click empty cell | Nothing |
| Double-click empty cell | Place single note (1/16th, snapped to scale) |
| Click + drag on empty | Draw marquee selection rectangle |
| Click note body (no drag) | Preview note (play audio ~0.15s) |
| Double-click note body | Delete note (or entire chord group by chordId) |
| Click + drag note body | Move note; audio preview on each new pitch row |
| Drag any selected note body | Move entire selection by same delta |
| Click + drag right edge (within 6px) | Resize note (unchanged) |
| Click empty area | Deselect all |

**Implementation notes:**
- Distinguish single-click vs double-click using `e.detail` on `mousedown` — but since `dblclick` fires after two clicks, use a short timer (200ms) on mousedown: if a second click arrives within 200ms on the same target, treat as double-click action.
- More robustly: listen for `dblclick` event separately for place-note and delete-note actions. Single `mousedown` initiates drag detection (move/resize/marquee).
- Move drag: on mousedown on note body, record `startX/Y`, `origPitch`, `origStart`. On mousemove if delta > 4px, enter move mode. On mouseup without move → play preview. With move → finalize.
- Marquee: on mousedown on empty cell (no note hit), record `marqueeStart`. On mousemove draw translucent rect. On mouseup, select all notes whose bounding box intersects the rect.

---

## 2. UI Scaling

Double the size of all dimensional constants. Add more padding throughout.

### CSS changes
| Property | Current | New |
|---|---|---|
| `html, body` font-size | 8px | 14px |
| `#topbar` height | 48px | 72px |
| `#topbar` padding | `0 8px` | `0 16px` |
| `#topbar` gap | 16px | 24px |
| `#pianoroll-panel` height | 300px | 480px |

### JS constant changes
| Constant | Location | Current | New |
|---|---|---|---|
| `RULER_HEIGHT` | AV | 20 | 36 |
| `TRACK_HEIGHT` | AV | 48 | 96 |
| `BEAT_WIDTH` | AV | 20 | 40 |
| `HEADER_WIDTH` | AV | 160 | 240 |
| `KEY_WIDTH` | PR | 32 | 56 |
| `ROW_HEIGHT` | PR | 8 | 16 |
| `BEAT_WIDTH` | PR | 40 | 80 |
| `VELOCITY_HEIGHT` | PR | 60 | 90 |

### Canvas pixel positions (AV._drawTracks)
All hardcoded pixel positions for M/S buttons, REC button, knobs, and text in `AV._drawTracks` and `AV.initEvents` hit zones must be updated to match the new `TRACK_HEIGHT: 96`. Scale all y-offsets by 2x and x positions proportionally:
- M button: `(4, y+18, 14, 10)` → `(8, y+36, 20, 14)`
- S button: `(22, y+18, 14, 10)` → `(44, y+36, 20, 14)`
- REC button (DPCM): `(40, y+18, 24, 10)` → `(72, y+36, 36, 14)`
- Volume knob: `cx=60, cy=y+12, r=8` → `cx=100, cy=y+20, r=12`
- Pan knob: `cx=80, cy=y+12, r=8` → `cx=130, cy=y+20, r=12`
- Hit zones in `AV.initEvents` updated to match

---

## 3. Arrangement Lane Note Previews

Inside each pattern block in `AV._drawTracks`, draw a miniature representation of the pattern's notes after drawing the block background and label.

**Pitched tracks (pulse50, pulse25, triangle, chord, dpcm):**
- Map MIDI pitch to vertical position within block: `noteY = blockY + blockH - ((pitch / 127) * (blockH - 4))`
- Draw a 2px tall filled rect at that Y, x proportional to note.start within the block width
- Color: same as track NOTE_COLOR at 80% opacity

**Noise/drum tracks:**
- Draw a 2px tall tick mark at the note's x position, vertically centered in the block
- Color: same as track NOTE_COLOR at 80% opacity

**Clipping:** all mini-note drawing is clipped to the block's bounding rect using `ctx.save()` / `ctx.rect()` / `ctx.clip()` / `ctx.restore()`.

---

## 4. Loop Region Visibility

In `AV._drawRuler`, wrap the loop region drawing block in a condition:
```js
if (ProjectState.loopEnabled) {
  // draw loop region highlight and markers
}
```

The LOOP button in `buildTopBar` already toggles `ProjectState.loopEnabled` and calls `AV.draw()` — no further wiring needed.

---

## 5. Insert Chord Button

A `CHORD` button added to `#global-controls` in `buildTopBar()`.

**Popup UI** (fixed-position div, same pattern as `showContextMenu`):
- Root selector: C / C# / D / D# / E / F / F# / G / G# / A / A# / B
- Type selector: maj / min / dim / aug / maj7 / min7 / dom7
- Notes: 3 or 4 (radio/toggle)
- INSERT button

**Chord voicing table** (root = MIDI 60 = C4, intervals in semitones):
| Type | 3-note intervals | 4-note intervals |
|---|---|---|
| maj | 0, 4, 7 | 0, 4, 7, 12 |
| min | 0, 3, 7 | 0, 3, 7, 12 |
| dim | 0, 3, 6 | 0, 3, 6, 9 |
| aug | 0, 4, 8 | 0, 4, 8, 12 |
| maj7 | 0, 4, 7 | 0, 4, 7, 11 |
| min7 | 0, 3, 7 | 0, 3, 7, 10 |
| dom7 | 0, 4, 7 | 0, 4, 7, 10 |

**Insert behavior:**
- If no piano roll pattern is open (`PR.activePattern === null`), show `alert('Open a pattern in the piano roll first.')` and return.
- Find root MIDI = `rootSemitone + 60` (C4 as base, where rootSemitone = index of root in NOTE_ORDER).
- Compute playhead position relative to pattern: `relBar = ProjectState.playheadBar - entry.startBar`, snapped to nearest 1/16th.
- Create chord notes with `makeId()` as shared `chordId`, duration = 2 × sixteenth, velocity = 100.
- Push to `PR.activePattern.pattern.notes`, call `PR.draw()` and `PR.drawVelocity()`.

---

## 6. Multi-Note Selection

### Data model
Each note object gains an optional `selected` boolean (defaults to `undefined`/falsy = unselected).

### Selection rendering
In `PR.draw()`, selected notes render at `globalAlpha = 1.0` with a brighter/lighter version of the track color (mix toward white: `color + '88'` tint approach, or explicitly lighter hex). Unselected notes keep the velocity-based alpha (`0.4 + 0.6 * vel/127`).

Brighter color map (selected state):
| Track | Normal | Selected |
|---|---|---|
| pulse50 | `#cc3300` | `#ff6644` |
| pulse25 | `#ff6600` | `#ff9955` |
| triangle | `#00ccaa` | `#44ffdd` |
| noise | `#33aa66` | `#55dd88` |
| dpcm | `#9933cc` | `#cc66ff` |
| chord | `#ff9900` | `#ffcc44` |

### Marquee selection
`dragState = { type: 'marquee', startX, startY, currentX, currentY }`

On `mousemove`: update `currentX/Y`, draw the marquee rect (translucent teal fill + border), call `PR.draw()`.

On `mouseup`: compute which notes overlap the marquee rect, set `note.selected = true` for those, clear dragState, call `PR.draw()`.

Marquee rect drawn in `PR.draw()` as an overlay after all notes: `rgba(0, 204, 170, 0.15)` fill, `#00ccaa` 1px stroke.

### Moving a selection
When `dragState.type === 'move'` and the dragged note is selected (`dragState.note.selected`), apply the same pitch and start delta to ALL notes where `note.selected === true`.

### Deselect
On mousedown on empty cell (no note hit, not starting marquee initiation): set `note.selected = false` for all notes in active pattern, call `PR.draw()`.

Actually: deselect happens at the START of a new marquee drag (mousedown on empty replaces old selection with new marquee).

---

## File Impact

All changes are in `nesforge.html` only. No new files.

**Sections modified:**
- `<style>` — font size, topbar, pianoroll-panel height
- `PR` object — `KEY_WIDTH`, `ROW_HEIGHT`, `BEAT_WIDTH`, `VELOCITY_HEIGHT` constants; `draw()` for selected note rendering and marquee overlay
- `PR.initEvents` — full replacement with new interaction model
- `AV` object — `RULER_HEIGHT`, `TRACK_HEIGHT`, `BEAT_WIDTH`, `HEADER_WIDTH` constants; `_drawRuler()` for loop visibility; `_drawTracks()` for note previews and scaled pixel positions
- `AV.initEvents` — updated hit zone coordinates
- `buildTopBar()` — CHORD button + popup + insert logic
