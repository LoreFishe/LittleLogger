# Little Logger — Development Handoff

Single-file HTML/CSS/JS workout tracker (currently `workout_tracker_v2.html`), built as a Claude.ai artifact. No frameworks — vanilla JS with manual DOM rendering via a `render()` function that rebuilds the view based on a `view` state variable (`home`, `session`, `manage`, `history`, `trends`, `calendar`).

**Important environment note:** the app currently persists all data through `window.storage` (`get`/`set`), an API that only exists inside Claude.ai's artifact sandbox. It will not work in a normal browser or Claude Code preview as-is. Before doing anything else, decide whether this stays a Claude.ai artifact (keep `window.storage`) or becomes a standalone app (needs `localStorage`, IndexedDB, or a real backend instead). Flag this decision back if it's not already been made — it affects how much of the persistence layer needs rewriting.

---

## 1. Bug fixes

### 1a. Row swipe (delete a set)

- Each set row should have a dedicated drag handle: **two parallel vertical lines** (a grip/handle icon, not dots, not a square), positioned at the **far left of the row — before the set number**, for every row layout (weighted, bodyweight, timed).
- Dragging must only initiate from that handle. Dragging elsewhere on the row (inputs, labels, padding) should not trigger the swipe.
- Swiping the handle reveals the row's delete button, same mechanics as today (drag left, releases either snap fully open or spring back closed — no partial-open states).

### 1b. Card swipe ("Same as Last")

- Remove the dedicated handle icon entirely for this gesture.
- The **entire exercise card is draggable** to reveal "Same as Last", **except** for the individual set rows themselves (which have their own handle-based swipe from 1a) and any interactive elements (buttons, inputs).
- Practically: dragging on the card background, header, exercise name, padding, etc. → triggers "Same as Last". Dragging inside a set row (even off its handle) → does nothing at the card level, since that area belongs to the row.
- Direction/size unchanged from the current implementation: swipe right, opens to half the card's width, only holds open if dragged the full distance, springs back otherwise.

### 1c. General swipe rule (already implemented, keep as-is)

- Swipes only "hold open" if dragged the full distance; otherwise they spring back on release. Don't reintroduce a partial-drag threshold.

---

## 2. New feature: Template color themes

Each template (Core Session, Secondary Session, and any user-created ones) can have its own accent color, chosen from a fixed 6-color palette:

| Name   | Hex       |
|--------|-----------|
| Pink   | `#ec4899` (current default) |
| Blue   | `#3b82f6` |
| Amber  | `#f0a83c` |
| Green  | `#22c55e` |
| Purple | `#a78bfa` |
| Cyan   | `#22d3ee` |

**Where the color applies:**
- The template's home-screen "Start" button (border + title text)
- The session header title text when that template is active
- The "Save Session" button while inside that template's session

**Where it's selected:**
- In Edit Templates, add a row of 6 tappable color swatches (small circles or squares) alongside the existing Rename / Reset / Delete controls for the selected template.
- Store the chosen color as a hex value on the template object (e.g. `data.templates[key].color`).
- Default to Pink (`#ec4899`) for templates that don't have a color set yet (including existing saved data — handle the missing-field case gracefully).

**Calendar integration:**
- On the Calendar page, the small dot marking a day with a logged session should use that session's template color instead of the current fixed pink.
- If multiple sessions were logged on the same day, show multiple dots side by side, one per distinct template color represented that day (dedupe by color/template — if two sessions that day used the same template, still show just one dot for it, not two identical ones).
- Keep dots small enough that 2-3 side by side still fit cleanly in the day cell; wrap or cap at a reasonable max (e.g. 3 dots, with remaining sessions still counted but not separately dotted) if a day gets unusually busy.

---

## 3. New feature: Configurable per-template widgets

Widgets (starting with the Rest Timer, which already exists) should become a **manageable set per template**, not a fixed always-on feature. In Edit Templates, add an "Add Widget" button (same visual pattern as the existing "Add Exercise to [Template]" button) that lets the user pick which widgets are active for that template. Selected widgets appear live during any session started from that template, positioned in the session screen the same place the Rest Timer sits today (between "Add Exercise" and "Save Session").

Build in this priority order:

### 3a. Plate calculator (highest priority)
- User enters a target weight.
- Widget outputs which plates to load per side, assuming a standard barbell (default 45 lb bar, but consider making bar weight configurable).
- Standard plate sizes: 45, 35, 25, 10, 5, 2.5 lb (adjust if the user wants kg support later — not required now).

### 3b. Stopwatch
- Simple count-up timer: start / pause / reset.
- No target time, no alert — just elapsed time on demand, for exercises timed by feel.

### 3c. HIIT / interval timer (needs design discussion before building)
This one is more involved than the other two and shouldn't be built until the following is resolved:
- Interval timers imply a **work/rest cycle exercise type** (e.g. 40s work / 20s rest × 8 rounds), which doesn't map cleanly onto the existing exercise types (weight, bodyweight, timed, checkbox). Does this become a new exercise type (e.g. `interval`), or is it purely a standalone widget with no exercise/logging tie-in at all (just a utility timer, unconnected to what gets saved in session history)?
- If it does log data, what's actually worth recording — rounds completed? Total work time? Nothing at all, and it's just a clock?
- Needs a round/work/rest configuration UI (number of rounds, work duration, rest duration) before the timer itself can be built.

Bring this back for a quick design pass before implementing — don't guess at the data model.

---

## 4. Interface decluttering

Two small visual cleanups, independent of everything else above:

- **Remove the up/down spinner arrows** on number inputs (weight, reps, seconds fields). These are the native browser increment/decrement arrows on `<input type="number">`. Hide them cross-browser (`-webkit-appearance: none` on the spin buttons for Chrome/Safari, `-moz-appearance: textfield` on the input itself for Firefox). Purely visual — keep `type="number"` and `inputmode` attributes as-is so the correct mobile keyboard still shows.
- **Remove the grey background strip behind each set row.** Currently each row sits on a slightly different dark shade (`--iron-900`) than the card it's inside (`--iron-800`), creating a subtle banding effect. Drop that background so the row blends into the card, leaving only the individual input fields (which have their own background) as visual contrast.

---

## 5. Suggested build order

1. Fix the two swipe bugs (1a, 1b) — small, contained, unblocks everything else being tested cleanly.
2. Interface decluttering (section 4) — trivial CSS-only changes, good to knock out alongside the swipe fixes since you'll already be in that part of the code.
3. Template color themes, including the calendar dot integration — self-contained, no dependency on the widget system.
4. Widget system scaffolding (the "Add Widget" picker in Edit Templates, storing which widgets are active per template, rendering them in session).
5. Plate calculator widget.
6. Stopwatch widget.
7. Pause and revisit HIIT/interval timer design before building it.
